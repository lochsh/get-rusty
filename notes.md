# Let's get rusty!
## Fast and safe systems programming with Rust
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
* then it grabs a series of bits from some place dictated by the wacky 
  interpretation,
* it blithely assumes that these bits are meant to be a number (not even a 
character),
* it multiplies that practically-random number by 37, and
then reports the result, all without any hint of a problem.



<!--
        * Why do people use them?  For a variety of reasons, but in large part 
          because they give runtime efficiency, and control.

        * For a long time, it has seemed we have to choose between this control 
          and safety.  Languages like Java use garbage collection to ensure 
          memory safety, but this is slow.  Java takes away a lot of the 
          control C and C++ give.

        * With Rust, we no longer have to choose between safety and control.

    * What is Rust?  Is it just like C but safe?

-->
