---
title: Defining Functions in Header Files
date: '2018-09-04T16:26:12-05:00'
draft: false
---
I discovered C++ inline functions a few weeks ago as I worked on the Blip programming language, which was the end-of-semester project in EE312 Software Design and Implementation I. I didn't write here about it and I can't reveal the source code because it's a school assignment.

However, I do want to write about a specific incident where I tried to define a method outside of the class definition, but inside the header file. Like this:
```
// header_file.h
#ifndef header_file
#define header_file 1
class MyClass {
    void my_method();
};

void MyClass::my_method() {
    // code
}
#endif
```
When I compiled, there was a linker error that said the method had multiple definitions. My first thought was that the `#ifndef` should have protected the contents of the header file from being repeated. It wasn't long before I realized that the `#ifndef` only protects the header file from being repeated in a certain source file (compilation unit).

Say I `#include` the header and several other files in my `main.cpp` file. The `#ifndef` prevents the various `#include`s from copy pasting the header file into `main.cpp` multiple times. However, each `.cpp` file (compilation unit) has its own distinct `#define` values because the preprocessor runs separately for each `.cpp` file. So if I have another file called `other.cpp`, the header's contents could still be pasted into there by a `#include` *at most once*. And in fact, this is the situation that I had. Thus, when the linker tried to combine my `.cpp` files, it discovered that my method had multiple definitions: one in each `.cpp` file that included the header. 

To summarize, `#ifndef` prevents the header from being included multiple times in a single `.cpp` file, but it doesn't prevent the header from being included into two or more different `.cpp` files.

I learned by Googling that the solution is to make the method `inline`, like this:
```
inline void MyClass::my_method() {
```
From what I understand, inline functions are functions that are copy pasted wherever they are called. This means the program counter doesn't need to jump to the location of the function at runtime, which would be the case if the function wasn't inline. I'm not exactly sure why, but in C++ (and *not* in C),  an inline function is allowed to be defined in multiple compilation units. Perhaps it is because the copy pasting occurs only within a given compilation unit.

As a final interesting note, I learned that functions defined inside a class definition are inline functions! This is probably why you can have methods defined inside class definitions in header files that are included in multiple `.cpp` files without any linker error at all. That may have been confusing, so here is what I mean:
```
// header_file.h
#ifndef header_file
#define header_file `
class MyClass {
    void my_method() {
        // code
    }
};
#endif
```
No problems, because methods defined inside class definitions are implicitly inline.
