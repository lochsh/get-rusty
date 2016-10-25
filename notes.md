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

A large part of how Rust achieves memory safety is through the concept of 
ownership. This is an exciting feature, a vital part of Rust that allows it to 
be both memory-safe _and_ efficient. All objects have an owner at all times, which is 
tracked by the compiler. The owner can give out references to other users, with 
some restrictions, or it can transfer the ownership of the variable. Crucially, 
there can only be one owner. The Rust compiler ensures that there is _exactly 
one_ binding to any given value at a time.


#### Ownership in practice
Consider this short code snippet from the [Rust 
docs](https://doc.rust-lang.org/stable/book/ownership.html#the-details)

```rust
let v = vec![1, 2, 3];
let mut v2 = v;
v2.truncate(2);
```
The first line allocates memory for the vector object `v` on the stack, and 
allocates memory on the heap for the data, `[1, 2, 3]`. Rust copies the 
address of this heap allocation to an internal pointer, which is part of the 
vector object.

We have a vector object in one place (the stack), and its data in another (the 
heap). The two parts must agree at all times, e.g. with regards to length.

Now let's consider the second line of code. Here, we move `v` to `v2`. A 
shallow copy is performed: Rust does a bitwise copy of the vector object `v` 
into the stack allocation represented by `v2`. This shallow copy does _not_ 
create a copy of the heap allocation containing the vector's actual data.

If both `v` and `v2` both point to the same data, what happens if we change 
that data, as we do in the third line?  The vector object `v` would have out of 
date information about its data, and become invalid. We would have introduced
a data race.

To prevent this occurring, Rust enforces its rule of each value having _exactly 
one_ binding at any given time. Once we move the vector to `v2`, `v` is no 
longer accessible. Trying to use it will result in a compiler error:

```rust
let v = vec![1, 2, 3];
let mut v2 = v;
println!("{:?}", v);
```

```
error[E0382]: use of moved value: `v`
 --> <anon>:4:22
   |
   |2 |     let mut v2 = v;
   |  |         ------ value moved here
   |  3 |     println!("{:?}", v);
   |    |                      ^ value used here after move
```

Preventing memory problems like the one described above is important: at best, 
we can get a segmentation fault; at worst, we could allow an unauthorised user 
to read memory they shouldn't have access to.


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


#### Memory safety at zero run-time cost
As we have seen, Rust's ownership and borrowing model completely prevents data 
races. This, along with compile-time checks for array bounds &c., ensures 
complete memory safety. That means freedom from buffer overflow, dangling 
pointers and use-after-free. Rust's key innovation is enforcing these checks at 
_compile time_, meaning they do not present any run-time cost. We get memory 
safety, without overhead. This completely overturns the old trade-off between 
safety and speed/control.

We have now covered Rust's first core principle; what about the others?


### Concurrency without data races
Concurrency has always presented a lot of challenges to programmers, sometimes 
with devastating consequences.  Most of us will know of the famous Therac-25 
disaster, where concurrent programming errors (most notably a data race) 
resulted in radiotherapy patients being given radiation poisoning, and several 
consequent deaths.

We have already seen how Rust makes data races impossible, already paving the 
way for fearless concurrency.  Concurrent programming in Rust is not something 
I have much experience in, but I think it's promising that reliable concurrency 
is a core design principle of Rust.


### Abstraction without overhead

Abstraction without overhead is a design principle shared with C++.  As C++'s 
[originator](http://www.stroustrup.com/) put it:

> What you don’t use, you don’t pay for. And further: What 
you do use, you couldn’t hand code any better.

In Rust, _traits_ are a large part of how it achieves this.   ["The trait 
system gives Rust the ergonomic, expressive feel of high-level languages while 
retaining low-level control over code execution and data 
representation."](https://blog.rust-lang.org/2015/05/11/traits.html)

#### So, what is a trait?

Traits are interfaces: they specify the expectations that one piece of code has 
on another, allowing each to be switched out independently.  Hooray for 
modularity and flexibility!  Traits specify this interface through _methods_:

```rust
trait Hash {
    fn hash(&self) -> u64;
}


impl Hash for bool {
    fn hash(&self) -> u64 {
        if *self {0} else {1}
    }
}

impl Hash for i64 {
    fn hash(&self) -> u64 {
        *self as u64
    }
}
```
Here, we define a trait called `Hash`, and say that any type implementing this 
trait _must_ have the method `hash`.  Later, we implement this trait for the 
types `bool` and `i64`, by defining this method for this particular type.

Traits allow for _generic programming_:

```rust
fn print_hash<T: Hash>(t: &T) {
    println!("The hash is {}", t.hash())
}
```
This function `print_hash` is generic over type `T`.  It can take any type that 
implements the `Hash` trait!  As with C++ templates, the compiler will now 
generate a copy of `print_hash` for every type implementing the `Hash` trait. 
When we actually call `t.hash()`, at runtime, there is no cost.  Abstraction, 
without overhead!

One of the notable differences between this and C++ templates, is that clients 
of traits are fully type-checked in advance, _once_.  In C++, the code is 
checked repeatedly when applied to concrete types.  Rust's way means clearer 
and earlier compilation errors, something template meta-programming in C++ is 
notorious for its shortcomings in.

### The Rust community and tools

I hope I've now given you a good feel for what Rust is like as a language. I 
think the community and the development tools they have created is also 
something worth shouting about, and something that gives me hope that Rust will 
continue to grow both as a language and in popularity.
