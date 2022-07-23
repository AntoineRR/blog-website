---
layout: article
title: Rust polymorphism - From abstract classes to traits
mathjax: false
show_subscribe: false
---

I guess facing challenges when developing in Rust is a mandatory path when you mostly used Object Oriented Programming (OOP) languages before. In this article, I want to highlight different ways of creating a function with a parameter that has an unknown concrete type at the time of writing the code.

<!--more-->

# A tale of building a function

Let's say we want to create a really simple project for calculating the 2D area of various shapes. To build this in Rust, we just have to define several structs that will have one shared behavior: *the ability to calculate their area*. Rust provides **traits** to help us factorize our code, and more. Here is probably how we would do it using the trait system:

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

*Cool*! We defined a `Shape` trait with a `get_area` function, and implemented this trait for two structures as an example: a `Rectangle` and a `Circle`. So now we have our structures ready, but how do we use them? We may want for example a function that **prints the area of a given Shape** for the user. Let's see how we will implement it.

{% highlight rust %}
pub fn print_area(shape: ...
{% endhighlight %}

*Wait*... How shall we choose the type of the `shape` argument? It could be either a `Rectangle` or a `Circle` in our case! In a classical OOP language, we would have an **abstract class** to specify the type of the argument, but in Rust we only have... A trait right? Let's see how we could use this trait to define the type of this argument.

# Trait bounds for arguments

Rust provides several ways to specify that the argument of a function has to implement a trait, without ever mentioning its real type. This feature is called a **trait bound**, and is something you apply on a **generic type** parameter. There are several ways to write a trait bound for generics in Rust:

{% highlight rust %}
// First function, using the full `where` syntax:
pub fn print_area_where<T>(shape: &T)
    where T: Shape
{
    println!("{}", shape.get_area());
}

// Second function using a simplified syntax:
pub fn print_area_simplified<T: Shape>(shape: &T) {
    println!("{}", shape.get_area());
}

// Last function using the syntactic sugar keyword `impl`:
pub fn print_area_impl(shape: &impl Shape) {
    println!("{}", shape.get_area());
}
{% endhighlight %}

In this case there is no need to take ownership of the `shape` variable so we are just borrowing it, but if your function needs to own `shape` it will be ok to just remove the `&` for the argument type.
{:.info}

Nice isn't it? This seems to be exactly what we were looking for. Those three functions all allow for a `shape` argument of any type, as long as it implements the `Shape` trait. They are all equivalent, and will be understood in the same way by the Rust compiler.

What? You are asking which one you should use? Well that's up to you. In my opinion, you should use the one that leads to the most *beautiful* and *readable* implementation for your function. The only rule here is that the `where` keyword is best when you have to specify a lot of trait bounds, especially if you have several generic types.

Anyway, this seems like a very nice way to avoid implementing a different function for a `Rectangle` and a `Circle`. Now we can just call the function like this:

{% highlight rust %}
print_area(&Rectangle { width: 2.0, height: 3.0 });
print_area(&Circle { radius: 2.0 })
{% endhighlight %}

Everything seems to be fine ([playground](https://play.rust-lang.org/?version=stable&mode=debug&edition=2021&gist=20911bea4cc174e96f3cca557d100b39))! Let's spice things up a little and create either a Rectangle or a Circle depending on a boolean value, eventually provided by a user.

{% highlight rust %}
// `is_circle` is provided by the user earlier in the code
let shape = match is_circle {
    true => Circle { radius: 2.0 },
    false => Rectangle { width: 2.0, height: 3.0 },
};

print_area(&shape);
{% endhighlight %}

***Kaboum***.

``error[E0308]: `match` arms have incompatible types``
{:.error}

The compiler doesn't want the `match` arms to return a different type :(
Indeed, Rust wants to know the type of the `shape` variable at compile time, and thus this kind of thing is not allowed. We have to find a trick to store our `shape` variable, and this is where trait objects join the party!

# Trait objects

For when you do not know the type of a variable before runtime, Rust has a feature called dynamic dispatch. To declare a variable as dynamically dispatched, you have to use the `dyn` keyword, and such a variable is called a **trait object**.

{% highlight rust %}
// `is_circle` is provided by the user earlier in the code
let shape: &dyn Shape = match is_circle {
    true => &Circle { radius: 2.0 },
    false => &Rectangle { width: 2.0, height: 3.0 },
};
{% endhighlight %}

Finally, this code compiles! We are now using a trait object to store the `shape` variable, which can either be a `Circle` or a `Rectangle`. Note that we have to use a reference to declare our variable here. This is due to trait objects not having a size known at compile time (they are dynamically sized). This means the compiler do not know how many bytes to allocate to store a trait object. Hiding it behind a reference allows us to declare the `shape` variable, because references do have a known size.

We could have used a smart pointer instead of a simple reference, for example a `Box`.
{:.info}

There is one drawback to using a trait object though... We cannot use our previous `print_area` function with this `shape` variable! Indeed, our previous function is using generics, and what Rust doesn't tell you explicitely is that every generic types are expected to implement `Sized` by default!

`Sized` is a trait that is automatically implemented for every type that has a size known at compile time.
{:.info}

As we use `shape` as an argument for our `print_area` function, Rust will understand that our generic type `T` is `dyn Shape`, which again is not `Sized`.

Fortunatelly, there is a way to specify that a function's argument does not have to necesseraly be `Sized`, and it is with the special `?Sized` syntax. Here is how we should rewrite our functions to make them work using a trait object as an argument:

{% highlight rust %}
// First function, using the full `where` syntax:
pub fn print_area_where<T>(shape: &T)
    where T: Shape + ?Sized
{
    println!("{}", shape.get_area());
}

// Second function using a simplified syntax:
pub fn print_area_simplified<T: Shape + ?Sized>(shape: &T) {
    println!("{}", shape.get_area());
}

// Last function using the syntactic sugar keyword `impl`:
pub fn print_area_impl(shape: &(impl Shape + ?Sized)) {
    println!("{}", shape.get_area());
}
{% endhighlight %}

As the `?Sized` trait bound specifies that an argument can either be `Sized` or not, we can use those functions with references to an instance of a concrete type or to a trait object ([playground](https://play.rust-lang.org/?version=stable&mode=debug&edition=2021&gist=32a2e18abbbc80b38ed9397569ca96fd)).

If we used a `Box<dyn Shape>` instead of a `&dyn Shape` to store our `shape` variable, we can call the `print_area`s functions like this: `print_area_...(&*shape)`. This is because we first dereference the `Box` to access its inner value (the `Deref` trait is implemented for `Box`, more details [here](https://doc.rust-lang.org/book/ch15-02-deref.html#using-boxt-like-a-reference)) and then take a reference to it.
{:.info}

Another thing we could do is using `&dyn Shape` as the type of the `shape` argument for our `print_area` function!

{% highlight rust %}
// A function using a trait object
pub fn print_area_dyn(shape: &dyn Shape) {
    println!("{}", shape.get_area());
}
{% endhighlight %}

Doing this allows us to use our function with both concrete type instances and trait objects as before ([playground](https://play.rust-lang.org/?version=stable&mode=debug&edition=2021&gist=bdc762428ca8311e5ac398facdef7e92)). So... How do we know what we should use as the type of the `shape` argument then???

# Generics VS trait objects

Let's compare the two methods we highlighted. As we saw before, the first three functions we wrote (`print_area_where`, `print_area_simplified`, and `print_area_impl`) rely on generics and really work the same way behind the scenes. This means we only have two methods to compare: using **generics** and using **trait objects**. As you may have guessed, none of these methods is inherently better than the other, and it all depends on your personal use case. To really understand that, we should go deeper into the implementation of those features in Rust.

## Generics

Rust is a stronlgy typed language. This means you cannot rely on [duck typing](https://en.wikipedia.org/wiki/Duck_typing) like you do with *Python* or *Javascript* for example. Rust *has* to know the exact type you are using in your code. When using generics, Rust needs a way to know what concrete type you use in your function. Rust allows developers to use generics because it compiles the function for each type it is called with behind the scenes. This means that if we call our `print_area_impl` with a `Rectangle` and a `Circle`, we will have in fact two compiled versions of this function in our executable.

This process is called **monomorphisation** and is a **zero cost abstraction** meaning it adds no overload to our program execution time.
{:.info}

The counter part to this abstraction is that if we call the function with a lot of different types, we will end up with a bigger executable in the end, because we will have a version of this same function for each type.

In short, generics favor *performances* at the cost of *memory usage*.

## Trait objects

Dynamic dispatch is another useful feature of Rust which allows to abstract away the concrete type of a variable as we saw earlier, using **trait objects**. In fact, trait objects are nothing more than an object that links to a concrete type instance and its methods. Performing dynamic dispatch means that we will at runtime look inside our trait object and follow the pointers to the object's methods to know the one we should call. This has the benefit of not generating any more static code when compiling, but has a runtime cost.

We could sum that up saying trait objects favor *memory usage* over *performances*.

## Benchmarks

To further compare those two methods, I ran a benchmark comparing their performances using [criterion](https://github.com/bheisler/criterion.rs). The benchmark is available [here](https://github.com/AntoineRR/rust-polymorphism-benchmark) on GitHub if you want to run it locally. The times are given for my personal computer, it is only relevant to compare those times relative to each others and you should not care about the absolute values. 

|                                       | Generics: `print_area_impl` | Trait objects: `print_area_dyn` |
| ------------------------------------- | --------------------------- | ------------------------------- |
| **Reference to concrete object**: `&` | 0.5 ns                      | 2.5 ns                          |
| **Reference to dyn object**: `&dyn`   | 1.0 ns                      | 2.9 ns                          |

We can notice that specifying a generic type as the `shape` argument is 3 to 5 times faster than specifying a trait object type.

# Conclusion

We have two different choice for the type of the parameter for our `print_area` function. Using generics will provide great performances, at the cost of a bigger executable. On the other hand, using trait objects has an oposite effect: bad performances but a smaller executable.
Therefor, the choice of the method you want to use depends on your use case. Although, in my opinion, the use of generics with trait bounds is probably the best choice in most situations. If you do provide your function as an external API, consider adding the `?Sized` trait bound for a compability with trait objects, the users of your crate will probably thank you!
