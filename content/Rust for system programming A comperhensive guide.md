+++
title = "Rust for Systems Programming: A Comprehensive Guide"
date = 2025-12-18
+++

## Introduction

Coming from C and systems programming, Rust will feel both familiar and radically different. Rust offers the same low-level control as C but with modern safety guarantees enforced at compile time. This guide focuses on systems programming concepts in Rust, helping you leverage your C experience while mastering Rust's unique approach to memory safety, concurrency, and performance.

## Why Rust for Systems Programming?

Rust eliminates entire categories of bugs that plague C programs:
- **No null pointer dereferences**: The type system prevents null without runtime overhead
- **No data races**: The borrow checker ensures thread safety at compile time
- **No use-after-free**: Ownership rules prevent dangling pointers
- **No buffer overflows**: Bounds checking with zero-cost abstractions

All this while maintaining C-level performance and direct hardware access.

## Ownership and Borrowing: The Core Innovation

This is Rust's most important concept. Understanding it deeply is crucial for systems programming. Ownership is Rust's solution to memory management without garbage collection.

### Ownership Rules: Deep Dive

The three fundamental rules:

1. **Each value has a single owner** - Only one variable owns a piece of data at any time
2. **When the owner goes out of scope, the value is dropped** - Automatic cleanup, like C++ RAII
3. **Values can be moved or borrowed, but never copied implicitly** - Explicit about expensive operations

#### Move Semantics

When you assign one variable to another, ownership moves:

```rust
fn main() {
    let s1 = String::from("hello");  // s1 owns the string on the heap
    let s2 = s1;                      // Ownership moved to s2
    
    // println!("{}", s1);            // ERROR: value used after move
    println!("{}", s2);               // OK: s2 is the new owner
}  // s2 dropped here, memory freed automatically
```

**What happens in memory:**
```
Before move:
s1 → [ptr, len, cap] → heap: "hello"

After move:
s1 → [invalidated]
s2 → [ptr, len, cap] → heap: "hello"
```

Compare to C, where this would be unclear:
```c
char *s1 = strdup("hello");  // s1 points to heap
char *s2 = s1;               // Now both point to same memory
free(s1);                    // Free the memory
printf("%s", s2);            // Undefined behavior! Dangling pointer
```

#### Copy vs Move

Some types implement `Copy` trait (stack-only data like integers):

```rust
fn main() {
    let x = 5;        // i32 is Copy
    let y = x;        // x is copied, not moved
    
    println!("x: {}, y: {}", x, y);  // Both valid!
    
    // But String is NOT Copy
    let s1 = String::from("hello");
    let s2 = s1;      // s1 is moved
    // println!("{}", s1);  // ERROR
}
```

**Types that are Copy:**
- All integer types (i32, u64, etc.)
- Boolean (bool)
- Floating point (f32, f64)
- Character (char)
- Tuples of Copy types: (i32, i32)

**Types that are NOT Copy (they own heap data):**
- String
- Vec<T>
- Box<T>
- Any type with heap allocation

#### Clone: Explicit Deep Copy

When you need a deep copy, use `.clone()`:

```rust
fn main() {
    let s1 = String::from("hello");
    let s2 = s1.clone();  // Explicit deep copy
    
    println!("s1: {}, s2: {}", s1, s2);  // Both valid!
}
```

This makes expensive operations visible in code, unlike C where `strcpy` might not be obvious.

#### Ownership in Functions

Functions take ownership unless you pass references:

```rust
fn takes_ownership(s: String) {  // s comes into scope
    println!("{}", s);
}  // s goes out of scope, memory freed

fn makes_copy(x: i32) {  // i32 is Copy
    println!("{}", x);
}  // x goes out of scope, but nothing special happens (Copy type)

fn main() {
    let s = String::from("hello");
    takes_ownership(s);  // s moved into function
    // println!("{}", s);  // ERROR: s no longer valid
    
    let x = 5;
    makes_copy(x);      // x copied into function
    println!("{}", x);  // OK: x still valid
}
```

#### Return Values and Ownership

Functions can return ownership:

```rust
fn gives_ownership() -> String {
    let s = String::from("hello");
    s  // s moved out to caller
}

fn takes_and_gives_back(s: String) -> String {
    s  // s moved back to caller
}

fn main() {
    let s1 = gives_ownership();        // s1 gets ownership
    let s2 = String::from("world");    
    let s3 = takes_and_gives_back(s2); // s2 moved in, returned value to s3
    
    println!("{} {}", s1, s3);
    // println!("{}", s2);  // ERROR: s2 was moved
}
```

### Borrowing: References Without Ownership Transfer

Borrowing solves the problem of passing data without losing ownership.

#### Immutable References

```rust
fn calculate_length(s: &String) -> usize {  // s is a reference
    s.len()  // Can read through reference
    // s.push_str("!"); // ERROR: can't modify through immutable reference
}  // s goes out of scope, but we don't own the String, so nothing happens

fn main() {
    let s1 = String::from("hello");
    let len = calculate_length(&s1);  // Pass a reference
    println!("Length of '{}' is {}", s1, len);  // s1 still valid!
}
```

**Memory diagram:**
```
Stack:
s1  → [ptr, len, cap] → heap: "hello"
&s1 → points to s1 (just an address)
```

#### Multiple Immutable Borrows

You can have as many immutable references as you want:

```rust
fn main() {
    let s = String::from("hello");
    
    let r1 = &s;  // First immutable borrow
    let r2 = &s;  // Second immutable borrow
    let r3 = &s;  // Third immutable borrow
    
    println!("{}, {}, {}", r1, r2, r3);  // All valid!
}
```

This is safe because none of them can modify the data - no race conditions possible.

