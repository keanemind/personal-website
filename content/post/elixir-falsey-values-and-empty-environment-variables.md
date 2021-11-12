---
title: "Elixir: Falsey Values and Empty Environment Variables"
date: 2021-11-12T22:13:47.014Z
draft: false
---
I keep forgetting that the complete list of falsey values in Elixir is `:nil` and `:false`. Being accustomed to JavaScript, I find myself continuing to make the mistake of expecting `if 0`, `if ""`, and the like not to execute.

One thing that surprised me is that [`System.get_env`](https://hexdocs.pm/elixir/1.12/System.html#get_env/2) and [`System.fetch_env`](https://hexdocs.pm/elixir/1.12/System.html#fetch_env/1) treat an environment variable with an empty string value as "set". 

I often want to use a default value if an environment variable is an empty string or not set.

In JavaScript, I only have to do 
```
if (process.env.MY_ENV_VAR) {
  /* ... */
} else {
  /* Use a default */
}
```
But in Elixir, I have to do
```
if System.get_env("MY_ENV_VAR") && String.length(System.get_env("MY_ENV_VAR")) > 0 do
  # ...
else
  # Use a default
end
```
to achieve the same thing.