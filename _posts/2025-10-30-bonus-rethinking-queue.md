---
layout: post
title: "Rethinking the Humble Queue: A more learned perspective"
author: Siddham
date: 2025-10-30
last_updated: 2025-10-30
categories: [Computer-Science, Programming]
tags: [C++, Concurrency, Pipelining, Computer-Architecture]
series_title: "Pipelining Parallels"
prev_post: "pipelining-parallels"
next_post: "part-2-pipelining-paralles"
---
## Beyond the ADT
Treating data structures as black-boxes has some highly rewarding use cases, as it allows developers to focus on the higher-level interfacing with the data structure, instead of it's low-level implementatoin and other nooks and crannies.
A queue is a great example for this, it's a box with 2 main operations:
- `enqueue(X)`: Adds an item to the back
- `dequeue(X)`: Removes an item from the front

It follows a simple First-In, First-Out principle. I have implemented and used it in code quite a few times now, and really thought I got what it was about.

## The Perfect Use Case... not quite.
When looking for the most apt candidate for our in-tray [from]({% post_url _posts/2025-10-26-pipelining-parallels %}) implementation, the queue seemed to be obvious as it was. The Producer(our Fetch Stage) would simply `enqueue()` our instructions into the queue while The Consumer(our Decode stage) simply `dequeue()`s them one by one, as per it's convenience.

But of course, abstracting things away can have their defects. Unfortunately this was one of them.

My DSA class assumed a single-threaded world, where only one single thing happens at a time. My std`::queue` was designed for that world.
*What happens when two threads try to use the "black box" at the exact same time?*
### Enter Race Conditions
What happens when two workers try to use our queue at once? Our box breaks.
An operation like `enqueue()` or `dequeue()` isn't a singular magical step, it's not **atomic**.
It has many internal steps, a thread can be paused anywhere between those steps, by an OS context-switch.
My example fron the [last post]({% post_url _posts/2025-10-26-pipelining-parallels %}) creates a good case with size++; essentially, a simple size++ operation can result in violating our ADT invariants:
- Thread 1 `push()`es, copies data, but gets paused _before_ `size++`.
    
- Thread 2 `pop()`s, checks `size` (which is still the old value), and thinks the queue is empty.
    
- Disaster.

## The Solution
So we can conclude that our candidate cannot be a queue, so what should it be?
We will be designing our own from scratch; **The `ThreadSafeQueue`**.
It will still have the same interface, our trusy `enqueue()` and `dequeue()` operations, but now they must be made atomic, somehow.

When my fetch_worker calls `enqueue()`, it must be able to 'lock' the queue do all its work (copy data, update size), and then 'unlock' the queue," all as one indivisible unit. No other thread shall be permitted to interact with the queue while it's locked. This is the principle of **mutual exclusion**.

---

This was a nice little thought-experiment, it does provide me with a fresh perspective about data structures, earlier I used to look at them as a collection of 'solved' and proven tools, but these tools have a certain set of agreed assumptions. When we play dangerously and violate these assumptions, we must go back to first principles and rethink them.
