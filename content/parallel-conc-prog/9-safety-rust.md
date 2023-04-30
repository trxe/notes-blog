---
layout: default
usemathjax: true
title: Safety in Rust
permalink: /pcp/ch9
---

# Rust

Motivation: Fearless concurrency
- Safety in Systems Programming
  1. **Guarantees Memory** safety: 
     1. No **data race**
     2. No UAF/double-free
     3. No dangling pointer
     4. No segmentation fault/null pointer access
  2. **Helps Thread** safety:
     1. No data race
     2. Easier to write race condition free
- Without performance compromise
  - No runtime environment
  - No garbage collector
  - Zero cost abstractions
    - No **global cost**: a ZCA should not hinder the performance of programs not using it
    - Optimal performance: Compiles to the best performing implementation as if handwritten with primitives

### Cause of memory unsafety

Data races are caused by concurrent access to same memory location and at least one access being a write.

can be caused by 
**aliasing** (different pointer symbols point to same memory location)
+ **mutation** (at least one of the pointers attempts to modify this memory location)

### Zero cost abstractions in Rust

1. Ownership/Borrowing
2. Iterators & Closures
3. Async/Await & Futures
4. `unsafe` and the module boundary

# Rust Features

## Move semantics

In Rust, **all values are (by default) passed-by-move**. Except for values that **implement the `Copy` trait**.

Moving and borrowing 

## Result

`Result<T,R>` is an enum with variants `Ok(T)` and `Err(E)`.

- If result *successfully*  returns, it's value is `Ok(val)` where val is the return value
- otherwise it's value is `Err(err_type)` where `err_type` is the error reported

```rust
use std::fs::File;

fn main() {
    let greeting_file_result = File::open("hello.txt");

    let greeting_file = match greeting_file_result {
        Ok(file) => file,
        Err(error) => panic!("Problem opening the file: {:?}", error),
    };
}
```

## Traits

Traits are like interfaces in other languages, provide an API for the type implementing it

```rust
pub struct MyClass;

pub trait Testable{
    fn test(&self) -> Result<(), String> {}
}

impl Testable for MyClass {
    fn test(&self) -> Result<(), String> {
        Ok(())
    }
}
```

You can get Rust to do default implementations of certain traits Rust has defined (e.g. `Copy, Debug`).
`#[derive(Debug)]` causes the Debug trait to be implemented for the decorated type.

If deriving `Copy`, its supertrait `Clone` must be derived too, i.e. `#[derive(Copy, Clone)]`.
- Copy: implicit (like copy assignment)
- Clone: explicit (call `x.clone()`)

Type cannot implement `Copy` by default if 
- contains `&mut T` (would create an aliased mutable ref)
- contains member object that doesn't implement `Copy`

## Closures

A closure is a function + the scope of variables it can access.

```rust
let my_closure = |x: param_type| -> return_type { func(x) };
let result : return_type = my_closure(x);
```

The closure can **include bindings** from the **enclosing scope**.

```rust
#[allow(unused_variables)]
fn main() {
    let mut num = 5;
    let plus_num = |x: i32| x + num; // can use

    // if the following part is added, then it won't compile
    /*
    let y = &mut num; // y is a mutable borrow
    *y+= 1;
    let m = plus_num(10); // num is an immutable borrow
    */
}
```

Forcing the closure to **take ownership** of an environment variable requires `move` keyword.

```rust
fn main() {
    let mut nums = vec!(1,2,3,4);
    let plus_nums = move |mut x: i32| {
        nums.push(5);
        for num in nums {
            x += num;
        }
        x
    };

    // the following cause error: borrow of moved variable `nums`
    // nums.push(5);
    // let y = num;
}
```

## Iterators

```rust
let mut nums = vec!(1,2,3,4);
let plus_nums = move |mut x: i32| {
    nums.push(5);
    for mut num in nums.into_iter() {
        num += 1; // these values are forgotten once `num` goes out of scope.
    }
    for num in nums.iter() {
        *num += 1; // the memory locations in nums updated
        x += *num;
    }
    x
};
```

## Pointer types

Zero cost abstractions:
- `Box<T>`: something like `std::unique_ptr`
  - When gone out of scope the destructor is run
- `&T` or `&mut T`: references that follow read-write-lock pattern
- `*const T` or `*mut T`: raw pointers with no ownership, only usable in `unsafe`

Abstractions with cost:
- `Rc<T>`: like `std::shared_ptr` without the atomic refernece count
  - has a reference count.
  - internal data is immutable
  - clone doesn't do a deep copy, copies the ref to the internal.
  - guarantee: destructor on internal run when rc = 0.
