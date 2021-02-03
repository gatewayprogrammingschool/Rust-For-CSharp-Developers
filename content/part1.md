+++
title = "The .NET Developer's Guide to Rust"
description = "Or How to Get There From Here"

date ="2021-02-02"
updated ="2021-02-02"

# The weight as defined on the Section page of the documentation.
# If the section variable `sort_by` is set to `weight`, then any page that lacks a `weight`
# will not be rendered.
weight = 1000

# A draft page is only loaded if the `--drafts` flag is passed to `zola build`, `zola serve` or `zola check`.
draft = false

# The path the content will appear at.
# If set, it cannot be an empty string and will override both `slug` and the filename.
# The sections' path won't be used.
# It should not start with a `/` and the slash will be removed if it does.
path = "1"

in_search_index = true
template = "page.html"
[extra]
external_links_target_blank = true
author="The Sharp Ninja"

+++

## Get Ready!

C# has curly braces and owes its origins to C.  Rust also has curly braces and owes its origins to C.  There are even some shared keywords such `struct`, `for` and `while`.

*And that's where the similarities end.*

You must clear your mind now and be prepared to reorient for Rust.  This doesn't mean that Rust will be totally alien and make you rethink your life as a C# developer.  *Well, maybe it does.*

## The Journey Begins

I have been wanting to learn Rust for quite a while now.  I've jealously eyed how [Rust often overtakes C++ in performance](https://1drv.ms/x/s!At7D0T4Uz1vyjM8AlDMCWKyIYS1cOQ?e=Lv5fnM) while C# lags behind the "fast three (C/C++/Rust)" in benchmarks.  Every time C# gets faster, so does Rust.  It's not fair!

