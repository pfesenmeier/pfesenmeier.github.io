+++
draft = true
title = "7 Nice Things About Rust"
date = 2021-12-11

[taxonomies]
tags = ["Rust"]
+++
From talking to a couple people about Rust, it seems Rust can have a bit of a reputation as an obscure and dificult language. Here is my take: After the initial learning phase, Rust is an ergonomic language and a load of fun to work with, despite its requirements of having no garbage collection and to be free of data races and memory errors. Here are 7 things I like about Rust:

## No null
From this decision flows much of the flavor of Rust. Consider this function signature in typescript:

```typescript
// returns index of first match if found, else null
function findIndex(word: string, sought_char: string) -> number | null 
```
Use looks like:
```typescript
const index = findIndex("Hello world", "w");

if (index) {
   console.log("fonud character at index:", index);
} 
```

In Rust, such a function would look like this:

```rust
fn find_index(word: &str, sought_char: char) -> Option<number>
```
and using it would look like: 
```rust
if let Some(index) = find_index("Hello world".to_string(), 'w') {
  println!("Found character at index {}", index);
}
```
Any language that is upfront about when objects are null, such as Typescript in strict mode or C# 8 and up, are nice to work with, and mitigate most of the pain of having null in a system.

But Rust is one example that shows we can avoid null all together without using any ergonomics, even though we're adding more types to the system. This is because Rust makes plenty use of:

## Pattern Matching

Our `find_index` function returns a value of type Option<number>, which is defined as so:

enum Option<T> {
  Some(T),
  None
}

The above example `if let` syntax is nice for when you only care about the `Some` value.  Another is a `match` statement:

```rust
match find_index("Hello world".to_string(), 'w') {
  Some(i) => println!("Found char at index {}:", i),
  None => println!("Did not find char")
};
```
Notice that for match statements, compilation will fail if you don't handle every member of the enum.

Option also has some nice utility function:
// Return the Some(T) value, else stop and exit the program
let index = find_index("Hello world".to_string(), 'w').unwrap();
// Return the Some(T) value, else use this value instead
Option::unwrap_or()
// create iterator that returns Some() or None. nice for method chaining
Option::iter
And a ton of more obscure ones like
// applies fn to Some value, or returns provided default value
Option::map_or

Rust's liberal use of enums and pattern matching also informs another important aspect to the language: 

## Error Handling

Here is a sample of the most common error handling trick, try-catch, in C# ([source](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/file-system/how-to-read-from-a-text-file)):
```csharp
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

In VSCode, if you hover over [ReadAllText](https://docs.microsoft.com/en-us/dotnet/api/system.io.file.readalltext?view=net-6.0), you get a nice little description of all the Errors that can be raised if you call this function. But it raises the question if we did not have that documentation, would we know it could throw it all?

Rust leaves no doubt. Rust models recoverable errors in the [Result<T,E>](https://doc.rust-lang.org/std/result/index.html) enum:

```rust
enum Result<T, E> {
   Ok(T),
   Err(E),
}
```
Just like the Option type, callers will have to explicitly handle success and failure cases. Conveniently, many of the utility methods, such as `unwrap`. For a string of fallible calls that could error, we can simplify error handling with the "?" operator:

```rust
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

## The Borrow Checker

The borrow checker gets a bad rap sometimes. Yeah it's essential for writing concurrent code, but is it all so important in my synchronous, single-threaded script? 

To start, it might be nice to have some background of what the borrow checker is. Simply put, at compile time, the borrow checker makes sure that for every point in your code that there is either one mutable reference or multiple immutable references to every value in your code (if there are zero references, the value is automatically dropped).

Consider this maybe surprising effect of having two mutable references in Typescript:

```typescript
const goodTwin = { is: "good" };
const evilTwin = goodTwin;
evilTwin.is = "evil";

// good or evil???
console.log("The good twin:", goodTwin);
```

Even though we declare goodTwin as a constant variable, and do not mutate goodTwin directly, goodTwin becomes evil because we gave a reference to evilTwin, who mutated the object.

If we try this is Rust:
```rust
let good_twin =  Twin{ is: "good".to_string() };
let mut evil = good_twin;
evil.is = "evil".to_string();

println!("Good twin: {:?}", good_twin);
```
We get this build error:
```
error[E0382]: borrow of moved value: `good_twin`
  --> src/twins.rs:13:31
   |
9  |   let good_twin =  Twin{ is: "good".to_string() };
   |       --------- move occurs because `good_twin` has type `Twin`, which does not implement the `Copy` trait
10 |   let mut evil = good_twin;
   |                  --------- value moved here
...
13 |   println!("Good twin: {:?}", good_twin);
   |                               ^^^^^^^^^ value borrowed here after moved
```
In laymen's terms, what happen was that when evil was given a reference to good_twin's value, `good_twin` can no longer access that object. `good_twin` is no longer valid.

