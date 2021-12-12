+++
title = "Six Nice Things About Rust" 
date = 2021-12-11
[taxonomies] 
tags = ["Rust"] 
+++ 
From talking to a couple people about Rust, it seems Rust can have a bit of a reputation as an obscure and difficult language.  Here is my take: After the initial learning phase, Rust is an ergonomic language and a load of fun to work with, despite its requirements of having no garbage collection and to be free of data races and memory errors. Here are some aspects
to the language that make Rust nice to work with:

## 1. No null

From this decision flows much of the flavor of Rust. Consider this function in typescript:

```ts
// returns index of first match if found, else null
function findLetter(word: string, letter: string) -> number | null {
  // ...snip...
}
```

Using it would looks like:

```ts
const index = findLetter("Hello world", "w");

// null check
if (index) {
  console.log("found character at index:", index);
}
```

In Rust, such a function would look like this:

```rs
fn find_index(word: &str, sought_char: char) -> Option<number> {
  // ...snip...
}
```

and using it would look like:

```rs
if let Some(index) = find_index("Hello world".to_string(), 'w') {
  println!("Found character at index {}", index);
}
```

Any language that is upfront about when objects are null, such as Typescript in
strict mode or C# 8 and up, are nice to work with, and mitigate the pain
of having null in a system.

But Rust is one example that shows we can avoid null all together without using
any ergonomics, even though we're adding more types to the system. This is
because Rust makes plenty use of:

## 2. Pattern Matching

