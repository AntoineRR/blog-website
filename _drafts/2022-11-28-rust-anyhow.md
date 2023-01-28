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

# Error handling tips

Let's now see how to use anyhow based on the problems you could face during error handling!

## Creating an Error

### From a simple &str

Sometimes, you may want to create an `Err` value by using `Err("Failure!".into())` but using it as a `anyhow::Result` variant will make the compiler display this error:

mismatched types
expected struct `anyhow::Error`, found `&str`
{:.error}

In this case, you want to use the `anyhow!` macro. It will convert your `&str` or any type implementing `Debug` and `Display` to an error.
{%highlight rust%}
#[derive(Debug)]
struct MyError;

impl std::fmt::Display for MyError {
    fn fmt(&self, f: &mut std::fmt::Formatter<'_>) -> std::fmt::Result {
        write!(f, "Bad :(")
    }
}

fn failing_function() -> Result<String> {
    let err1: Result<String> = Err(anyhow!("Oh no!"));
    let err2: Result<String> = Err(anyhow!(MyError));  // MyError must implement Debug and Display
    return err1;  // or err2, both have the appropriate type
}
{%endhighlight%}

### From a more sophisticated enum

You can also easily use an enum to represent your custom errors.

{%highlight rust%}
#[derive(Debug)]
enum MyErrors {
    LetsFixThisTomorrowError,
    ThisDoesntLookGoodError,
    ImmaGetFiredError,
}

impl std::fmt::Display for MyErrors {
    fn fmt(&self, f: &mut std::fmt::Formatter<'_>) -> std::fmt::Result {
        match self {
            Self::LetsFixThisTomorrowError => write!(f, "Doesn't look too bad"),
            Self::ThisDoesntLookGoodError => write!(f, "Let's get to work"),
            Self::ImmaGetFiredError => write!(f, "Wish me luck"),
        }
    }
}

fn failing_function() -> Result<String> {
    Err(anyhow!(MyErrors::LetsFixThisTomorrowError))
}
{%endhighlight%}

### The purpose of the Error trait

You may have noticed that we implemented `Debug` and `Display` only on our errors, but not the `std::error::Error` trait. This trait isn't mandatory for the `anyhow!` macro. However, you should probably still implement it, for the following reasons:
- It helps people to immediately spot the purpose of your struct
- It standardizes what struct can be considered as an error
- It has a `source` attribute for additional information

Therefore, you should add this one more line to make our enum a "real" error!

{%highlight rust%}
impl std::error::Error for MyErrors {}
{%endhighlight%}

## Adding context to errors

Adding context to your errors can be helpful for debugging your application. `anyhow` provides a `Context` trait that gives access to a `context` method you can use on `Result` types (not only the `Result` from anyhow). This method will add the information you specify in the error.

{%highlight rust%}
fn main() {
    error_with_context().unwrap()
}

fn error_with_context() -> Result<()> {
    Err(anyhow!("Wrong")).context("From a useless function")
}
{%endhighlight%}

This code will display the following error when run:

thread 'main' panicked at 'called `Result::unwrap()` on an `Err` value: From a useless function
Caused by:
    Wrong', src/main.rs:16:26
{:.error}

The context method can be chained as it returns the object it was called on, so you can add context as your error is returned by several methods.

## Error conversion

Error conversion isn't specifically related to `anyhow` but it is always useful to know, so I included it here.

### Converting Result to Option

Imagine you have a method that has to return an `Option` and you are calling a method that returns a `Result` inside it. You may want to convert your `Result` to an `Option` to be able to use the beautiful `?` operator without having to write a `match` statement. Calling the `ok` method on your `Result` will do just that!

{%highlight rust%}
fn return_result() -> Result<()> {
    Ok(())
}

fn return_option() -> Option<()> {
    return_result().ok()?;
    Some(())
}
{%endhighlight%}

Note that this method will discard any `Err` variant and replace it with a `Option::None`!
{:.warning}

### Converting Option to Result

Now you can expect something similar that will work the other way around. Here is how it is done for `anyhow`:

{%highlight rust%}
fn return_result() -> Result<()> {
    return_option().ok_or_else(|| anyhow!("None value received"))?;
    Ok(())
}

fn return_option() -> Option<()> {
    None
}
{%endhighlight%}

This works but is quite verbose. An easier way to do this with `anyhow`, is to use the `context` method, which happens to also be available on `Option`!

{%highlight rust%}
fn return_result() -> Result<()> {
    return_option().context("None value received")?;
    Ok(())
}

fn return_option() -> Option<()> {
    None
}
{%endhighlight%}

## Retrieve the error type

As stated above, `anyhow` uses trait objects to handle different type of errors. This means that the actual type of the error is lost in the process. Fortunately, it is still possible to retrieve the type of the error by downcasting it!
Here is an example of how you could do it for our previous `MyErrors` enum and our `failing_function`:

{%highlight rust%}
match failing_function() {
    Ok(s) => println!("{}", s),
    Err(e) => match e.downcast_ref() {
        Some(MyErrors::LetsFixThisTomorrowError) => (),
        Some(MyErrors::ThisDoesntLookGoodError) => (),
        Some(MyErrors::ImmaGetFiredError) => (),
        None => (),
    },
}
{%endhighlight%}

## Returning early with an error

Finally, `anyhow` provides a utility macro to return early from your methods with an error: the `bail!` macro. Here is how it is used:

{%highlight rust%}
fn bail_macro() -> Result<()> {
    // Equivalent to: return Err(anyhow!("Error occured!"));
    bail!("Error occured!");
}
{%endhighlight%}