#### Mutable References

To modify borrowed data, use mutable references:

```rust
fn append_world(s: &mut String) {  // Mutable reference
    s.push_str(", world");  // Can modify through mutable reference
}

fn main() {
    let mut s = String::from("hello");  // Must be mutable
    append_world(&mut s);  // Pass mutable reference
    println!("{}", s);     // Prints: hello, world
}
```

#### The One Mutable Reference Rule

**Critical rule:** You can have either ONE mutable reference OR any number of immutable references, but not both:

```rust
fn main() {
    let mut s = String::from("hello");
    
    let r1 = &s;      // Immutable borrow
    let r2 = &s;      // Another immutable borrow - OK
    println!("{} {}", r1, r2);  // r1 and r2 last used here
    
    // let r3 = &mut s;  // ERROR: can't borrow as mutable while immutable borrows exist
    
    let r3 = &mut s;  // OK now: immutable borrows ended
    r3.push_str("!");
    println!("{}", r3);
}
```

**Why this rule exists:** Prevents data races at compile time:

```rust
// This would be a data race in C, but Rust prevents it
let mut data = vec![1, 2, 3];
let r1 = &data;           // Reader
// let r2 = &mut data;    // ERROR: can't have writer while reader exists
// r2.push(4);            // Would invalidate r1!
println!("{:?}", r1);     // r1 guaranteed to be valid
```

#### Non-Lexical Lifetimes (NLL)

Modern Rust is smart about when borrows end:

```rust
fn main() {
    let mut s = String::from("hello");
    
    let r1 = &s;
    let r2 = &s;
    println!("{} {}", r1, r2);  // r1 and r2 last used here
    // Borrows r1 and r2 end here, even though they're still in scope
    
    let r3 = &mut s;  // OK: no other borrows are active
    r3.push_str(" world");
}
```

#### Dangling References Are Impossible

Rust prevents dangling pointers at compile time:

```rust
fn dangle() -> &String {  // ERROR: missing lifetime specifier
    let s = String::from("hello");
    &s  // We're returning a reference to s
}  // s goes out of scope and is dropped
   // Reference would be dangling!

// Correct version: return ownership
fn no_dangle() -> String {
    let s = String::from("hello");
    s  // Move ownership out
}
```

Compare to C:
```c
char* dangle() {
    char s[] = "hello";
    return s;  // Compiles, but s is on stack - dangling pointer!
}
```

#### Borrowing in Data Structures

```rust
struct User {
    username: String,
    email: String,
}

fn get_username(user: &User) -> &str {
    &user.username  // Return reference to part of User
}

fn main() {
    let user = User {
        username: String::from("alice"),
        email: String::from("alice@example.com"),
    };
    
    let name = get_username(&user);
    println!("Username: {}", name);  // Both user and name valid
}
```

### Lifetimes: Explicit Relationship Between References

Lifetimes are Rust's way of ensuring references are valid. Every reference has a lifetime.

#### Implicit Lifetimes

Most of the time, Rust infers lifetimes:

```rust
fn first_word(s: &str) -> &str {
    // Compiler infers: fn first_word<'a>(s: &'a str) -> &'a str
    s.split_whitespace().next().unwrap_or("")
}
```

The compiler knows: "The returned reference lives as long as the input reference."

#### Explicit Lifetime Annotations

When the compiler can't infer, you must be explicit:

```rust
fn longest<'a>(x: &'a str, y: &'a str) -> &'a str {
    if x.len() > y.len() { x } else { y }
}
```

**What `'a` means:** "The returned reference will be valid as long as BOTH x and y are valid."

```rust
fn main() {
    let string1 = String::from("long string");
    let result;
    
    {
        let string2 = String::from("xyz");
        result = longest(string1.as_str(), string2.as_str());
        println!("{}", result);  // OK: string2 still alive
    }
    
    // println!("{}", result);  // ERROR: string2 dropped, result invalid
}
```

#### Multiple Lifetimes

Different parameters can have different lifetimes:

```rust
fn first_half<'a, 'b>(x: &'a str, _y: &'b str) -> &'a str {
    // Return value only depends on x's lifetime
    let mid = x.len() / 2;
    &x[..mid]
}

fn main() {
    let string1 = String::from("hello world");
    let result;
    
    {
        let string2 = String::from("xyz");
        result = first_half(string1.as_str(), string2.as_str());
    }  // string2 dropped
    
    println!("{}", result);  // OK: result only depends on string1
}
```

#### Lifetime Elision Rules

Rust has rules to infer lifetimes automatically:

1. Each reference parameter gets its own lifetime
2. If there's one input lifetime, it's assigned to all output lifetimes
3. If there are multiple input lifetimes and one is `&self`, output gets `self`'s lifetime

```rust
// Written:
fn foo(s: &str) -> &str

// Compiler infers:
fn foo<'a>(s: &'a str) -> &'a str

// Written:
fn bar(x: &str, y: &str) -> &str  // ERROR: can't infer which lifetime

// Must write:
fn bar<'a>(x: &'a str, y: &'a str) -> &'a str
```

#### Lifetimes in Structs

Structs can hold references, but need lifetime annotations:

```rust
struct ImportantExcerpt<'a> {
    part: &'a str,  // This reference must live at least as long as the struct
}

fn main() {
    let novel = String::from("Call me Ishmael. Some years ago...");
    let first_sentence = novel.split('.').next().unwrap();
    
    let excerpt = ImportantExcerpt {
        part: first_sentence,
    };
    
    println!("{}", excerpt.part);
}  // excerpt dropped, then novel - OK order
```