So I'm learning Rust.  I'm doing ["The Rust Programming Language" on Udemy](https://www.udemy.com/share/101Z1IAEETdF9SQXUD/).  It's a great course that is very thorough and moves quickly enough to keep your interest.  It does not assume you have any experience in a specific language, but does assume you are a programmer already and that you understand the purpose of each element that is being presented to you.  

This article is going to focus on the differences between C# and Rust as I understand them from doing the course and working with some code on my own.

## The Community

First, I want to point out that the people in the Rust community have been fantastic to me so far.  No question has been left unanswered and no snarky answers or comments have come my way.  This is amazing to me for a very specific reason.  Rust is a big-brain language.  I can see where being effective with it and coding efficiently with it requires planning and consideration of the long term ramifications of every decision you make.  You honestly get the sense that Rust's followers are in it for the right reasons: they want people to write *safe*, *secure* and *well performing* code, no matter what.  This has been consistent whether you are on the official Rust forums, Reddit or Discord.

## Getting Set Up

 This is probably the topic that draws the most disagreement among Rustaceans.  Half of them like to use Visual Studio Code + Rust Analyzer.  The other half like Jet Brains IDEs (Usually CLion) + the Rust Extension.  I went with CLion (by Jet Brains) with the Rust Extension.  If you have used Rider then this is a no brainer, the UI and Debugging tools are nearly identical, and I'm already fluent in Rider.  If you have a preference for either debugger than stick with that because the debugger will ultimately light up the bulbs for you more than any tutorial.

There's also a Visual Studio 2019 Extension for Rust.  I haven't used it yet, mainly because the speaker in the Video Series uses it (alternating with IntelliJ) about half the time and the extension as presented in the videos is very incomplete with poor hover text and error messages that are incomplete.  Rust, more than any other language, gives you very explicit and useful error messages, and their community takes great pride in that.  C# devs are accustomed to having the best tooling and documentation.  Although Rust lacks a dedicated IDE, the tooling with Jet Brains IDEs and Visual Studio Code are top-notch, and Rust's documentation rivals Microsoft's documentation for .NET Framework (we all know the docs for Dotnet Core have taken a step backwards from the days of physical MSDN Library Discs).

## Ecosystem

Beyond tooling, the availability and visibility of frameworks makes a huge difference in the day-to-day life of a developer.  C# developers currently revel in what is hands-down the best ecosystem to work in.  20 years of quality and maturity is hard to compete with when you are both new and fundamentally different.  Rust has some very impressive elements in its ecosystem, too.  [Actix Web](https://actix.rs/) is one of the [fastest web frameworks anywhere](https://www.techempower.com/benchmarks/), rivaling [C++'s Drogon](https://github.com/an-tao/drogon) in raw performance on benchmarks.  [Yew](https://github.com/yewstack/yew) is the most mature WASM library for creating Web clients, but interestingly the [Rustaceans have been warning me](https://users.rust-lang.org/t/old-programmer-thats-new-to-rust/54739/11?u=sharpninja) to wait and not use it for a production project and to use TypeScript instead.  This, of course, blows my mind as a C# dev as I see ANY functioning WASM library as an escape from JavaScript crap frameworks.

The last major area of the ecosystem I've looked into is ORMs and it seems to me that [Diesel](https://diesel.rs/) is the most complete.  It fits in somewhere between EF Core and Dapper.  It's ORM features far surpass those of Dapper, but it's migration tools require more manual SQL DDL than EF Core requires.  The documentation for Diesel warns of ensuring that your DDL exactly matches your entities or queries will blow up.  Frankly, full round trip engineering for DDL has been a normal feature of ORM systems since the mid 2000's, including Entity Framework.

## Language Paradigms

C# is first and foremost an Object Oriented Programming Language in the mold of C++.  In C#, everything is an object of some kind, even simple types such as `int` or `decimal`.  The explanation for this is that *everything* in C# is a **closure**.  Of course, as you add members to objects you are adding layers of closures with scoping and accessibility modifiers that determine visibility on each member.  The difference between a `struct` and a `class` is only a matter of how the compiler treats memory access to the root closure that is created with the `struct `being treated as a Value type and the `class` being treated as a Reference type.  In C# we have a Garbage Collector that keeps track of what variables point to a Reference-type closure.  When that count reaches zero, then the closure is marked for deletion.  For value types, there is only one reference at all time.  When that reference falls out of scope, the value is destroyed.  Every time you use a value type, it is copied on the stack and that copy is referenced.

Rust is very different.  Rust introduces the concepts of Ownership, Borrowing and Timeline.  Only one variable can own a value in Rust, regardless of what it is.  The value can be Borrowed by other scopes, but ownership remains with a single variable.  When a variable that Owns data falls out of scope, it's Timeline ends and the data is destroyed.  If there were borrowed references to this destroyed data then they would become invalid.  But Rust doesn't let that happen.  The compiler tracks all borrowed references and if there are outstanding borrowed references when the owner falls out of scope, then the compiler throws a very descriptive error.

```rust
fn main() {
    // Declare a mutable vector
    let vec = &mut vec![&1,&2,&3];
	// vec is the owner
    
    // pass it to a function
    do_something(vec);

    // display the result
    println!("{:?}", vec);
}

// Function to accept mutable vector
pub fn do_something(vec: &mut Vec<&u32>)
{
    // vec is borrowing the Vector
    let a = 100;
    // a owns 100
    
    // Add another value to the vector
    vec.push(&a);
    // a is now borrowed by vec
}
```

This code will not compile because `a` in `do_something` owns the value placed in the vector and when `do_something` falls out of scope the value will be destroyed.  However, that would leave the vector with invalid data and so the compiler refuses to do this.

```rust
error[E0597]: `a` does not live long enough
  --> src/main.rs:14:14
   |
11 | pub fn do_something(vec: &mut Vec<&u32>)
   |                                   - let's call the lifetime of this reference `'1`
...
14 |     vec.push(&a);
   |     ---------^^-
   |     |        |
   |     |        borrowed value does not live long enough
   |     argument requires that `a` is borrowed for `'1`
15 | }
   | - `a` dropped here while still borrowed
```

This does two things:

1. No garbage collector is necessary because all memory is destroyed when its owner falls out of scope.
2. It is impossible to have a pointer to invalid data, or a pointer that references invalid memory.

So with Rust, we eliminate the biggest security flaw with C/C++ and the biggest performance penalty of C#.  AND, we get incredibly useful error messages as well.

```rust
pub fn do_something(vec: &mut Vec<&u32>)
{
    vec.push(&100);

    let a = vec[vec.len()-1];

    println!("a: {}", a);
}
```

By changing things around, we make `vec` the owner of the new value and borrow it locally, thus giving us our expected result.

```rust
a: 100
[1, 2, 3, 100]

