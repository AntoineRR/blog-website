---
layout: article
title: Rust polymorphism
mathjax: false
---

I often face polymorphism challenges when developing in Rust. I guess it is a mandatory path when you mostly used OOP languages before, and suddenly have to adapt to a new way of sharing behavior between structures. Rust provides traits to help structuring code, and I really like those, although it is sometimes difficult to work with.

<!--more-->

# An example to better understand the issue

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

# Trait bounds for arguments

Rust provides several ways to specify that the argument of a function has to implement a trait, without ever mentioning its type. This feature is called a trait bound, and is something you apply on a generic type parameter.

{% highlight rust %}
// First method, using the full `where` syntax:
pub fn print_area_where<T>(shape: &T)
    where T: Shape
{
    println!("{}", shape.get_area());
}

// Second method using a simplified syntax:
pub fn print_area_simplified<T: Shape>(shape: &T) {
    println!("{}", shape.get_area());
}

// Last method using the syntactic sugar keyword `impl`:
pub fn print_area_impl(shape: &impl Shape) {
    println!("{}", shape.get_area());
}
{% endhighlight %}

Cool isn't it? This seems to be exactly what we where looking for. Of course, in this case there is no need to take ownership of the `shape` variable so we are just borrowing it, but if your method needs to own `shape` it will be ok to just remove the `&` for our argument type.

Seems like a very nice way to avoid implementing a different method for Rectangle and Circle. Now we can just call the method like this:

{% highlight rust %}
print_area(&Rectangle { width: 2.0, height: 3.0 });
print_area(&Circle { radius: 2.0 })
{% endhighlight %}

Everything seems to be fine ([playground](https://play.rust-lang.org/?version=stable&mode=debug&edition=2021&gist=20911bea4cc174e96f3cca557d100b39))! Let's complicate things a little and create either a Rectangle or a Circle depending on a boolean value, eventually provided by a user.

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

# Trait objects

For when you do not know the type of a variable in advance, Rust has a feature called dynamic dispatch. To declare a variable as dynamically dispatched, you have to use the `dyn` keyword.

{% highlight rust %}
// `is_circle` is provided by the user earlier in the code
let shape: &dyn Shape = match is_circle {
    true => &Circle { radius: 2.0 },
    false => &Rectangle { width: 2.0, height: 3.0 },
};
{% endhighlight %}

Well, this code compiles! We are now using a trait object to store our shape, which can either be a `Circle` or a `Rectangle`. Note that we have to use a reference to declare our variable here. This is because trait objects do not have a size known at compile time (they are dynamically sized), and hiding them behind a reference allows us to declare the variable, as references do have a known size.

We could have used a smart pointer instead of a simple reference, for example a `Box`.
{:.info}

There is one drawback to using a trait object tho... We cannot use our previous `print_area` function with this shape variable! Indeed, our previous function is using generics, and what Rust doesn't tell you explicitely is that every generic types are expected to be `Sized` by default! As we use shape as an argument for our `print_area` method, Rust will understand that our generic type `T` is `dyn Shape`, which again is not `Sized`.

Fortunatelly, there is a way to specify that a function's argument does not have to be `Sized`, and it is with the special `?Sized` syntax. Here is how we should rewrite our methods to make them work for the trait object parameters:

{% highlight rust %}
// First method, using the full `where` syntax:
pub fn print_area_where<T>(shape: &T)
    where T: Shape + ?Sized
{
    println!("{}", shape.get_area());
}

// Second method using a simplified syntax:
pub fn print_area_simplified<T: Shape + ?Sized>(shape: &T) {
    println!("{}", shape.get_area());
}

// Last method using the syntactic sugar keyword `impl`:
pub fn print_area_impl(shape: &(impl Shape + ?Sized)) {
    println!("{}", shape.get_area());
}
{% endhighlight %}

As the `?Sized` trait bound specifies that an argument can either be `Sized` or not, we can use those functions with renferences to an instance of a concrete type or to a trait object ([playground](https://play.rust-lang.org/?version=stable&mode=debug&edition=2021&gist=32a2e18abbbc80b38ed9397569ca96fd)).

If we used a `Box<dyn Shape>` instead of a `&dyn Shape` to store our `shape` variable, we can call the `print_area`s methods like this: `print_area_...(&*shape)`. This is because we first dereference the `Box` to access its inner value and then take a reference to it.
{:.info}

Another thing we could do is using `&dyn Shape` as the type of the `shape` argument!

{% highlight rust %}
// A method using a trait object
pub fn print_area_dyn(shape: &dyn Shape) {
    println!("{}", shape.get_area());
}
{% endhighlight %}

Doing this allows us to use our method with both concrete type instances and trait objects as before ([playground](https://play.rust-lang.org/?version=stable&mode=debug&edition=2021&gist=bdc762428ca8311e5ac398facdef7e92)). So... How do we know what we should use as a type argument then?

# Generics VS trait objects

<!-- TODO: develop, benchmark ? -->

As we saw before, the first three methods we wrote (`print_area_where`, `print_area_simplified`, and `print_area_impl`) rely on generics and really work the same way. This means we only have two methods to compare: using generics and using trait objects. As you may have guessed, none of these methods is inherently better than the other, and it all depends on the context. To really understand that, we should go deeper into the implementation of those features in Rust.

## Generics

Rust is a stronlgy typed language. This means you cannot rely on some [duck typing](https://en.wikipedia.org/wiki/Duck_typing) like you do with Python or Javascript for example. Rust *has* to know the exact type you are using for your code. When using generics, Rust needs a way to know what type you use in your method. Rust allows developers to use generics because it compiles the method for each types it is called with behinf the scenes. This means that if we call our `print_area_impl` with a `Rectangle` and a `Circle`, we will have in fact two compiled versions of this method in our executable. This process is called **monomorphisation** and is a **zero cost abstraction** meaning it adds no overload to our program execution time.

The counter part to this abstraction is that if we call the method with a lot of different types, we will end up with a bigger executable in the end, because we will have a method for each type.

## Trait objects

Dynamic dispatch is another useful feature of Rust which allows returning different concrete types from a scope, such as the `match` statement we wrote earlier. However, Rust remains strongly typed, so how could it possibly violate its own rules? In fact, trait objects are nothing more than an object that links to a concrete type instance and its methods. Perfomring dynamic dispatch means that we will at runtime look inside our trait object and follow the pointer to the object's methods to know which one to call. This has the benefit of not generating any more static code when compiling, but has a runtime cost.

# Conclusion

<!-- TODO: table to explain how to choose a type, benchmark ? -->