**Invalid usage:**
```rust
fn main() {
    let excerpt;
    {
        let novel = String::from("Call me Ishmael.");
        let first = novel.split('.').next().unwrap();
        excerpt = ImportantExcerpt { part: first };
    }  // ERROR: novel dropped, but excerpt.part still references it
    
    // println!("{}", excerpt.part);
}
```

#### The `'static` Lifetime

`'static` means a reference lives for the entire program:

```rust
let s: &'static str = "hello";  // String literals have 'static lifetime
```

This is stored in the binary, not on the heap or stack.

#### Complex Lifetime Example

```rust
struct Context<'a> {
    data: &'a str,
}

impl<'a> Context<'a> {
    fn get_data(&self) -> &'a str {  // Explicit: returns with 'a lifetime
        self.data
    }
    
    fn process<'b>(&'a self, input: &'b str) -> &'a str {
        // 'a: lifetime of Context
        // 'b: lifetime of input
        // We return 'a because we return self.data
        println!("Processing: {}", input);
        self.data
    }
}

fn main() {
    let data = String::from("context data");
    let ctx = Context { data: &data };
    
    {
        let input = String::from("temporary");
        let result = ctx.process(&input);
        // result has lifetime 'a (tied to ctx), not 'b (tied to input)
    }  // input dropped
    
    let result = ctx.get_data();
    println!("{}", result);  // Still valid: tied to ctx and data
}
```

#### Lifetime Bounds

You can specify that a lifetime must outlive another:

```rust
struct Ref<'a, T: 'a> {  // T must live at least as long as 'a
    data: &'a T,
}

fn print_ref<'a, T>(r: &'a Ref<'a, T>)
where
    T: std::fmt::Display + 'a,
{
    println!("{}", r.data);
}
```

## Memory Management Without Garbage Collection

### Stack vs Heap in Rust

Like C, Rust has explicit stack and heap allocation, but with automatic cleanup.

```rust
// Stack allocation (like C's automatic variables)
let x = 5;               // i32 on stack - Copy type
let arr = [1, 2, 3, 4];  // Fixed-size array on stack

// Heap allocation (like C's malloc, but automatic free)
let v = Vec::new();      // Vector on heap, automatic cleanup
let s = String::from("hello");  // String on heap
let b = Box::new(5);     // Box: explicit heap allocation

// When v, s, b go out of scope, they're automatically freed
```

**Memory layout comparison:**

In C:
```c
int x = 5;              // Stack
int *ptr = malloc(sizeof(int) * 10);  // Heap - must manually free
free(ptr);              // Easy to forget or double-free
```

In Rust:
```rust
let x = 5;              // Stack
let v = Vec::with_capacity(10);  // Heap - automatically freed when v dropped
// No manual free needed!
```

### Smart Pointers: Safe Memory Management

Smart pointers provide the power of C pointers with safety guarantees. They implement `Deref` and `Drop` traits to behave like references while managing memory.

#### Box<T>: Owned Heap Allocation

`Box<T>` is the simplest smart pointer - owned heap allocation.

**Basic usage:**
```rust
fn main() {
    let b = Box::new(5);  // Allocate i32 on heap
    println!("b = {}", b);
}  // Box automatically freed here
```

**Use cases:**

1. **Large data you want to move without copying:**
```rust
struct LargeStruct {
    data: [u8; 10000],
}

fn main() {
    let large = Box::new(LargeStruct { data: [0; 10000] });
    let moved = large;  // Only moves pointer, not 10KB of data
}
```

2. **Recursive types (required for known size at compile time):**
```rust
// This won't compile - infinite size!
// struct Node {
//     value: i32,
//     next: Node,  // ERROR: recursive type has infinite size
// }

// This works - Box has known size (just a pointer)
struct Node {
    value: i32,
    next: Option<Box<Node>>,  // Box breaks the recursion
}

impl Node {
    fn new(value: i32) -> Box<Self> {
        Box::new(Node { value, next: None })
    }
    
    fn append(&mut self, value: i32) {
        match self.next {
            Some(ref mut next) => next.append(value),
            None => self.next = Some(Box::new(Node { value, next: None })),
        }
    }
}

fn main() {
    let mut list = Node::new(1);
    list.append(2);
    list.append(3);
    // Entire list freed automatically when list dropped
}
```

Compare to C where you'd need:
```c
struct Node {
    int value;
    struct Node *next;
};

// Manual cleanup required
void free_list(struct Node *node) {
    while (node) {
        struct Node *next = node->next;
        free(node);
        node = next;
    }
}
```

3. **Trait objects for dynamic dispatch:**
```rust
trait Drawable {
    fn draw(&self);
}

struct Circle { radius: f64 }
struct Square { side: f64 }

impl Drawable for Circle {
    fn draw(&self) {
        println!("Drawing circle");
    }
}

impl Drawable for Square {
    fn draw(&self) {
        println!("Drawing square");
    }
}

fn main() {
    let shapes: Vec<Box<dyn Drawable>> = vec![
        Box::new(Circle { radius: 5.0 }),
        Box::new(Square { side: 3.0 }),
    ];
    
    for shape in &shapes {
        shape.draw();  // Dynamic dispatch
    }
}  // All shapes automatically freed
```

#### Rc<T>: Reference Counting for Shared Ownership

`Rc` (Reference Counted) allows multiple owners of the same data. Similar to `shared_ptr` in C++.

**When to use:** When you need multiple parts of your program to read the same data, and you can't determine at compile time which part will finish last.

