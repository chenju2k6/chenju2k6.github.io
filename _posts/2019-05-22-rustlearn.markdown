---
layout: post
title:  "Rust learning notes"
date:   2019-05-22 23:00:28 -0500
categories: jekyll update
---


# Using Match for enums
I am a newbie of the Rust. If you are an advanced user, please turn off the page. 

I decide to learn the language by practising. To start with, I write a simple program to solve the two sum problem. The code is ugly especially for the first `result.push` line 

```rust
pub fn two_sum(nums: Vec<i32>, target: i32) -> Vec<i32> {
        let mut result = Vec::new();
        let mut mymap: HashMap<i32,i32> = HashMap::with_capacity(nums.len());

        for (i,&item) in nums.iter().enumerate() {
            if mymap.contains_key(&item) {
                result.push(*mymap.get(&item).unwrap());
                result.push(i as i32);
                return result;
            }
            mymap.insert(target-item,i as i32);
        }
        return result;
    }
```


Turns out a better way to do this is 

```rust
pub fn two_sum(nums: Vec<i32>, target: i32) -> Vec<i32> {
        let mut result = Vec::new();
        let mut mymap = std::collections::HashMap::with_capacity(nums.len());

        for (i,&item) in nums.iter().enumerate() {
            match mymap.get(&item) {
                None => (),
                Some(&x) => { result.push(x); result.push(i as i32); return result;}
            }
            mymap.insert(target-item,i as i32);
        }
        return result;

    }
```

This is a common pattern in programming rust. The type of `Option<T>` is pervasily used in the language. The use of `Option<T>` is considered as a measure to mitigate the `NULL` pointer issue. You can find more arguments found in here [The Definitive Reference To Why Maybe Is Better Than Null](http://www.nickknowlson.com/blog/2013/04/16/why-maybe-is-better-than-null/)


Later on, I am trying to print some value within a tuple. Like below

```rust
enum List {
    Cons(i32, Box<List>),
    Nil,
}

use crate::List::{Cons, Nil};

fn main() {
    let list = Cons(1,
        Box::new(Cons(2,
            Box::new(Cons(3,
                Box::new(Nil))))));
    println!("The first element of list is {}",list.0);
}
```

This code can't compile. The compiler complains that there is no such a filed in List enum type. Instead I have to do this

```rust
enum List {
    Cons(i32, Box<List>),
    Nil,
}

use crate::List::{Cons, Nil};

fn main() {
    let list = Cons(1,
        Box::new(Cons(2,
            Box::new(Cons(3,
                Box::new(Nil))))));
    if let Cons(i,j) = list {
        println!("value is {}",i);
    }
} 
```

# Lifetime

Lifetime is the most unique feature of the Rust. I am now struggling with it. In the Rust book, it mentions this example

```rust
fn longest(x: &str, y: &str) -> &str {
    if x.len() > y.len() {
        x
    } else {
        y
    }
}

```

This code can't compile, because the `borrow checker` cannot decide which reference is returned at compile time.


But the below code can compile. Looks like the `borrow checker` can analyze the code in a single function, but cannot work crossing the function "boundary". For example, in the below code, the `borrow checker` can annotate the lifetime automatically for `str1` and `str2`. However, for the code listed above, the `borrow check` cannot decide the lifetime of the references passed in within the `longest` function. In addition, the `borrow checker` working in the main function cannot decide which reference will be returned by the `longest` function.

The Rust book says that: 

> When annotating lifetimes in functions, the annotations go in the function signature, not in the function body. Rust can analyze the code within the function without any help. However, when a function has references to or from the code outside the function, it becomes almost impossible for Rust to figure out the lifetimes of the parameters or return values on its own. The lifetimes might be different each time the function is called. This is why we need to annotate the lifetimes manually.

```rust
fn main() {
    let string1 = String::from("abcd");
    let str1 = string1.as_str();
    let str2 = "xyz";

    let result;
    if str1.len() > str2.len() {
        result = &str1;
    }
    else {
        result = &str2;
    }
    println!("The longest string is {}", result);
}
```

## Lifetimes for struct

If a struct hold references, then it needs a lifetime annotation for all of its referenes. The `borrow chcker` then analyzes the code by using the hint indicated by lifetime annotations.

The below example is given by the Rust book

```rust
struct ImportantExcerpt<'a> {
    part: &'a str,
}

fn main() {
    let novel = String::from("Call me Ishmael. Some years ago...");
    let first_sentence = novel.split('.')
        .next()
        .expect("Could not find a '.'");
    let i = ImportantExcerpt { part: first_sentence };
}
```


## Why Rc<T>

The first dilemma of using lifetiems to annotate the references used in the structure comes from the below example

```rust
enum List {
    Cons(i32, Box<List>),
    Nil,
}

use crate::List::{Cons, Nil};

fn main() {
    let a = Cons(5,
        Box::new(Cons(10,
            Box::new(Nil))));
    let b = Cons(3, Box::new(a));
    let c = Cons(4, Box::new(a));
}
```

The code will not compile because in the initialization code of `b`, the value `a` is already moved into `b`. The `a` will not be there because it has been moved. If we change the definition of the `Cons` to let it hold references, then we need to specify the lifetimes for the members.


Note that in Rust, every value has only one owner. The value can be moved (by assigning) and borrowed (by reference), the there will not be two owners for one type


Note that in Rust, every value has only one owner. The value can be moved (by assigning) and borrowed (by reference), the there will not be two owners for one value. Using `RC<T>`, we can virtually created multiple owners of the value. Each time we call `RC::Clone`, we create a new owner for the value.


Using `RC<T>`, the above code can be optimized as below

```rust
enum List {
    Cons(i32, Rc<List>),
    Nil,
}

use crate::List::{Cons, Nil};
use std::rc::Rc;

fn main() {
    let a = Rc::new(Cons(5, Rc::new(Cons(10, Rc::new(Nil)))));
    let b = Cons(3, Rc::clone(&a));
    let c = Cons(4, Rc::clone(&a));
}
```
Noted that `Box<T>' enables the pointers to the data storing on the heap.