To acheive the same thing we did in Javascript, we would have to declare `good_twin` as mutable, and pass an explicitly mutable reference to `evil_twin`:

```rust
let mut good_twin =  Twin{ is: "good".to_string() };
let evil = &mut good_twin;
evil.is = "evil".to_string();
```

Perhaps a better example of the power of the borrow checker is this classic mistake. In Python:

```python
numbers = [1, 2, 8, 3, 4, 5]

for num in numbers:
   if num % 2 == 0:
      numbers.remove(num)

print(numbers)
```
We're mutating a list while we iterate over it. If you run the sample, the eight is not removed from the list.

Let's try again in Rust:
```rust
let mut list = vec![1, 2, 8, 8, 1];

for (i, num) in list.iter().enumerate() {
    if num % 2 == 0 {
        list.remove(i);
    }
}
```

The compilation fails with this error:

```
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
As long as the iterator itself has a reference to the list, no other refernce can mutate the list. Pretty neat! Speaking of pretty neat:

## Traits (Interfaces)

Traits are interfaces that allow default implementation (think recent C#). They are used in all the places you would use interfaces: as parameter and return types, as bounds on generic parameters, as super and sub traits of other traits, etc.. Since Rust does not support inheritance for structs, traits do the work of sharing code.

This may be old hat for experienced C# developer, but I'm having a lot of fun using traits to fit in my custom types into the language with traits:

Want to create a custom display type for your type to show users? There is a trait for that:
```rust
use std::fmt::{Display, Formatter};

pub struct JabberWocky {
  face: char,
  body: char,
}

impl Display for JabberWocky {
    fn fmt(&self, f: &mut Formatter<'_>) -> Result<(), std::fmt::Error> {
        write!(f, "{}-🫀={}-<", self.face, self.body)
    }
}

#[test]
fn test_jabberwocky_display() {
    let jb = JabberWocky { face: '👹', body: '🦎'};
    assert_eq!(format!("{}", jb), "👹-🫀=🦎-<");
}
```
Want to override the plus sign for your type? Trait:
```rust
impl Add for JabberWocky {
    type Output = Self;

    fn add(self, rhs: Self) -> Self::Output {
        let heads = self.face + &rhs.face;
        Self { face: heads, body: self.body }
    }
}

#[test]
fn test_jabberwocky_add() {
  let harry = JabberWocky { face: '👹'.to_string(), body: '🦎'};
  let frank = JabberWocky { face: '🦊'.to_string(), body: '🐋'};

  let double_jb = harry + frank;

  assert_eq!(format!("{}", double_jb), "👹🦊-🦎=<");
}
```
Want to create conversions of another type to your type? First try it:
```rust
impl From<(char, char)> for JabberWocky { }

#[test]
fn test_jabberwocky_from_tuple() {
    let hollis: JabberWocky = ('👽','🦗').into();
    assert_eq!(format!("{}", hollis), "👽-🦗=<");
}
```
and read the error message:
```
error[E0277]: the trait bound `JabberWocky: From<(char, char)>` is not satisfied
  --> src/jabberwocky.rs:27:41
   |
27 |     let hollis: JabberWocky = ('👽','🦗').into();
   |                                           ^^^^ the trait `From<(char, char)>` is not implemented for `JabberWocky`
   |
   = note: required because of the requirements on the impl of `Into<JabberWocky>` for `(char, char)`
```
and then do what it says:

```rust
impl From<(char, char)> for JabberWocky {
    fn from(tuple: (char, char)) -> Self { 
        Self { face: tuple.0.to_string(), body: tuple.1 }
    }
}
```

Want to add functionality to your type from a third party crate? Just bring the trait in scope. Here is an Advent of Code solution thanks to the Itertools::tuple_windows function:

```rust
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
Where can you find all this information about traits about the standard library and other libraries? This is all readily accessible thanks to Rust's great story around... 

## Documentation

It's not the most beloved feature. It's actually been smeared by the Xtreme Programming Book. But Rust does a lot to remove the traditional pain points.

First, any Rust project can make documentation by running
`cargo doc --open`

At a mininum, the documentation will have all the function signatures of the public functions, traits, and structs of your modules. It will also have easy links to documntation to all your dependencies. But it will also put any doc-comments (comments with `///` or `//!`) in there. You can even put code snippets. Those code snippets can even automatically be run when you run `cargo test`!

All libraries hosted on crates.io automatically have their documentation hosted on docs.rs. 

## Ecosystem

Finally, Rust already has some create libraries to use:
extra iterator methods: itertools
custom cargo commands: i.e., cargo-edit
parser generators: peg, nom

## Learning Materials

- The Rust Book
- The Embedded Discovery Book
- Programming Rust
- Rust for Rustaceans + Jon Gjengset's YouTube streams

# Conclusion

Rust's learning curve is indeed steep, but rest assured that once summitted, there are a ton of slick syntax that you've enjoyed from other languages, plus some new ones thrown in too.