```rust
use std::rc::Rc;

fn main() {
    let data = Rc::new(vec![1, 2, 3]);
    println!("Count: {}", Rc::strong_count(&data));  // 1
    
    let data2 = Rc::clone(&data);  // Increment reference count
    println!("Count: {}", Rc::strong_count(&data));  // 2
    
    let data3 = Rc::clone(&data);
    println!("Count: {}", Rc::strong_count(&data));  // 3
    
    {
        let data4 = Rc::clone(&data);
        println!("Count: {}", Rc::strong_count(&data));  // 4
    }  // data4 dropped, count decremented
    
    println!("Count: {}", Rc::strong_count(&data));  // 3
}  // All Rc instances dropped, data freed when count reaches 0
```

**Important:** `Rc` is NOT thread-safe. Use `Arc` for multithreading.

**Graph example - multiple parents:**
```rust
use std::rc::Rc;

struct Node {
    value: i32,
    children: Vec<Rc<Node>>,
}

fn main() {
    let leaf = Rc::new(Node {
        value: 3,
        children: vec![],
    });
    
    let branch1 = Rc::new(Node {
        value: 5,
        children: vec![Rc::clone(&leaf)],  // Shared ownership
    });
    
    let branch2 = Rc::new(Node {
        value: 10,
        children: vec![Rc::clone(&leaf)],  // Both branches share leaf
    });
    
    println!("Leaf ref count: {}", Rc::strong_count(&leaf));  // 3
}  // Everything freed in correct order automatically
```

**Weak references to avoid cycles:**
```rust
use std::rc::{Rc, Weak};
use std::cell::RefCell;

struct Node {
    value: i32,
    parent: RefCell<Weak<Node>>,  // Weak to avoid cycle
    children: RefCell<Vec<Rc<Node>>>,
}

fn main() {
    let leaf = Rc::new(Node {
        value: 3,
        parent: RefCell::new(Weak::new()),
        children: RefCell::new(vec![]),
    });
    
    println!("Leaf strong count: {}", Rc::strong_count(&leaf));  // 1
    println!("Leaf weak count: {}", Rc::weak_count(&leaf));      // 0
    
    {
        let branch = Rc::new(Node {
            value: 5,
            parent: RefCell::new(Weak::new()),
            children: RefCell::new(vec![Rc::clone(&leaf)]),
        });
        
        *leaf.parent.borrow_mut() = Rc::downgrade(&branch);
        
        println!("Branch strong count: {}", Rc::strong_count(&branch));  // 1
        println!("Leaf strong count: {}", Rc::strong_count(&leaf));      // 2
        println!("Leaf weak count: {}", Rc::weak_count(&leaf));          // 0
        println!("Branch weak count: {}", Rc::weak_count(&branch));      // 1
    }  // branch dropped, no memory leak!
    
    println!("Leaf parent: {:?}", leaf.parent.borrow().upgrade());  // None
}
```

#### Arc<T>: Atomic Reference Counting (Thread-Safe)

`Arc` is like `Rc` but safe for concurrent access using atomic operations.

```rust
use std::sync::Arc;
use std::thread;

fn main() {
    let data = Arc::new(vec![1, 2, 3, 4, 5]);
    let mut handles = vec![];
    
    for i in 0..3 {
        let data = Arc::clone(&data);  // Clone the Arc, not the data
        let handle = thread::spawn(move || {
            // Each thread has shared read access
            println!("Thread {}: sum = {}", i, data.iter().sum::<i32>());
        });
        handles.push(handle);
    }
    
    for handle in handles {
        handle.join().unwrap();
    }
    
    println!("Original still valid: {:?}", data);
}  // Data freed when last Arc dropped
```

**Performance note:** `Arc` uses atomic operations (more expensive than `Rc`), so use `Rc` for single-threaded code.

**Real-world example - shared configuration:**
```rust
use std::sync::Arc;
use std::thread;

struct Config {
    max_connections: usize,
    timeout_ms: u64,
    server_name: String,
}

fn worker(id: usize, config: Arc<Config>) {
    // Each worker thread reads shared config
    println!(
        "Worker {}: max_conn={}, timeout={}, server={}",
        id, config.max_connections, config.timeout_ms, config.server_name
    );
}

fn main() {
    let config = Arc::new(Config {
        max_connections: 100,
        timeout_ms: 5000,
        server_name: String::from("MyServer"),
    });
    
    let mut handles = vec![];
    for i in 0..5 {
        let config = Arc::clone(&config);
        let handle = thread::spawn(move || worker(i, config));
        handles.push(handle);
    }
    
    for handle in handles {
        handle.join().unwrap();
    }
}
```

#### RefCell<T>: Interior Mutability

`RefCell` provides runtime borrow checking. Use when you need mutability in an immutable context.

**The problem it solves:**
```rust
// This doesn't work - can't mutate through immutable reference
struct Counter {
    count: i32,
}

impl Counter {
    fn increment(&self) {  // Takes &self, not &mut self
        // self.count += 1;  // ERROR: can't mutate
    }
}
```

**Solution with RefCell:**
```rust
use std::cell::RefCell;

struct Counter {
    count: RefCell<i32>,  // Interior mutability
}

impl Counter {
    fn new() -> Self {
        Counter {
            count: RefCell::new(0),
        }
    }
    
    fn increment(&self) {  // Takes &self
        let mut count = self.count.borrow_mut();  // Runtime mutable borrow
        *count += 1;
    }
    
    fn get(&self) -> i32 {
        *self.count.borrow()  // Runtime immutable borrow
    }
}

fn main() {
    let counter = Counter::new();
    counter.increment();
    counter.increment();
    println!("Count: {}", counter.get());  // 2
}
```

