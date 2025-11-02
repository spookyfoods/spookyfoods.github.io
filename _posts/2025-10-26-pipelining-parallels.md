---
layout: post
title: "Pipelining Parallels: My First Introduction to C++ Concurrency"
author: Siddham
date: 2025-10-26
# changelog:
#   - date: 2025-10-26
#     description: "Added Section 1, 'The Goal'"
#   - date: 2025-10-27
#     description: "Added Section 2, 'The Workers (Stages vs. Threads)'"
#   - date: 20252-10-29
#     description: "Added Section 3, 'Winning by... waiting? How buffers solve race conditions'"
last_updated: 2025-10-29
categories: [Computer-Science, Programming]
tags: [C++, Concurrency, Pipelining, Computer-Architecture]
series_title: "Pipelining Parallels"
next_post: "bonus-rethinking-queue"
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
~~*In Part 3, we'll get to the most critical part: how do these workers pass information to each other without causing a quagmire of confusion? We'll look at 'Interstage Buffers' and how they form the basis of all pipelines. And also... `r a c e  c o n d i t i o n s` 🎃*~~

---

## Part 3: Winning by... waiting? How buffers solve race conditions

Right. Through Part 1, we were able to settle on what our 'goal' actually is.
In Part 2, we hired our workers to help us chip away at it.
Now. We must discuss how do these workers communicate and/or cooperate? How does our `fetchFunction` worker pass on it's result to `decodeFunction`?
This was quite englightening.

## The COA Concept: The In-Tray and the Gate
<figure>
  <img src="{{ site.baseurl }}/assets/Pipelining-Parallels/fourStagePipeline.png" 
       alt="A block diagram of a four-stage instruction pipeline. The stages are labeled F for Fetch instruction, D for Decode instruction, E for Execute operation, and W for Write results. These stages are connected in sequence and separated by interstage buffers labeled B1, B2, and B3.">
  <figcaption>Figure 1 : Computer Organization [5e] by Hamacher et. al..</figcaption>
</figure>
Pipeline stages aren't connected end to end, they are separated by Interstage Buffers.
Initially, I thought, yep, makes sense. The buffer was an effective 'conveyer belt' of sorts, it decouples the stages, allowing, e.g: A 'Fetch' stage can simply dump-and-run its output into it's connected buffer(B1 in this instance). Correspondingly start working on the next instruction.
Honestly, I wasn't too happy with this explanation. I got stuck on a plot hole.
>*If the faster stage (let's say Stage 1) finishes its job in 30 minutes but has to wait for the 40-minute clock, why does it need to "dump" its result into a buffer? Why can't it just "hold" the result for those 10 minutes, and then pass it to Stage 2 at the clock tick?*
If it can wait, why the need for a separate "in-tray"? The answer to this should have been obvious to me, essentially:
- A Pipeline Stage is made of '*Combinational*' logic. The very instant it's inputs change, it's reflected in the output simultaneously, there is no 'waiting' in this realm.
- Whereas, the Interstage Buffer is made of '*Sequential* logic. It has a hold button, which is tied to the system clock.

This aided in mending a seam that was being a bother. The buffer isn't just a convenience; it's a necessity. Without it, as soon as Pipeline Stage would be done with it's job, it would immediately forward it to the next stage, without a bother, indifferent of the fact, if the next stage is already busy, or if the clock cycle has hit yet!
Let's go through a step-by-step to make sure we're on the same page, we will use a more simple 2 Stage Pipeline setup. Parameters: S1: Stage 1(takes 2ns), S2: Stage 2(takes 10ns), B1: Buffer, System Clock: 10ns;

- **AT T=0 ns (The Clock Ticks!):**
    
    - **B1**'s "gate" opens. It latches the result of I0​ (which S1 had prepared). B1's output _instantly_ becomes the stable I0​ result.
        
    - **S2** (slow) sees this new, stable I0​ result at its input and _immediately_ starts its 10 ns calculation.
        
    - **S1** (fast) is fed the new I1​ instruction and _immediately_ starts its 2 ns calculation.
        
- **AT T=2 ns:**
    
    - **S1**'s calculation is **finished**. Its output wires (which are connected to B1's _input_) now hold the stable, final result for I1​.
        
    - **B1** sees this new I1​ result, but its "gate" is **closed**. It _ignores_ this new value.
        
    - **Crucially, B1** continues to output the _old_ I0​ result to S2.
        
    - **S2** is only 20% done with its I0​ calculation. It continues working, completely _shielded_ by B1 from S1's new, irrelevant-for-now result.
        
