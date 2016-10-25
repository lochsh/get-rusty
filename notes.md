# Let's get rusty!
## Fast and safe systems programming with Rust

_Rust_ – what is it? Rust is a systems programming language as fast as C and 
C++, but without the horrible capacity for memory danger.

_Imagine_: freedom from the horrors of out-of-bounds access, use-after-free, 
data races – without sacrificing speed!

Not only does Rust guarantee memory safety, but it brings with it the joy of 
namespaces, trait-based generics, pattern matching and sensible build systems.
<!--
Ideas for talk:
* Introduction: why C and C++ are so popular, where Rust fits in and why it is 
  exciting.
* Three pillars of design
    * Abstraction without overhead
    * Memory safety without garbage collection
    * Concurrency without data races
* Features
    * trait-based generics
    * pattern matching
    * sensible build systems!
    * namespaces
    * modules
    * "blisteringly fast"
    * type inference-->


### For motivation, I present this excerpt from Ian Barland's ["Why C and C++ are Awful Programming Languages"](http://www.radford.edu/ibarland/Manifestoes/whyC++isBad.shtml)

> Imagine you are a construction worker, and your boss tells you to connect the 
gas pipe in the basement to the street's gas main. You go downstairs, and find 
that there's a glitch; this house doesn't have a basement. Perhaps you decide 
to do nothing, or perhaps you decide to whimsically interpret your instruction 
by attaching the gas main to some other nearby fixture, perhaps the neighbor's 
air intake. Either way, suppose you report back to your boss that you're done.

> KWABOOM! When the dust settles from the explosion, you'd be guilty of 
> criminal 
negligence.

> Yet this is exactly what happens in many computer languages. In C/C++, the 
programmer (boss) can write `"house"[-1] * 37`. It's not clear what was 
intended, but clearly some mistake has been made. It would certainly be 
possible for the language (the worker) to report it, but what does C/C++ do?

> * It finds some non-intuitive interpretation of `"house"[-1]` (one which may 
>   vary 
each time the program runs!, and which can't be predicted by the programmer),
then it grabs a series of bits from some place dictated by the wacky 
interpretation,
* it blithely assumes that these bits are meant to be a number (not even a 
character),
* it multiplies that practically-random number by 37, and
then reports the result, all without any hint of a problem.

> In a world where programs control credit-card databases, car brakes, my 
personal finances, airplanes, and x-ray machines, it is criminal negligence to 
use a language with the flaws of C/C++. Even for games, browsers, and 
spreadsheets, the use of C/C++ needlessly helps inflict buggy software on the 
everyday user.

The writing here is provocative, but for good reason. C has some very real 
issues: undefined behaviour, weakly enforced typing, notorious lack of memory 
safety. C++ provides a lot of new features, but is still unsafe, and inherits 
many of C's problems. Many famous exploits are due to vulnerabilities made 
possible by these languages. [Stagefright](https://en.wikipedia.org/wiki/Stagefright_(bug)) is due to 
integer overflow in C++; [Heartbleed](http://heartbleed.com/) was due to C's 
absence of bounded arrays.


### So, why are C and C++ so popular?

I think C and C++ are popular for a variety of reasons. I want to focus on the 
high level of control they provide, and their runtime efficiency, which are 
strong motivators for many. For a long time, it has seemed like we have to 
make a choice between control and safety. Languages like Java use garbage 
collection to ensure memory safety, but at the cost of control and speed. C 
and C++ run fast and offer control, but at the expense of safety guarantees.

With Rust, we no longer have to choose between safety and control.


### What is Rust?

Rust is an exciting new language (its first stable release was in 2015, last 
year!) designed to be safe, concurrent and practical. It is useful for the 
same kind of applications as C and C++ due to its speed and high level of 
control.

Rust isn't simply a memory safe version of C++. Rust is its own language with 
its own design goals, and many features borrowed from functional programming.

To understand Rust and its design, let's move onto Rust's core principles.


### Memory safety without garbage collection

Briefly, let's discuss what garbage collection is and why it is slow. Managing 
resources in a program is about freeing up memory being used by unreachable 
objects. For example, a variable that has gone out of scope is unreachable. 

Garbage collection achieves this management by periodically checking memory to 
find unused objects, releasing their associated resources and memory.

Garbage collection basically eliminates double frees, dangling pointers, and 
some memory leaks. Important achievements!  But GC does this at an overhead, 
at the expense of resources and performance.

So, how does Rust achieve memory safety without this overhead?

#### Ownership

Rust achieves memory safety through the concept of ownership. This is an 
exciting feature, a vital part of Rust that allows it to be both
memory-safe _and_ efficient. All objects have an owner at all times, which is 
tracked by the compiler. The owner can give out references to other users, with 
some restrictions, or it can transfer the ownership of the variable. Crucially, 
there can only be one owner. The Rust compiler ensures that there is _exactly 
one_ binding to any given value at a time.


#### Ownership in practice

```rust
fn foo() {
    let x = String::from("Hello");
    let y = x;
    println!("{}", x);  // error[E0382]: use of moved value: `x`
}
```

This program doesn't compile. Why?  We have created a binding `x` to a string. 
When we create the binding `y`, we transfer ownership of that string to `y`. 
The name `x` is no longer bound, and trying to use it will cause the compiler 
to raise an error.

Note that for primitive types, like integers, this _would_ compile, as a deep 
copy is performed for convenience.


#### Borrowing

Alone, this ownership model is very restrictive; but Rust also provides a 
mechanism for _borrowing_, or referencing. Borrowing must follow these rules:

* Any borrow must last for a scope no greater than that of the owner
* You may only have one or the other of these two types of borrows, but not 
  both at the same time:
    * One or more (immutable) references to a resource
    * Exactly one mutable reference

With these restrictions, Rust makes it impossible to have a data race. A data 
race occurs when two or more pointers access the same memory location at the 
same time, at least one of them is writing, and the operations aren't 
synchronised.

We can have as many immutable references as we like without causing a data 
race, as none of them are writing to the memory location. Or, we can have 
exactly one write to the memory location, which will never be racing against 
any other operations.

In languages like C and C++, there are no such restrictions. In Rust, 
everything is immutable by default. If want to allow a value to be changed, we 
must explicitly tell the compiler that is what we intend. Attempting to change 
an immutable value gives a compiler error.  In C and C++, everything is mutable 
by default, and we can have any number of mutable references to an object.


#### Memory safety at zero run-time cost
<!--TODO haven't made it clear enough how memory safety is achieved-->
As we have seen, Rust's ownership and borrowing model ensures memory safety and 
prevents data races. Buffer overflow, dangling pointers, use-after-free are all 
impossible. Rust's key innovation is enforcing these checks at _compile time_, 
meaning they do not present any run-time cost. We get memory safety, without 
the overhead of garbage collection.