**Runtime borrow checking:**
```rust
use std::cell::RefCell;

fn main() {
    let data = RefCell::new(5);
    
    {
        let r1 = data.borrow();  // OK
        let r2 = data.borrow();  // OK - multiple immutable borrows
        println!("{} {}", r1, r2);
    }  // Borrows end
    
    {
        let mut m1 = data.borrow_mut();  // OK - mutable borrow
        *m1 += 1;
        // let m2 = data.borrow_mut();   // PANIC at runtime: already borrowed
    }  // Mutable borrow ends
    
    println!("{}", data.borrow());  // OK
}
```

**Combining Rc and RefCell for shared mutable state:**
```rust
use std::rc::Rc;
use std::cell::RefCell;

#[derive(Debug)]
struct Node {
    value: i32,
    children: Vec<Rc<RefCell<Node>>>,
}

fn main() {
    let leaf = Rc::new(RefCell::new(Node {
        value: 3,
        children: vec![],
    }));
    
    let branch = Rc::new(RefCell::new(Node {
        value: 5,
        children: vec![Rc::clone(&leaf)],
    }));
    
    // Modify leaf through shared reference
    leaf.borrow_mut().value = 10;
    
    println!("Leaf: {:?}", leaf.borrow());
    println!("Branch: {:?}", branch.borrow());
}
```

**Mock object example:**
```rust
use std::cell::RefCell;

trait Messenger {
    fn send(&self, msg: &str);
}

struct MockMessenger {
    sent_messages: RefCell<Vec<String>>,  // Mutable in immutable struct
}

impl MockMessenger {
    fn new() -> Self {
        MockMessenger {
            sent_messages: RefCell::new(vec![]),
        }
    }
}

impl Messenger for MockMessenger {
    fn send(&self, msg: &str) {  // &self, not &mut self
        self.sent_messages.borrow_mut().push(String::from(msg));
    }
}

#[cfg(test)]
mod tests {
    use super::*;
    
    #[test]
    fn test_messenger() {
        let messenger = MockMessenger::new();
        messenger.send("Hello");
        messenger.send("World");
        
        assert_eq!(messenger.sent_messages.borrow().len(), 2);
    }
}
```

#### Cell<T>: Simple Interior Mutability for Copy Types

For `Copy` types, `Cell` is simpler than `RefCell`:

```rust
use std::cell::Cell;

struct Counter {
    count: Cell<i32>,
}

impl Counter {
    fn increment(&self) {
        let current = self.count.get();
        self.count.set(current + 1);
    }
    
    fn get(&self) -> i32 {
        self.count.get()
    }
}

fn main() {
    let counter = Counter { count: Cell::new(0) };
    counter.increment();
    counter.increment();
    println!("Count: {}", counter.get());
}
```

**No runtime overhead** - `Cell` just moves values in and out.

#### Choosing the Right Smart Pointer

Decision tree:

1. **Single owner, heap allocated?** → `Box<T>`
2. **Multiple owners, single-threaded?** → `Rc<T>`
3. **Multiple owners, multi-threaded?** → `Arc<T>`
4. **Need interior mutability with runtime checks?** → `RefCell<T>` (or `Mutex<T>` for threads)
5. **Need interior mutability for Copy types?** → `Cell<T>`

**Combinations:**
- `Rc<RefCell<T>>` - Multiple owners, shared mutation (single-threaded)
- `Arc<Mutex<T>>` - Multiple owners, shared mutation (multi-threaded)
- `Arc<RwLock<T>>` - Multiple owners, many readers or one writer (multi-threaded)

**Memory overhead:**
- `Box<T>`: One pointer (8 bytes on 64-bit)
- `Rc<T>`: Pointer + 2 counters (strong, weak) ≈ 24 bytes
- `Arc<T>`: Pointer + 2 atomic counters ≈ 24 bytes
- `RefCell<T>`: Value + borrow flag ≈ size of T + 8-16 bytes

**Real-world combination example:**
```rust
use std::rc::Rc;
use std::cell::RefCell;

struct Database {
    data: RefCell<Vec<String>>,
}

struct Connection {
    db: Rc<RefCell<Database>>,
}

impl Connection {
    fn insert(&self, value: String) {
        self.db.borrow_mut().data.borrow_mut().push(value);
    }
    
    fn query(&self) -> Vec<String> {
        self.db.borrow().data.borrow().clone()
    }
}

fn main() {
    let db = Rc::new(RefCell::new(Database {
        data: RefCell::new(vec![]),
    }));
    
    let conn1 = Connection { db: Rc::clone(&db) };
    let conn2 = Connection { db: Rc::clone(&db) };
    
    conn1.insert(String::from("Alice"));
    conn2.insert(String::from("Bob"));
    
    println!("{:?}", conn1.query());  // ["Alice", "Bob"]
}
```

## Zero-Cost Abstractions

Rust's mantra: "What you don't use, you don't pay for. And what you do use, you couldn't hand code any better."

### Generics: Compile-Time Polymorphism

Unlike C where void pointers or macros are needed, Rust has zero-cost generics:

```rust
// Generic function
fn swap<T>(a: &mut T, b: &mut T) {
    std::mem::swap(a, b);
}

// Generic struct
struct Point<T> {
    x: T,
    y: T,
}

impl<T> Point<T> {
    fn new(x: T, y: T) -> Self {
        Point { x, y }
    }
}

// Compiler generates specialized versions at compile time
let mut a = 5;
let mut b = 10;
swap(&mut a, &mut b);  // Generates swap::<i32>

let p1 = Point::new(1, 2);      // Point<i32>
let p2 = Point::new(1.0, 2.0);  // Point<f64>
```

No runtime cost - this is monomorphization, like C++ templates but with better error messages.

### Traits: Compile-Time Interfaces

Traits are like interfaces but resolved at compile time:

