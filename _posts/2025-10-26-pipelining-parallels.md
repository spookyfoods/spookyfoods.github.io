---
layout: post
title: "Pipelining Parallels: My First Introduction to C++ Concurrency"
author: Siddham
date: 2025-10-26
changelog:
  - date: 2025-10-26
    description: "Added Section 1: 'The Goal'"
  - date: 2025-10-27
    description: "Added Section 2, 'The Workers (Stages vs. Threads)'"
last_updated: 2025-10-27
categories: [Computer Science, Programming]
tags: [C++, Concurrency, Pipelining, Computer-Architecture]
series_title: "Pipelining Parallels"
---
I've been learning C++ for a few months now, and I'm finally starting to get into the weeds. I have a habit of straying when I'm learning, and I recently got fascinated by concurrency while exploring what C++ is even used for. Every discussion spoke of it as something few can master. Naturally, I thought, 'Could it really be that tricky?'
With my Operating Systems course being in the next semester, I figured this might be one of the best times to get into it. Just a few days later, we started Pipelining in our COA class and i started drawing some parallels(no pun intended) to concurrency, so i figured, why not sit down and just get the basics for concurrenct C++ down, for now.

It turns out, the concept of **hardware pipelining** I've been studying is a near-perfect analogy for the fundamentals of concurrent programming. This blog series is my attempt to document my learning, starting with the first and most important parallel.

## Part 1: The Goal (It's All About Throughput)

Our first introduction to pipelining(and why it's pretty darn good), was through the famous sequential laundry load experiment. This example did help me get some inferences down immediately.

### The COA Concept: Throughput vs. Latency

If you have one load of laundry, it has three stages: Wash (30 min), Dry (40 min), and Fold (20 min).

* **Latency** is the total time it takes for that *one* load to go from dirty to folded, in isolation. That's $30 + 40 + 20 = 90$ minutes.
* **Throughput** is the *rate* at which you finish *all* your laundry (e.g., four loads).

If you do this sequentially, you'd wait the full 90 minutes for the first load to be *completely* done before starting the second. This takes 6 hours for four loads.

But a pipeline works differently. We can 'un-abstract' the entire process of a single load, into it's constituents of washing, drying, folding.
As soon as the first load is done with the washer, the *next* load can go in. The washer doesn't have to sit idle and "wait till the previous instruction is completely done."

By doing this, a new load of laundry is finished every 40 minutes (the time of the slowest stage, the dryer)

This helps us capture one of the key ideas, **"Pipelining doesn't help latency of single task, it helps throughput of entire workload"**. The latency for one load is *still* 90 minutes, but the total throughput is massively increased.

### The Concurrency Parallel: One Core vs. Many Cores

So, what does this have to do with C++? It turns out, the analogy is perfect.

A modern CPU has multiple "cores". When I write a simple program, it runs *sequentially*. It only uses **one** of those cores. This is the "sequential laundry" method. It's like having four sets of washers and dryers in my house but only ever using one.

**Concurrent programming** is the software equivalent of the "pipelined laundry" method. It's how I can write C++ code to use *all* (or at least, *more than one*) of my CPU cores at the same time.

The goal is identical:
* If I have 10,000 images to process, I can't do much about the **latency** (the time to process *one* image).
* But by using multiple cores, I can create a *software pipeline* and process many images at once, dramatically increasing my **throughput** (the total images processed per minute).

So, the first parallel is clear:

* **COA Pipelining Goal:** Use specialized hardware stages to increase **throughput**.
* **C++ Concurrency Goal:** Use multiple CPU cores as "stages" to... also increase **throughput**.

The goal is the same.

---
~~*Stay tuned for Part 2, where I'll explore the "workers" themselves—what what computer-architecture calls "Pipeline Stages" and what C++ calls `std::thread`🧵.*~~

---

## Part 2: The Workers (Stages vs. Threads)

We have established that both hardware pipelining and software concurrency share the same goal: **increasing throughput**.
Now, let's look at these curios 'workers' themselves.

### The COA Concept: Pipeline Stages

In computer architecture, the 'workers' are the **pipeline stages**. A pipeline stage is a singular piece of hardware with a very specific job, like the "Fetch" unit, the "Decode" unit, or the "Execute" (ALU) unit.

>"I fear not the man who has practiced 10,000 kicks once, but I fear the man who has practiced one kick 10,000 times." ~Bruce Lee

The magic of pipelining comes from this specialization. Each stage has a very specific job, so as soon as an instruction moves on, the now-free stage immediately pulls in the next one. This allows multiple instructions to be processed in parallel, each at a different stage of its life.

### The Concurrency Parallel: `std::thread`

So, how do we create a "specialist worker" in C++? The answer is the **`std::thread`**.

A CPU core is a *general-purpose* worker. A `std::thread` (from the C++ `<thread>` library) is our way of giving that general-purpose worker a *specialized role*. It's a "worker" that we can create in our code and assign a specific function to run.

The parallel is incredibly direct:
* **COA Pipeline Stage:** A *hardware* specialist, like the "Fetch unit," whose job is physically built into the circuit.
* **C++ `std::thread`:** A *software* specialist, which we create and assign a "job" (a function) to run, simulating a pipeline stage.

For reference, if I were to build a software analogue to a 4-stage CPU pipeline, I would write four separate C++ functions:
1.  `fetchFunction()`
2.  `decodeFunction()`
3.  `executeFunction()`
4.  `writeFunction()`

Then, in my `main()` program, I could launch four different `std::thread`s and assign each one a function. Just like that, I'd have four specialist workers ready to run in parallel, as would occur in a real pipelined system.

So, the second parallel is:
* **COA Pipeline Stage:** A specialized hardware unit.
* **C++ `std::thread`:** A general-purpose CPU core, which can be assigned a specialized role through our code.

---
*In Part 3, we'll get to the most critical part: how do these workers pass information to each other without causing a quagmire of confusion? We'll look at 'Interstage Buffers' and how they form the basis of all pipelines. And also... `r a c e  c o n d i t i o n s` 🎃