Process finished with exit code 0
```

* [Invalid Code](https://play.rust-lang.org/?version=stable&mode=debug&edition=2018&gist=6b59d6bee45f621a77ed7ea01bce3bfe) in Rust Playground
* [Working Code](https://play.rust-lang.org/?version=stable&mode=debug&edition=2018&gist=89e7027f1040904f9ca75d6305084d63) in Rust Playground

## Types

### Numbers

Rust and C# support the same basic numeric types (with one exception).

| Type                                   | Rust    | C#       |
| -------------------------------------- | ------- | -------- |
| Boolean                                | `bool`  | `bool`   |
| 8-bit byte                             | `u8`    | `byte`   |
| Signed 8-bit byte                      | `i8`    | `sbyte`  |
| 16-bit word                            | `i16`   | `short`  |
| Unsigned 16-bit word                   | `u16`   | `ushort` |
| 32-bit double-word                     | `i32`   | `int`    |
| Unsigned 32-bit double word            | `u32`   | `uint`   |
| 64-bit long-word                       | `i64`   | `long`   |
| Unsigned 64-bit long-word              | `u64`   | `ulong`  |
| Native Integer                         | `isize` | `nint`   |
| Unsigned Native Integer                | `usize` | `nuint`  |
| Single-Precision Floating Point 32-bit | `f32`   | `float`  |
| Double-Precision Floating Point 64-bit | `f64`   | `double` |

You'll notice that `decimal` is not listed.  Rust does not currently include a lossless decimal type.  There are crates (the equivalent of NuGet packages) that provide this functionality, but until one becomes part of the language you should be very careful about handling money in Rust code.

### Strings

`Strings` are something that .NET Framework nailed on day 1.  Since the beginning of C#, `Strings` have been internally represented as full Unicode (UTF-16, UTF-32 didn't exist when C# was created) character arrays that are immutable.  The `StringBuilder` class has always been around for doing proper string manipulation from the beginning.  `Strings` in C# are always a reference type, meaning you never get direct access to it, all you can do is request a copy of it or feed the reference to something else.  Operations on strings in C# always result in a new `string` (except when using `StringBuilder`, which is why `StringBuilder` exists).

In Rust, there are **two** string types.  The first, declared as `&str` is very much like a .NET Framework `string`.  String literals are always of this type.  

```rust
let my_string = "My String";
// my_string has type of &str	
```

One major difference is that internally, Rust represents strings with UTF-8 instead of Unicode (UTF-16).  This means that strings are more memory efficient in Rust at the expense of higher overhead for interpreting individual characters.  Whereas you can calculate a character in a C# string from an offset into the string's memory, with Rust you must first tokenize the byte array to a char array because UTF-8 is a variable-sized construct where a character may be defined by one to four bytes.

The second type in Rust is `String`, which actually behaves like the C# `StringBuilder` class.  If you are assembling strings on the fly you use this type.  One major difference between C# and Rust is that the output of `ToString()` and `to_string()` are opposites.  The .NET `ToString()` method on all objects returns an immutable `string`.  The `to_string()` method found on most Rust objects returns `String`, not the immutable `&str`.  As a matter of fact, you can get a `String` from a `&str` in two ways.

```rust
let mut s = "Template {{0}}".to_string();
let mut s = String::from("Template {{0}}");
```

### String Formatting

One of the best features of .NET is string formatting and string interpolation.  C# has always rich support for formatting strings using the `string.Format` method and passing formatters to the `ToString` method of numeric types.  Now, interpolated strings allow using a formatted string anywhere that accepts a string.

Things are a little different in Rust.  Instead of using a simple method of the `String` object to format strings, you use a system supplied macro called `format!` which outputs a `&str`.  The syntax is familiar, though.

```rust
let name = "The Sharp Ninja";
let s = format!("My name is {}.", name);
```

The variable `s` is an immutable `&str` containing `My name is The Sharp Ninja.`

Writing to the console is similarly performed with a macro named `println!`

```rust
// Output values multiple times
println!("My name is {name}, but you can call me {name}.", name);

// Pretty Pring a Range
println!("Range: {:?}", (1..11).collect::<Vec<_>>());
```

> Note: The formatting macros always require a format string followed by any values to place in the format string.  So `println!(s)` is not valid, but `println!("{}", s)` is.

## Collections

`Vector` is the primary collection used in Rust.  Vectors are strongly typed and Rust's strong type system is normally used to define the type of a `Vector's` contents via generics.

```rust
// Create a vector of &str values
let mut v = vec!("ABC", "def");
println!("{:#?}", v);
```

```
OUTPUT: 

[
    "ABC",
    "def",
]
```

Notice that I added the `mut` keyword to the vector's declaration.  This means that the vector is mutable and thus more values can be added.  Without this, attempting to add to the vector generates a compile-time error.

```rust
// Create a vector of &str values
let mut v = vec!("ABC", "def");
println!("{:?}", v);
v.push("HIJK");
println!("{:?}", v);
v.push("lmno");
println!("{:?}", v);
v.push("P");
println!("{:?}", v);
```

Yields

```
["ABC", "def"]
["ABC", "def", "HIJK"]
["ABC", "def", "HIJK", "lmno"]
["ABC", "def", "HIJK", "lmno", "P"]
```

The `Vector` acts like a stack as well.  Notice we place new items with `push`?  We can `pop` items from the vector as you would expect from a stack.