```rust
// Define a trait
trait Drawable {
    fn draw(&self);
    fn area(&self) -> f64;
}

struct Circle {
    radius: f64,
}

struct Rectangle {
    width: f64,
    height: f64,
}

// Implement trait for types
impl Drawable for Circle {
    fn draw(&self) {
        println!("Drawing circle with radius {}", self.radius);
    }
    
    fn area(&self) -> f64 {
        std::f64::consts::PI * self.radius * self.radius
    }
}

impl Drawable for Rectangle {
    fn draw(&self) {
        println!("Drawing rectangle {}x{}", self.width, self.height);
    }
    
    fn area(&self) -> f64 {
        self.width * self.height
    }
}

// Generic function using trait bounds
fn print_area<T: Drawable>(shape: &T) {
    shape.draw();
    println!("Area: {}", shape.area());
}

// Or use trait objects for dynamic dispatch
fn print_area_dyn(shape: &dyn Drawable) {
    shape.draw();
    println!("Area: {}", shape.area());
}
```

Static dispatch (generics) has zero runtime cost. Dynamic dispatch (trait objects) has the same cost as C function pointers.

### Enums and Pattern Matching

Rust's enums are algebraic data types, far more powerful than C enums:

```rust
enum IpAddr {
    V4(u8, u8, u8, u8),
    V6(String),
}

let home = IpAddr::V4(127, 0, 0, 1);
let loopback = IpAddr::V6(String::from("::1"));

// Pattern matching is exhaustive - compiler ensures you handle all cases
match home {
    IpAddr::V4(a, b, c, d) => {
        println!("IPv4: {}.{}.{}.{}", a, b, c, d);
    }
    IpAddr::V6(addr) => {
        println!("IPv6: {}", addr);
    }
}
```

### Option and Result: No Null, No Exceptions

Rust has no null pointers. Instead:

```rust
// Option<T>: either Some(T) or None
fn find_item(items: &[i32], target: i32) -> Option<usize> {
    for (i, &item) in items.iter().enumerate() {
        if item == target {
            return Some(i);
        }
    }
    None
}

// Result<T, E>: either Ok(T) or Err(E)
fn divide(a: i32, b: i32) -> Result<i32, String> {
    if b == 0 {
        Err(String::from("Division by zero"))
    } else {
        Ok(a / b)
    }
}

// Pattern matching for error handling
match divide(10, 2) {
    Ok(result) => println!("Result: {}", result),
    Err(e) => println!("Error: {}", e),
}

// Or use the ? operator for propagation
fn calculate() -> Result<i32, String> {
    let a = divide(10, 2)?;  // Returns early if Err
    let b = divide(20, 4)?;
    Ok(a + b)
}
```

This is checked at compile time - no forgotten error checks like in C.

## Unsafe Rust: When You Need Low-Level Control

Rust has an escape hatch for systems programming scenarios where safety can't be proven:

```rust
unsafe {
    // Five superpowers unlocked:
    // 1. Dereference raw pointers
    let mut num = 5;
    let r1 = &num as *const i32;
    let r2 = &mut num as *mut i32;
    
    println!("r1: {}", *r1);  // Must be in unsafe block
    *r2 = 10;
    
    // 2. Call unsafe functions
    unsafe fn dangerous() {
        // Low-level operations
    }
    dangerous();
    
    // 3. Access/modify mutable static variables
    static mut COUNTER: i32 = 0;
    COUNTER += 1;
    
    // 4. Implement unsafe traits
    // 5. Access fields of unions
}
```

### FFI: Calling C from Rust

Interoperability with C is crucial for systems programming:

```rust
// Declare external C functions
extern "C" {
    fn abs(input: i32) -> i32;
    fn malloc(size: usize) -> *mut u8;
    fn free(ptr: *mut u8);
}

fn main() {
    unsafe {
        let x = abs(-42);
        println!("Absolute value: {}", x);
        
        // Manual memory management when interfacing with C
        let ptr = malloc(100);
        if !ptr.is_null() {
            // Use the memory
            free(ptr);
        }
    }
}

// Expose Rust functions to C
#[no_mangle]
pub extern "C" fn rust_function(x: i32) -> i32 {
    x * 2
}
```

### Raw Pointers for Low-Level Operations

```rust
fn main() {
    let mut num = 5;
    
    // Create raw pointers (safe)
    let r1 = &num as *const i32;
    let r2 = &mut num as *mut i32;
    
    unsafe {
        // Dereference raw pointers (unsafe)
        println!("r1: {}", *r1);
        *r2 = 10;
        println!("num: {}", num);
    }
}
```

### Implementing Low-Level Data Structures

```rust
use std::alloc::{alloc, dealloc, Layout};
use std::ptr;

struct MyVec<T> {
    ptr: *mut T,
    len: usize,
    capacity: usize,
}

impl<T> MyVec<T> {
    fn new() -> Self {
        MyVec {
            ptr: ptr::null_mut(),
            len: 0,
            capacity: 0,
        }
    }
    
    fn push(&mut self, value: T) {
        if self.len == self.capacity {
            self.grow();
        }
        
        unsafe {
            ptr::write(self.ptr.add(self.len), value);
        }
        self.len += 1;
    }
    
    fn grow(&mut self) {
        let new_capacity = if self.capacity == 0 { 1 } else { self.capacity * 2 };
        let new_layout = Layout::array::<T>(new_capacity).unwrap();
        
        let new_ptr = if self.capacity == 0 {
            unsafe { alloc(new_layout) as *mut T }
        } else {
            let old_layout = Layout::array::<T>(self.capacity).unwrap();
            unsafe {
                std::alloc::realloc(
                    self.ptr as *mut u8,
                    old_layout,
                    new_layout.size(),
                ) as *mut T
            }
        };
        
        self.ptr = new_ptr;
        self.capacity = new_capacity;
    }
}

impl<T> Drop for MyVec<T> {
    fn drop(&mut self) {
        if self.capacity != 0 {
            unsafe {
                let layout = Layout::array::<T>(self.capacity).unwrap();
                dealloc(self.ptr as *mut u8, layout);
            }
        }
    }
}
```