- `Arc<T>`: Uses an atomic reference count.
  - (thread-safe) guarantee: destructor on internal run when arc = 0.
- `Mutex<T>`: mutual-exclusion via RAII guards 
  - guards are objects which maintain some state.
  - mutex is opaque until we call `lock()` on it, which blocks till acquired and then a guard will be returned. 
  - guard can be used to access the inner data (mutably)
  - lock will be released when the guard goes out of scope.
  - **Guarantee**: safe shared mutability across threads

## Threads

`std::thread` has `spawn` which creates threads. `spawn` returns a `JoinHandle<T>` where `T` is the return type.

### Example of concurrent add

```rust
use std::sync::{Arc, Mutex};
use std::thread;

fn gen_thread(counter: Arc<Mutex<i32>>, id: i32) -> thread::JoinHandle<()> {
    thread::spawn(move || {
        let mut myc = counter.lock().unwrap();
        
        *myc += id;
        println!("t{} {}", id, *myc);
    })
}

#[allow(unused_variables,unused_assignments)]
fn main() {
    let counter = Arc::new(Mutex::new(0));
    
    let mut threads: Vec<thread::JoinHandle<()>> = Vec::new();
    for i in 1..101 {
        let c = counter.clone();
        threads.push(gen_thread(c, i));
    }
    for t in threads {
        t.join().unwrap(); // returns what's in the result
    }
}
```

## Scoped threads

Originally from Crossbeam library and now in `std`!

### Motivation

Question: How to **return** variables after moving them into threads? You can with `scope`s.

```rust
use rand::Rng;
use std::thread;

#[allow(unused_variables,unused_assignments)]
fn main() {
    let myvec = "my sentence isn't very long bro"
        .split_whitespace().collect::<Vec<&str>>();
    let mut sentinel: &str = "";
    // myvec, being immutably borrowed by all threads here, 
    // can have its mutable reference passed into all threads via a scope
    // which returns the moved myvec upon completion.
    let myscope = thread::scope(|scope| {
        let mut threads: Vec<thread::ScopedJoinHandle<()>> = Vec::new();
        print!("concurrent sentence: ");
        for i in 0..(myvec.len()) {
            let t = scope.spawn(|| {
                let mut rng = rand::thread_rng();
                let id: u8 = rng.gen();
                print!("{} ",myvec[id as usize % myvec.len()]); 
            });
            threads.push(t);
        }
        for t in threads {
            t.join().unwrap();
        }
        scope.spawn(|| {
            // mutable borrowing is fine too!
            sentinel = "!!!";
        }).join().unwrap();
    });
    // can return myvec, sentinel to outer scope
    print!("\nsentinel: {}", sentinel);
    print!("\nexpected sentence: ");
    for s in myvec {
        print!("{} ", s);
    }
}
```

## Channels (with an example using Sleeping Barber)

Lock-free version for fun. Uses atomic capacity to check 

```rust
use std::sync::mpsc;
use std::sync::{atomic::{AtomicUsize, Ordering::SeqCst}, Arc};
use std::thread;
use rand::Rng;


pub struct Customer {
    id: u32,
    haircut: String,
}

fn choose_hair() -> String {
    let mut rng = rand::thread_rng();
    let r: u8 = rng.gen();
    match r % 3 {
        0 => "BOB",
        1 => "MULLET",
        _ => "SHAVE",
    }.to_owned()
}

#[allow(unused_variables,unused_assignments)]
fn barbershop_channels() {
    let capacity = Arc::new(AtomicUsize::new(10));
    let (tx, rx) = mpsc::channel::<Customer>();
    let total_custs = 100;
    thread::scope(|scope| {
        let barber_cap = capacity.clone();
        let barber = scope.spawn(move || {
            for i in 0..total_custs {
                let cust = rx.recv().unwrap();
                println!("serving {}: {}", cust.id, cust.haircut);
                barber_cap.fetch_add(1, SeqCst);
            }
        });
        for i in 0..total_custs {
            let txcust = tx.clone();
            let cap = capacity.clone();
            let cust = scope.spawn(move || {
                let mut seats : usize = cap.load(SeqCst);
                if seats == 0 {
                    println!("left: {}", i);
                    return;
                }
                
                loop {
                    let update = seats-1;
                    match cap.compare_exchange_weak(seats, update, SeqCst, SeqCst) {
                        Ok(_) => break,
                        Err(x) => seats = x,
                    }
                }

                println!("waiting: {} [{} seats left]", i, seats-1);
                txcust.send(Customer {
                    id: i,
                    haircut: choose_hair(),
                }).unwrap();
            });
        }
    });
}

fn main() {
    barbershop_channels();
}
```

## Crossbeam

## Rayon
