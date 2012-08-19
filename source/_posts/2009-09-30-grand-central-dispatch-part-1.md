---
layout: post
title: Grand Central Dispatch (Part 1)
tags:
- Objective-C
- Grand Central Dispatch
---


I’ve been working on optimising my Twitter client using Grand Central Dispatch (GCD) recently. [Grand Central Dispatch][1] is an Apple technology to optimise application with multicore processor. It was released with Mac OS 10.6 (Snow Leopard).

GCD makes it easier for programmers to perform tasks on different threads to optimise its algorithm performance. There are other interesting usages, which I will probably cover later. The GCD uses the new language feature of Objective-C 2.1 (also available on C and C++), called Blocks. Blocks lets us create closure-like objects to make it easy to execute a block of code parallel to the main thread. Blocks can also be used in C or C++. I’m not going to cover how to write blocks in this post, so it may be good to have a [read][2] about it if you don’t already know how it works.

The first example I’m going to cover is simple batch processing. It is quite common to batch process data in a loop, but it is usually done on the main thread, which is not desirable for multicore processors.

The following code is a small loop that can be seen in many applications.

    // iterate through all tab items and execute an expensive operation
    for (NSTabViewItem *currentItem in [tabView tabViewItems]) {
        [[currentItem identifier] doExpensiveOperation];
    }

As you can see, this code will only utilise one core, which is not efficient for multicore processors. This is where we can introduce one of the GCD feature. The following code basically replaces the loop with GCD batch processing function.

    // get all tab items
    NSArray *tabItems = [tabView tabViewItems];
    dispatch_apply([tabItems count],
            dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0),
            ^(size_t index) {
                NSTabViewItem *currentItem;
                currentItem = [tabItems objectAtIndex:index];
                [[currentItem identifier] doExpensiveOperation];
            });

The *“dispatch_apply” *function is used to batch process a specified block. The first argument specifies the number of loops to execute. In the example’s case, it is the number of tab items.

The second parameter specified the queue to use. Queue in GCD is an object that maintains set of blocks to execute. The actual execution of blocks are done automatically by GCD, so programmer does not need to worry about the execution after scheduling. GCD automatically detects the optimal number of threads to start, so the code does not need to be optimised for different systems.

The third parameter specifies the actual block to execute. In the code above, I’m getting the tab item I need to process from *“tabItems”* array. The *“index”* parameter provides the current thread index, starting from 0.

The *“dispatch_apply”* blocks, so it will wait until the batch process is complete. Batch processing feature of GCD will automatically optimise the code to run optimally on multicore machine. However, you have to be always be careful when using threads. The block you passed in as a parameter can obviously run on different threads at the same time, which can cause a big issue depending on your code. If *“doExpensiveOperation”* accesses a shared resource, it may cause a serious threading issue. I’m going to cover how semaphore works in GCD to solve this typical problem of threading in the later post.

[1]: http://www.apple.com/macosx/technology/
[2]: http://en.wikipedia.org/wiki/Blocks_(C_language_extension)