## Concurrency Without Data Races

Rust's ownership system prevents data races at compile time.

### Threads

```rust
use std::thread;
use std::time::Duration;

fn main() {
    let handle = thread::spawn(|| {
        for i in 1..10 {
            println!("Thread: {}", i);
            thread::sleep(Duration::from_millis(1));
        }
    });
    
    for i in 1..5 {
        println!("Main: {}", i);
        thread::sleep(Duration::from_millis(1));
    }
    
    handle.join().unwrap();
}
```

### Message Passing with Channels

```rust
use std::sync::mpsc;
use std::thread;

fn main() {
    let (tx, rx) = mpsc::channel();
    
    thread::spawn(move || {
        let vals = vec![
            String::from("hi"),
            String::from("from"),
            String::from("thread"),
        ];
        
        for val in vals {
            tx.send(val).unwrap();
        }
    });
    
    for received in rx {
        println!("Got: {}", received);
    }
}
```

### Shared State with Mutex

```rust
use std::sync::{Arc, Mutex};
use std::thread;

fn main() {
    let counter = Arc::new(Mutex::new(0));
    let mut handles = vec![];
    
    for _ in 0..10 {
        let counter = Arc::clone(&counter);
        let handle = thread::spawn(move || {
            let mut num = counter.lock().unwrap();
            *num += 1;
        });
        handles.push(handle);
    }
    
    for handle in handles {
        handle.join().unwrap();
    }
    
    println!("Result: {}", *counter.lock().unwrap());
}
```

The compiler ensures you can't have data races - if this compiles, it's thread-safe.

### Atomic Operations

```rust
use std::sync::atomic::{AtomicUsize, Ordering};
use std::sync::Arc;
use std::thread;

fn main() {
    let counter = Arc::new(AtomicUsize::new(0));
    let mut handles = vec![];
    
    for _ in 0..10 {
        let counter = Arc::clone(&counter);
        let handle = thread::spawn(move || {
            for _ in 0..1000 {
                counter.fetch_add(1, Ordering::SeqCst);
            }
        });
        handles.push(handle);
    }
    
    for handle in handles {
        handle.join().unwrap();
    }
    
    println!("Result: {}", counter.load(Ordering::SeqCst));
}
```

## Systems Programming Patterns

### Error Handling in Systems Code

```rust
use std::fs::File;
use std::io::{self, Read};

fn read_config(path: &str) -> Result<String, io::Error> {
    let mut file = File::open(path)?;
    let mut contents = String::new();
    file.read_to_string(&mut contents)?;
    Ok(contents)
}

// Custom error types
use std::fmt;

#[derive(Debug)]
enum ConfigError {
    Io(io::Error),
    Parse(String),
}

impl fmt::Display for ConfigError {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        match self {
            ConfigError::Io(e) => write!(f, "IO error: {}", e),
            ConfigError::Parse(s) => write!(f, "Parse error: {}", s),
        }
    }
}

impl From<io::Error> for ConfigError {
    fn from(error: io::Error) -> Self {
        ConfigError::Io(error)
    }
}

impl std::error::Error for ConfigError {}
```

### Builder Pattern

```rust
struct Server {
    host: String,
    port: u16,
    timeout: u64,
    max_connections: usize,
}

struct ServerBuilder {
    host: String,
    port: u16,
    timeout: Option<u64>,
    max_connections: Option<usize>,
}

impl ServerBuilder {
    fn new(host: String, port: u16) -> Self {
        ServerBuilder {
            host,
            port,
            timeout: None,
            max_connections: None,
        }
    }
    
    fn timeout(mut self, timeout: u64) -> Self {
        self.timeout = Some(timeout);
        self
    }
    
    fn max_connections(mut self, max: usize) -> Self {
        self.max_connections = Some(max);
        self
    }
    
    fn build(self) -> Server {
        Server {
            host: self.host,
            port: self.port,
            timeout: self.timeout.unwrap_or(30),
            max_connections: self.max_connections.unwrap_or(100),
        }
    }
}

// Usage
let server = ServerBuilder::new("localhost".to_string(), 8080)
    .timeout(60)
    .max_connections(200)
    .build();
```

### Type State Pattern

Encode state in the type system to prevent invalid operations:

```rust
struct Locked;
struct Unlocked;

struct Door<State> {
    state: std::marker::PhantomData<State>,
}

impl Door<Locked> {
    fn new() -> Self {
        Door { state: std::marker::PhantomData }
    }
    
    fn unlock(self) -> Door<Unlocked> {
        println!("Door unlocked");
        Door { state: std::marker::PhantomData }
    }
}

impl Door<Unlocked> {
    fn open(self) {
        println!("Door opened");
    }
    
    fn lock(self) -> Door<Locked> {
        println!("Door locked");
        Door { state: std::marker::PhantomData }
    }
}

// Usage
let door = Door::new();        // Door<Locked>
// door.open();                // Compile error!
let door = door.unlock();      // Door<Unlocked>
door.open();                   // OK
```

## No-Std and Embedded Systems

Rust can run without the standard library for embedded systems:

```rust
#![no_std]
#![no_main]

use core::panic::PanicInfo;

#[panic_handler]
fn panic(_info: &PanicInfo) -> ! {
    loop {}
}

#[no_mangle]
pub extern "C" fn _start() -> ! {
    // Embedded system entry point
    loop {}
}
```

### Direct Hardware Access

