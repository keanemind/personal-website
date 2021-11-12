---
title: "Erlang httpc: Figuring Out Strings, Charlists, and Binaries"
date: 2021-11-11T21:48:08.911Z
draft: false
---
I started learning Elixir earlier this year by writing a simple script to download an HTML webpage and print it out to the console. Rather than reaching for a more popular, Elixir-native library like [httpoison](https://github.com/edgurgel/httpoison), I wanted to try using the standard library, or more specifically, Erlang's [httpc request function](https://www.erlang.org/doc/man/httpc.html#request-1). 

Everything was swell until I tried my script with a page that had non-ASCII characters in it. For some reason, `ë` in the original HTML file was being printed out as `Ã«` in my console. 

I was setting the `body_format` option to `string` (the default). That causes the response body to be provided as a list of integers. The docs say that it is a "list of ASCII characters", which is odd, seeing as neither `Ã` nor `«` are actually part of US-ASCII. It would rather appear that the body is a list of ISO 8859-1 (Latin-1) code points. [That was the format of Erlang strings before Erlang/OTP R13](https://www.erlang.org/doc/apps/stdlib/unicode_usage.html#standard-unicode-representation). 

The problem is that the body will be in that form even if the response has the header `content-type: text/html; charset=UTF-8`. So in the UTF-8 encoded HTML file that I requested, `ë` was encoded with two bytes: `0xC3 0xAB`. But `request` put each of those bytes as their own entry in the list of code points I received as the response body. I had assumed that the list was an Erlang string in Unicode (known as a char list in the Elixir docs), and accordingly, I converted it into an Elixir string (a UTF-8 binary) via [List.to_string](https://hexdocs.pm/elixir/1.12/List.html#to_string/1). That function interpreted every item in the list as a separate Unicode code point. And thus, `0xC3 0xAB` became `Ã«`.

To avoid this, I should have done one of two things

1. set the `body_format` option to `binary` when using `request`
2. kept `body_format` set to `string`, converting the list to a binary by interpreting every item in the list as a *byte*, not a code point: [`list_to_binary`](https://www.erlang.org/doc/man/erlang.html#list_to_binary-1) accomplishes this

The lingering questions I have following this investigation are

1. What does `file = File.open!("top50.json", [:write, :utf8])` do, as opposed to `file = File.open!("top50.json", [:write])`?
2. What is the difference between `IO.write` and `IO.binwrite`? What does the former do if the file is opened with `:utf8`, versus without? Note, `IO.write` of a string containing `ë` when `File.open` is without `:utf8` causes an `Erlang error: :no_translation`.

Below are the notes I accumulated while trying to figure this out

> * httpc with the default `:body_format, :string` was giving me a list of integers, not a binary.
> * `to_string` was reading the list of integers as a list of code points, with each element of the list representing a different character. 
> * The list of integers was supposed to be read as a list of byte values, with each element of the list representing a byte. 
> * The byte values as represented by the list were a proper UTF-8 encoding of the source text. In UTF-8, multiple bytes might represent one character.
> * This is (probably) why one character from the page (ë) became two characters in my strings (Ã«).
> * Latin 1 (ISO/IEC 8859-1) encoding uses a single byte (8 bits) for each character
> * UTF-8 encodes the first 128 Unicode code points with a single byte, and beyond that with 2 or more bytes
> * Latin 1 encodes the first 256 Unicode code points with a single byte
> * `List.to_string` has the important docs: "Note that this function expects a list of integers representing Unicode code points. If you have a list of bytes, you must instead use the :binary module."
> * `:unicode.characters_to_binary(body, :utf8, :latin1)` only converted the list of byte values into a binary, because every single-byte "character" it was seeing had a single-byte representation in Latin 1. It didn't do any transcoding. 
> * https://erlang.org/doc/man/binary.html#list_to_bin-1 or http://erlang.org/doc/man/erlang.html#iolist_to_binary-1 was probably the right thing to use to deal with the list I was getting from httpc
> * Erlang strings are lists of integer code points http://erlang.org/doc/apps/stdlib/unicode_usage.html#standard-unicode-representation
> * Before Erlang/OTP R13, all Erlang strings were lists of Latin 1 code points, and so were lists of bytes
> * httpc gave me a LEGACY Erlang string. It didn't decode the bytes into Unicode code points— instead, it just gave me the bytes as a list of integers, which is indistinguishable from a Unicode Erlang string!
> * httpc docs http://erlang.org/doc/man/httpc.html#data-types define `string() = list of ASCII characters`, and interestingly https://erlang.org/doc/man/string.html uses `string()` to mean legacy (Latin 1) strings