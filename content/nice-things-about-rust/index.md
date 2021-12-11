+++
draft = true
+++
# 10 Nice Things about Rust
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
let index = match find_index("Hello world".to_string(), 'w') {
  Some(i) => i,
  None => -1
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

To start, it might be nice to have some background: 