```rust
// Memory-mapped I/O
const GPIO_BASE: usize = 0x4002_0000;

fn set_pin_high(pin: u8) {
    unsafe {
        let gpio = GPIO_BASE as *mut u32;
        let value = gpio.read_volatile();
        gpio.write_volatile(value | (1 << pin));
    }
}
```

## Performance Optimization

### Inline Assembly

```rust
use std::arch::asm;

fn add_with_asm(a: u64, b: u64) -> u64 {
    let result: u64;
    unsafe {
        asm!(
            "add {result}, {a}, {b}",
            a = in(reg) a,
            b = in(reg) b,
            result = out(reg) result,
        );
    }
    result
}
```

### SIMD Operations

```rust
#[cfg(target_arch = "x86_64")]
use std::arch::x86_64::*;

fn simd_add(a: &[f32; 4], b: &[f32; 4]) -> [f32; 4] {
    unsafe {
        let va = _mm_loadu_ps(a.as_ptr());
        let vb = _mm_loadu_ps(b.as_ptr());
        let result = _mm_add_ps(va, vb);
        
        let mut output = [0.0f32; 4];
        _mm_storeu_ps(output.as_mut_ptr(), result);
        output
    }
}
```

### Zero-Copy I/O

```rust
use std::fs::File;
use std::os::unix::io::AsRawFd;

fn sendfile_example(from: &File, to: &File, count: usize) -> std::io::Result<()> {
    unsafe {
        let result = libc::sendfile(
            to.as_raw_fd(),
            from.as_raw_fd(),
            std::ptr::null_mut(),
            count,
        );
        
        if result < 0 {
            return Err(std::io::Error::last_os_error());
        }
    }
    Ok(())
}
```

## Tooling and Ecosystem

### Cargo: Build System and Package Manager

```toml
# Cargo.toml
[package]
name = "my_project"
version = "0.1.0"
edition = "2021"

[dependencies]
serde = { version = "1.0", features = ["derive"] }
tokio = { version = "1.0", features = ["full"] }

[profile.release]
opt-level = 3
lto = true
codegen-units = 1
```

```bash
cargo build              # Build project
cargo build --release    # Optimized build
cargo test              # Run tests
cargo bench             # Run benchmarks
cargo doc --open        # Generate and open docs
```

### Testing

```rust
#[cfg(test)]
mod tests {
    use super::*;
    
    #[test]
    fn test_addition() {
        assert_eq!(2 + 2, 4);
    }
    
    #[test]
    #[should_panic(expected = "divide by zero")]
    fn test_panic() {
        let _ = 1 / 0;
    }
}
```

### Benchmarking

```rust
#![feature(test)]
extern crate test;

#[bench]
fn bench_function(b: &mut test::Bencher) {
    b.iter(|| {
        // Code to benchmark
        (0..1000).sum::<i32>()
    });
}
```

## Async/Await for Systems Programming

Modern async I/O without callbacks:

```rust
use tokio::io::{AsyncReadExt, AsyncWriteExt};
use tokio::net::TcpListener;

#[tokio::main]
async fn main() -> Result<(), Box<dyn std::error::Error>> {
    let listener = TcpListener::bind("127.0.0.1:8080").await?;
    
    loop {
        let (mut socket, _) = listener.accept().await?;
        
        tokio::spawn(async move {
            let mut buf = [0; 1024];
            
            loop {
                match socket.read(&mut buf).await {
                    Ok(0) => return,
                    Ok(n) => {
                        if socket.write_all(&buf[0..n]).await.is_err() {
                            return;
                        }
                    }
                    Err(_) => return,
                }
            }
        });
    }
}
```

## Migrating from C to Rust

### Common Patterns Translation

```rust
// C: Manual memory management
// char *s = malloc(100);
// strcpy(s, "hello");
// free(s);

// Rust: Automatic memory management
let s = String::from("hello");
// Automatically freed when out of scope

// C: Pointer arithmetic
// int arr[5] = {1,2,3,4,5};
// int *p = arr;
// int x = *(p + 2);

// Rust: Safe indexing
let arr = [1, 2, 3, 4, 5];
let x = arr[2];  // Bounds checked

// C: Function pointers
// int (*func_ptr)(int, int) = &add;
// int result = func_ptr(1, 2);

// Rust: Function pointers or closures
let func_ptr: fn(i32, i32) -> i32 = add;
let result = func_ptr(1, 2);

let closure = |a, b| a + b;
let result = closure(1, 2);
```

## Best Practices

1. **Let the compiler guide you**: Rust's error messages are excellent
2. **Fight the borrow checker initially, embrace it later**: The struggle teaches you good design
3. **Use `clippy` for linting**: `cargo clippy` catches common mistakes
4. **Use `rustfmt` for formatting**: `cargo fmt` maintains consistent style
5. **Write tests**: Rust makes testing easy and natural
6. **Read The Rust Book**: It's comprehensive and well-written
7. **Start with safe Rust**: Only use `unsafe` when necessary
8. **Leverage the type system**: Encode invariants in types, not comments

## Conclusion

Rust brings modern safety to systems programming without sacrificing performance. The learning curve is steep initially, especially the borrow checker, but the benefits are substantial:

- Memory safety without garbage collection
- Thread safety without data races
- Zero-cost abstractions
- Excellent tooling and documentation
- Growing ecosystem

Your C experience gives you a solid foundation. Focus on understanding ownership and borrowing deeply, and you'll find Rust enables writing safer, more maintainable systems code than ever before. The compiler is your ally - when your code compiles, you can be confident it's free from entire categories of bugs that plague C programs.

Start with small projects, gradually increase complexity, and don't fight the compiler - learn from it. Welcome to safe systems programming.
