---
layout: post
title:  "Rust learning notes"
date:   2019-05-22 23:00:28 -0500
categories: jekyll update
---

I am a newbie of the Rust. If you are an advanced user, please turn off the page. 

I decide to learn the language by practising. To start with, I write a simple program to solve the two sum problem. The code is ugly especially for the first `result.push` line 

```
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

```
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
