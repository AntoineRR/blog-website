---
layout: article
title: Rust error handling with anyhow
mathjax: false
show_subscribe: false
---

Rust has a unique way of handling errors, and it can be a bit tricky at first. Some crates exists to help you facing the rough edges of Rust error handling, and [anyhow](https://crates.io/crates/anyhow) is one of the most popular.

<!--more-->

# Why you need anyhow

The most useful feature of anyhow is its `Result<T>` type. Yes. `Result<T>` and not `Result<T, U>` as the one provided in the standard library. With anyhow, there is no need to specify the type of the error that lies in an instance of the `Result` enum.

Consider the following code:

{%highlight rust%}
use std::{error::Error, fs};

fn main() {
    get_file_content().unwrap();
}

fn failing_function() -> Result<String, String> {
    Err("Failure!".to_string())
}

fn get_file_content() -> Result<String, Box<dyn Error>> {
    let output = failing_function()?;
    println!("{output}");
    let content = fs::read_to_string("some_file.txt")?;
    Ok(content)
}
{%endhighlight%}

The purpose of this code is very limited but it highlights something. The `get_file_content` method should return a String or an error, and if you do not care about the type of this error, you have to wrap it inside a `Box` as a trait object, and specify a `Result<String, Box<dyn Error>>` as the return type, which is verbose and annoying. A solution could be to alias your own `Result<T>` type to `Result<T, Box<dyn Error>>`, but a better solution is to use the anyhow crate, which expose his own `Result<T>` type as well as other useful things we will describe here.

The updated code looks like this:

{%highlight rust%}
use anyhow::{anyhow, Result};
use std::fs;

fn main() {
    get_file_content().unwrap();
}

fn failing_function() -> Result<String> {
    Err(anyhow!("Failure!"))
}

fn get_file_content() -> Result<String> {
    let output = failing_function()?;
    println!("{output}");
    let content = fs::read_to_string("some_file.txt")?;
    Ok(content)
}
{%endhighlight%}

We got rid of the error types, which makes the code more readable.

We do not have more information about the type of the error by using anyhow compared to `Box<dyn Error>` as it also uses trait objects. You would have to handle the error types separately in case you care about their nature, but it is not the purpose of this article.
{:.info}

# Error handling troubleshooting

Let's now see how to use anyhow based on the problems you could face during error handling!

## Returning an Err containing a String

## Returning a custom Error

## Adding context to errors

## Converting Result to Option

## Converting Option to Result

## Retrieve the error type

## Returning early with an error
