---
layout: article
title: Rust polymorphism
mathjax: false
---

I often face polymorphism challenges when developing in Rust. I guess it is a mandatory path when you mostly used OOP languages before, and suddenly have to adapt to a new way of sharing behavior between structures. Rust provides traits to help structuring code, and I really like those, although it is sometimes difficult to work with.

<!--more-->

## An example to better understand the issue

Let's say we want to create a really simple project for calculating the area of various shapes. Easy right? We just have to define several struct that will have one shared behavior: the ability to calculate their area. Here's how we'll do it using Rust trait system:

{% highlight rust %}
pub trait Shape {
    fn get_area(&self) -> f64;
}

pub struct Rectangle {
    width: f64,
    height: f64,
}

impl Shape for Rectangle {
    fn get_area(&self) -> f64 {
        self.width * self.height
    }
}

pub struct Circle {
    radius: f64,
}

impl Shape for Circle {
    fn get_area(&self) -> f64 {
        std::f64::consts::PI * self.radius * self.radius
    }
}
{% endhighlight %}

So now we have our structures ready, but how do we use them? We may want for example a function that prints the area of a given Shape for the user. Let's see how we will implement it.

{% highlight rust %}
pub fn print_area(shape: ...
{% endhighlight %}

Wait... How will we go about the type of shape? It could either be a Rectangle or a Circle! In a classical OOP language, we would have a parent class to specify the type of the argument, but in Rust we only have... A trait? Let's see how we could use this trait to define the type of this argument.

## Trait bounds for arguments

Rust provides several ways to specify that the argument of a function has to implement a trait, without ever mentioning its type. This feature is called a trait bound, and is something you apply on a generic type parameter.

{% highlight rust %}
// First method, using the full `where` syntax:
pub fn print_area<T>(shape: &T)
    where T: Shape
{
    println!("{}", shape.get_area());
}

// Second method using a simplified syntax:
pub fn print_area<T: Shape>(shape: &T) {
    println!("{}", shape.get_area());
}

// Last method using the syntactic sugar keyword `impl`:
pub fn print_area(shape: &impl Shape) {
    println!("{}", shape.get_area());
}
{% endhighlight %}

Cool isn't it? This seems to be exactly what we where looking for. Of course, in this case there is no need to take ownership of the `shape` variable so we are just borrowing it, but if your method needs to own `shape` it will be ok to just remove the `&` for our argument type.

Seems like a very nice way to avoid implementing a different method for Rectangle and Circle. Now we can just call the method like this:

{% highlight rust %}
print_area(&Rectangle { width: 2.0, height: 3.0 });
print_area(&Circle { radius: 2.0 })
{% endhighlight %}

Everything seems to be fine! Let's complicate things a little and create either a Rectangle or a Circle depending on a boolean value, eventually provided by a user.

{% highlight rust %}
// `is_circle` is provided by the user earlier in the code
let shape = match is_circle {
    true => Circle { radius: 2.0 },
    false => Rectangle { width: 2.0, height: 3.0 },
};

print_area(&shape);
{% endhighlight %}

Oh... The compiler refuses that the match arms return a different type :(
Indeed, Rust wants to know the type of the `shape` variable at compile time, and thus this kind of thing is not allowed. We have to find a trick to store our `shape` variable. This is where trait objects join the party!

## Trait objects

For when you do not know the type of a variable in advance, Rust has a feature called dynamic dispatch.

<!-- TODO: write about dyn -->

## Pros and cons of each method

<!-- TODO: develop -->

- Generics: a method for each type is created by the compiler
- Dynamic dispatch: Slower because of vtable

## Conclusion

<!-- TODO: table to explain how to choose a type ? -->