```rust
let mut v = vec!("ABC", "def");
println!("{:?}", v);
v.push("HIJK");
println!("{:?}", v);
v.push("lmno");
println!("{:?}", v);
v.push("P");
println!("{:?}", v);

let mut last = v.pop();
println!("{:?} - {:?}", v, last);
last = v.pop();    
println!("{:?} - {:?}", v, last);
last = v.pop();    
println!("{:?} - {:?}", v, last);
last = v.pop();    
println!("{:?} - {:?}", v, last);
last = v.pop();    
println!("{:?} - {:?}", v, last);
```

Yields

```
["ABC", "def"]
["ABC", "def", "HIJK"]
["ABC", "def", "HIJK", "lmno"]
["ABC", "def", "HIJK", "lmno", "P"]
["ABC", "def", "HIJK", "lmno"] - Some("P")
["ABC", "def", "HIJK"] - Some("lmno")
["ABC", "def"] - Some("HIJK")
["ABC"] - Some("def")
[] - Some("ABC")
```

Wait a minute!  What's with the `Some("P")`?  That is one of the great features of Rust, `Option<T>`, which is baked into the entire language.  We haven't talked about `null` handling so far because in Rust there is no `null`.  Quite simply, if you have a method where not returning a value is a valid option, then the return type of the method must be `Option<T>` and if there is no value to return, you return `None`, otherwise you return `Some<T>`.  What would happen if we `pop` one more time from an empty `Vector`?

```
["ABC", "def"]
["ABC", "def", "HIJK"]
["ABC", "def", "HIJK", "lmno"]
["ABC", "def", "HIJK", "lmno", "P"]
["ABC", "def", "HIJK", "lmno"] - Some("P")
["ABC", "def", "HIJK"] - Some("lmno")
["ABC", "def"] - Some("HIJK")
["ABC"] - Some("def")
[] - Some("ABC")
[] - None
```

Now we don't have to worry about `null` values causing errors, or attempting to reference a null pointer.  It's impossible.  Notice there's no value type on `None` just for this reason.  We'll talk more about `Option<T>` when we discussion program flow control.

### Iterators

In .NET, all collections implement `IEnumerable`, even arrays.  `IEnumerable` is the basis for all iterating of collections via `foreach` and Linq.  Rust does have a direct analogous concept to `IEnumerable`, it is the `IntoIterator` trait.   Let's look at `IntoIterator` in action.

```rust
let values = vec![1, 2, 3, 4, 5];

for x in values {
    println!("{}", x);
}
```

`Values` is a `Vector`, which implements the trait `IntoIterator`.  What's a trait?  Traits are like interfaces in C#.  They define contracts that objects may implement, or objects may use the default implementation of the trait, like default implementations in C# 9 interfaces.  The difference is that traits are implemented independently of the object.  In C#, classes implementing `IEnumerable` must implement `GetEnumerator()` in their class.  In Rust, Traits are implemented on a separate scope using the `impl` keyword.

```rust
impl Iterator for Counter {
    // we will be counting with usize
    type Item = usize;

    // next() is the only required method
    fn next(&mut self) -> Option<Self::Item> {
        // Increment our count. This is why we started at zero.
        self.count += 1;

        // Check to see if we've finished counting or not.
        if self.count < 6 {
            Some(self.count)
        } else {
            None
        }
    }
}	
```

Here we have an object named `Counter`, which has been previously defined in a `struct`.  We are implementing the `Iterator` trait on `Counter` (the return type of the `into_iter()` method from the `IntoIterator` trait) which adds the `next` method.  Notice our implementation has it's own instance variable?  This implementation is a closure that becomes "part" of the `Counter` object.  Just like objects in .NET are composed of closures, so are Rust objects as well.

However, traits are different from C# Extension Methods because they are part of the instance of the object thet they are attached to whereas Extension Methods are always static and you must pass the instance to them and they can only access the public members of the attached object.

## Member Accessibility

A very important point for code hygeine and code security is the accessibility of members of objects.

| C# | Rust | What it does |
|---|---|---|
| `public` | `pub` | Makes the object or member accessible to all other scopes. |
| `protected` | N/A | Makes the object or member accessible to sub-types.  Rust does not have this concept because there is no direct inheritence. |
| `internal` | `pub(crate)` | Makes the object or member accessible to all other scopes within the same binary library (assembly or crate). |
| `private` | N/A | Makes the object or member accessible only within the scope of the containing object.  Rust does not have this concept. |
| N/A | No Modifier | Makes the object or member accessible only within the Module containing the object.  C# does not have this concept. |

Essentially, in Rust the concept of a module is analogous to a namespace in C#, but where the concept of `private` applies to the individual objects of the namespace, in Rust the lack of a modifier scopes acessibility to the entirity of the module that the object or function is defined in.