Our `find_index` function returns a value of type [Option\<number\>](https://doc.rust-lang.org/std/option/), which is defined as so:

```rs
enum Option<T> { 
    Some(T), 
    None 
}
```

To consume this value, we can deconstruct it, much like can deconstruct in other languages. The above `if let` is one example. Another is a `match` statement:

```rs
match find_letter("Hello world".to_string(), 'w') {
  Some(i) => println!("Found char at index {}:", i),
  None => println!("Did not find char")
};
```

Notice that for match statements, compilation will fail if you don't handle
every member of the enum.

We can even handle more complicated examples:

```rs
pub enum QuestionMarkBox {
   Empty,
   Money(u32),
   PowerUp {
       effect: String,
       amount: String,
   }
}

struct Mario {}

impl Mario {
    fn open_box(&self, qmbox: QuestionMarkBox) -> String {
       match qmbox {
           QuestionMarkBox::Empty =>  "shucks!".to_string(),
           QuestionMarkBox::Money(amt) => format!("I got ${}!!!", amt),
           QuestionMarkBox::PowerUp{
               effect,
               amount,
           } => format!("Received {} effect for +{}.", effect, amount)
       }
    }
}
```

Additionally, types like Option  from the standard library also come with some nice utility functions:

```rs
// Return the Some(T) value, else stop and exit the program
let index = find_letter("Hello world".to_string(), 'w').unwrap();
// Return the Some(T) value, else use this value instead
let index = find_letter("Hello world".to_string(), 'w').unwrap_or(42);
// Returns true if Some, else false
let found_index = find_letter("Hello world".to_string(), 'w').is_some();
```

Rust's liberal use of enums and pattern matching also informs another important
aspect to the language:

## 3. Error Handling

Here is a sample of the most common error handling trick, try-catch, in C#
([source](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/file-system/how-to-read-from-a-text-file)):

```cs
class ReadFromFile
{
    static void Main()
    {
        string text = "";
        try
        {
            text = System.IO.File.ReadAllText(@"./file");
        }
        catch
        {
            System.Console.WriteLine("Something went wrong reading the file");
        }

        System.Console.WriteLine("Contents of files.txt = {0}", text);
    }
}
```

In VSCode, if you hover over
[ReadAllText](https://docs.microsoft.com/en-us/dotnet/api/system.io.file.readalltext?view=net-6.0),
you get a nice little description of all the Errors that can be raised if you
call this function. But it raises the question if we did not have that
documentation, would we know it could throw it all?

Rust leaves no doubt. Rust models recoverable errors in the
[Result<T,E>](https://doc.rust-lang.org/std/result/index.html) enum:

```rs
enum Result<T, E> {
   Ok(T),
   Err(E),
}
```

Just like the Option type, callers will have to explicitly handle success and
failure cases. Conveniently, many of the utility methods that applied to Option apply to Result. `Result` also has the "?" operator, which tells Rust to exit early with error if result of operation is an error:

```rs
// example from https://doc.rust-lang.org/std/result/index.html
fn write_info(info: &Info) -> io::Result<()> {
    let mut file = File::create("my_best_friends.txt")?;
    // Early return on error
    file.write_all(format!("name: {}\n", info.name).as_bytes())?;
    file.write_all(format!("age: {}\n", info.age).as_bytes())?;
    file.write_all(format!("rating: {}\n", info.rating).as_bytes())?;
    Ok(())
}
```

Speaking of errors, we can avoid a whole class of errors thank to...

## 4. The Borrow Checker

The borrow checker is both the most notorious and the the most consequential thing about Rust.

During compilation the borrow checker makes sure that every value in your code has either one mutable reference or multiple immutable references. If there are zero references,
the value is dropped.

This is an essential feature for writing correct concurrent code. But it is also helpful in single-threaded code.
Consider this maybe surprising effect of having two mutable references in Typescript:

```ts
const goodTwin = { is: "good" };
const evilTwin = goodTwin;
evilTwin.is = "evil";

// good or evil???
console.log("The good twin:", goodTwin);
```

Even though we declare goodTwin as a constant variable, and do not mutate
goodTwin directly, goodTwin becomes evil because we gave a reference to
evilTwin, who mutated the object (should have used Object.freeze).

If we try this is Rust:

```rs
let good_twin =  Twin { is: "good".to_string() };
let mut evil_twin = good_twin;
evil_twin.is = "evil".to_string();

println!("Good twin: {:?}", good_twin);
```

We get this build error:

```md
error[E0382]: borrow of moved value: `good_twin`
  --> src/twins.rs:13:31
   |
9  |   let good_twin =  Twin { is: "good".to_string() };
   |       --------- move occurs because `good_twin` has type `Twin`, which does not implement the `Copy` trait
10 |   let mut evil_twin = good_twin;
   |                  --------- value moved here
...
13 |   println!("Good twin: {:?}", good_twin);
   |                               ^^^^^^^^^ value borrowed here after moved
```

When good_twin gives a reference to evil_twin, good_twin gives up its reference. Good twin is no longer a valid reference, and now we don't have to deal with competing sources of what the value is.

To achieve the same thing we did in Javascript, we would have to declare
`good_twin` as mutable, and pass an explicitly mutable reference to `evil_twin`:

```rs
let mut good_twin =  Twin{ is: "good".to_string() };
let evil_twin = &mut good_twin;
evil_twin.is = "evil".to_string();
```

Perhaps a better example of the power of the borrow checker is this classic
mistake. In Python:

```py
numbers = [1, 2, 8, 3, 4, 5]

for num in numbers:
   if num % 2 == 0:
      numbers.remove(num)

print(numbers)
```

We're mutating a list while we iterate over it. If you run the sample, the eight, an even number,
is not removed from the list.

Let's try again in Rust:

```rs
let mut list = vec![1, 2, 8, 8, 1];

for (i, num) in list.iter().enumerate() {
    if num % 2 == 0 {
        list.remove(i);
    }
}
```

The compilation fails with this error:

```md
error[E0502]: cannot borrow `list` as mutable because it is also borrowed as immutable
 --> src/abusing_lists.rs:7:13
  |
5 |     for (i, num) in list.iter().enumerate() {
  |                     -----------------------
  |                     |
  |                     immutable borrow occurs here
  |                     immutable borrow later used here
6 |         if num % 2 == 0 {
7 |             list.remove(i);
  |             ^^^^^^^^^^^^^^ mutable borrow occurs here

F
```

As long as the iterator itself has a reference to the list, no other reference can mutate the list.

The borrow checker is the novel thing about Rust. The next thing is not as novel, but is just as crucial for feeling at home in Rust:

## 5. Traits (Interfaces)

Traits are interfaces that allow default implementations (think recent C#). They
are used in all the places you would use interfaces: as parameter and return
types, as bounds on generic parameters, as super and sub traits of other traits,
and so on. Since Rust does not support inheritance for structs, traits do the work of
sharing code.

This may be old hat for experienced C# developer, but I'm having a lot of fun
using traits to fit in my custom types into the language with traits:

Want to create a custom display type for your type to show users? There is a
trait for that:

```rs
use std::fmt::{Display, Formatter};

pub struct JabberWocky {
  face: String,
  body: char,
}

impl Display for JabberWocky {
    fn fmt(&self, f: &mut Formatter<'_>) -> Result<(), std::fmt::Error> {
        write!(f, "{}-{}=<", self.face, self.body)
    }
}

#[test]
fn test_jabberwocky_display() {
    let jb = JabberWocky { face: '👹'.to_string(), body: '🦎'};
    assert_eq!(format!("{}", jb), "👹-🦎=<");
}
```

Want to override the plus sign for your type? Trait:

```rs
impl Add for JabberWocky {
    type Output = Self;

    fn add(self, rhs: Self) -> Self::Output {
        let heads = self.face + &rhs.face;
        Self { face: heads, body: self.body }
    }
}

#[test]
fn test_jabberwocky_add() {
  let joe = JabberWocky { face: '👹'.to_string(), body: '🦎'};
  let bob = JabberWocky { face: '🦊'.to_string(), body: '🐋'};

  let joe_bob = joe + bob;

  assert_eq!(format!("{}", joe_bob), "👹🦊-🦎=<");
}
```

Want to create conversions of another type to your type? First try it:

```rs
impl From<(char, char)> for JabberWocky { }

#[test]
fn test_jabberwocky_from_tuple() {
    let hollis: JabberWocky = ('👽','🦗').into();
    assert_eq!(format!("{}", hollis), "👽-🦗=<");
}
```

and read the error message:

```md
error[E0277]: the trait bound `JabberWocky: From<(char, char)>` is not satisfied
  --> src/jabberwocky.rs:27:41
   |
27 |     let hollis: JabberWocky = ('👽','🦗').into();
   |                                           ^^^^ the trait `From<(char, char)>` is not implemented for `JabberWocky`
   |
   = note: required because of the requirements on the impl of `Into<JabberWocky>` for `(char, char)`
```

and then do what it says:

```rs
impl From<(char, char)> for JabberWocky {
    fn from(tuple: (char, char)) -> Self { 
        Self { face: tuple.0.to_string(), body: tuple.1 }
    }
}
```

Want to compare your structs? If all struct members implement the PartialEq trait, you can simply derive the PartialEq trait, and compare by comparing all struct members:

```rs
#[derive(PartialEq)]
pub struct JabberWocky {
    face: String,
    body: char,
}

#[test]
fn test_equal() {
    assert!(JabberWocky::from(('🐸', '🐋')) == JabberWocky::from(('🐸', '🐋')));
    assert!(JabberWocky::from(('🐸', '🐋')) != JabberWocky::from(('🧟', '🫀')));
}
```

Want to add default functionality to your type from a third party crate? Just bring the
trait in scope. Here is an Advent of Code solution thanks to the
Itertools::tuple_windows function:

```rs
use itertools::Itertools;

fn main() {
    println!("{:?}", part_2());
}

fn part_2() -> u32 {
    include_str!("input.txt")
        .lines()
        .map(str::parse::<u32>)
        .map(Result::unwrap)
        .tuple_windows::<(_, _, _)>()
        .map(|window| window.0 + window.1 + window.2)
        .tuple_windows::<(_, _)>()
        .fold(
            0,
            |acc, (first, second): (u32, u32)| {
                if first < second {
                    acc + 1
                } else {
                    acc
                }
            },
        )
}
```

Where can you find all this information about traits about the standard library
and other libraries? This is all readily accessible thanks to Rust's great story
around...

## 6. Documentation

Documentation is not a beloved part about programming. For example, Kent Beck dedicates a few paragraphs in his `Extreme Programming Explained` to explain why keeping up documentation is a burden for development teams without much benefit.

Rust changes the balance of that equation.

First, any Rust project can make an HTML site of documentation by running
`cargo doc --open`

At a minimum, the documentation will have all the function signatures of the
public functions, traits, and structs of your modules. It will also put any
comments with `///` or `//!` in there as well. It will also have
links to documentation to all your dependencies.

You can even include code snippets to show how to use your library. Those code snippets can even automatically be run as tests with `cargo test`!

All libraries hosted on crates.io automatically have their documentation hosted
on docs.rs.

## Conclusion

If a project performance requirements are strict enough to rule out more mainstream languages, and the economic and staffing realities of the client can make room for including a new language, I would not rule out Rust in fear of its complexity. Despite its steep learning curve (we've just scratched the surface here), underneath is a real gem of a language that is a joy to use.
