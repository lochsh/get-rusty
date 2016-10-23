# Let's get rusty!
## Fast and safe systems programming with Rust

_Rust_ – what is it? Rust is a systems programming language as fast as C and 
C++, but without the horrible capacity for memory danger.

_Imagine_: freedom from the horrors of out-of-bounds access, use-after-free, 
data races – without sacrificing speed!

Not only does Rust guarantee memory safety, but it brings with it the joy of 
namespaces, trait-based generics, pattern matching and gasp sensible build 
systems.
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

The writing here is provocative, but for good reason.  C has some very real 
issues: undefined behaviour, weakly enforced typing, notorious lack of memory 
safety.  C++ provides a lot of different features, but is still unsafe, 
inheriting many of C's problems.  Many famous exploits are due to 
vulnerabilities made possible by these languages.  
[Stagefright](https://en.wikipedia.org/wiki/Stagefright_(bug)) is due to C++'s 
undefined behaviour for integer overflow.  [Heartbleed](http://heartbleed.com/) 
was due to C's absence of bounded arrays.

### So, why are C and C++ so popular?

I think C and C++ are popular for a variety of reasons.  I want to focus on the 
high level of control they provide, and their runtime efficiency, which are 
strong motivators for many.  For a long time, it has seemed like we have to 
make a choice between control and safety.  Languages like Java use garbage 
collection to ensure memory safety, but at the cost of control and speed.  C 
and C++ run fast and offer control, but at the expense of safety guarantees.

With Rust, we no longer have to choose between safety and control.