- **BETWEEN T=2 ns and T=10 ns (The 8ns Wait):**
    
    - **S1** is completely **IDLE**. It has done its job. It's now just "shouting" the I1​ result at B1's closed "gate" for the next 8 ns.
        
    - **S2** is _actively calculating_ I0​ for this entire duration.
        
    - **B1**'s only job is to be a **"shield,"** holding the I0​ result rock-solid so S2 can work in peace.
        
- **AT T=10 ns (The Clock Ticks Again!):**
    
    - This is the simultaneous handoff!
        
    - A final register latches the I0​ result from S2's output. I0​ is _done_.
        
    - **B1**'s "gate" _instantly_ opens and latches the I1​ result that S1 has had waiting for 8 ns. B1's output now switches from I0​ to I1​.
        
    - **S2**'s input instantly sees the new I1​ result from B1. It _immediately_ starts its 10 ns calculation on I1​.
        
    - **S1** is _immediately_ fed the I2​ instruction and starts its 2 ns calculation.

## The Concurrency Parallel: The "Disaster" (Race Conditions) 🎃

Cool. so we have our hardware solution: a **1-item** in-tray (a buffer/register) combined with a synchronous gate (the system clock). This system is perfectly safe. By design.
Now, let's build a parallel model, in code. 
The first step is to build our in-tray. If we wanted a perfect 1-to-1 analogue of the hardware model, our obvious choice would be a single variable to act as our 1-item buffer.
**But here, we are going to make a crucial change.** We are going to intentionally stray from the exact hardware analogue.

Why? It's because although safe, the hardware model is quite rigid. The fast (**2ns**) 'Fetch' stage is forced to wait **8ns** for the slow (**10ns**) 'Execute' stage, all to stay in sync with the global clock. This is called tight coupling, and it's *in Larry David's voice* pretty-pretty inefficient.
In our simulation, we can do better. Why should our `fetch_worker` be forced to stop and sleep just because our `execute_worker` is slow. So instead of a 1-item buffer, we are going to use a different pattern using a **multi-item-buffer**. An `std::queue`.
By using a queue, we create a shock-absorber, of sorts.
1. Our fast fetch_worker (the info 'Producer') can work ahead consequently as soon as it generates an ouptut, filling the queue with 5, 10, or 100 items.
2. Our slow execute_worker (the 'Consumer') can then process those items at its own, slower pace.
Wow, so this is the famous **Producer-Consumer** pattern, I saw plastered all over when I was doing my initial superficial review of what concurrency might me.
It's far more flexible and efficient because it decouples our workers.

**And... Disaster. This powerful, asynchronous pattern we've just created is also the exact source of a new problem.**.
Our new conditions:
- **Multiple workers** (std::threads) running at different, unpredictable speeds.
- **No synchronous "gate"** (no global system clock) to keep them in check.
- **A shared "in-tray"** (the std::queue) that our workers will try and access at the same time.

Let's replay a situation which can help us highlight this **disaster**.
What happens if our fast **fetch_worker** (pushing) and our slow **decode_worker** (popping) try to access the std::queue at the exact same nanosecond?
A function like sampleQueue.push() isn't one single indivisible step. It's not atomic in nature.
It's many small steps in it's implementation (e.g., "check current size," "copy data to memory," "`size++`").
What if the OS scheduler performs a context switch on the `fetch_worker` right after it copies the data, but before it executes `size++`?
- `fetch_worker` (Thread 1) puts an instruction in the queue.
    
- `fetch_worker` gets paused. The queue's internal `size` variable still says `0`.
    
- `decode_worker` (Thread 2) runs. It checks the queue's `size`. It sees `0` and thinks the queue is empty (and maybe goes to sleep).
    
- The pipeline is now permanently broken. An instruction is "lost" in the queue forever, and our `decode_worker` may never wake up.

This disaster has a formal name: a **Race Condition**.
A race condition is a bug that occurs when the behavior of a program depends on the unpredictable timing of two or more threads "racing" to read and write a shared resource (our std::queue).

So, the third parallel is:

- **COA:** A _synchronous clock_(gate) + _1-item buffer_ **solves** the data handoff problem by design.
    
- **C++:** By _straying_ from the one-to-one analogue and using a _multi-item buffer_ (`std::queue`) with _asynchronous threads_, we create a more flexible system but also a massive new problem: **race conditions**.

We have to build the gate-equivalent from the hardware implementation, ourselves.

---
In Part 4, we'll do just that. We've identified the problem, so now we need a solution. We'll look at the C++ tool that acts as our very own software "gate" to protect our queue: the `std::mutex`.

---
