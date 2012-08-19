---
layout: post
title: Grand Central Dispatch (Part 2)
tags:
- Objective-C
- Grand Central Dispatch
---


The Grand Central Dispatch (GCD) can be used to optimise programs for multi-core processors. However, the usual issue with threading still exists in GCD (GCD is not a magic). I would like to cover how to use semaphore to make a program thread-safe.

The most common issues you may face when you introduce threading into your program are accesses to shared resources. An example of the issue is presented in the code below.

    int main (int argc, const char * argv[]) {
       NSAutoreleasePool * pool = [[NSAutoreleasePool alloc] init];

       NSMutableSet *items = [NSMutableArray array];

       dispatch_queue_t queue =
          dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);
       dispatch_apply(50, queue,
             ^(size_t index) {
                for (int i = 0; i < 500; i++) {
                   [items addObject:@"hi"];
                }
             });

       NSLog(@"%i", [items count]);

       [pool drain];
       return 0;
    }

The program above simply tries to add an item to the shared resource (NSMutableSet) from multiple threads. If you try to run this program, it will probably crash. It is because NSMutableSet is NOT thread-safe (actually, all collection classes are not thread-safe). To prevent this issue, we could use the traditional *@synchronized* block to ensure thread-safety (example below).

    int main (int argc, const char * argv[]) {
       NSAutoreleasePool * pool = [[NSAutoreleasePool alloc] init];

       NSMutableSet *items = [NSMutableArray array];

       dispatch_queue_t queue =
          dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);
       dispatch_apply(50, queue,
             ^(size_t index) {
                for (int i = 0; i < 500; i++) {
                   @synchronized (items) {
                      [items addObject:@"hi"];
                   }
                }
             });

       NSLog(@"%i", [items count]);

       [pool drain];
       return 0;
    }

Well, it runs... slowly. You must remember that locks are very expensive. In the case of the program above, it will lock 500 times per block, which is extremely inefficient. Apple has provided optimised version of semaphore for GCD (dispatch semaphore). The traditional semaphores always require calling down to the kernel to test the semaphore, but the dispatch semaphore tests semaphore in user space, and only traps into the kernel only when the test fails and needs to block the thread. The following code demonstrates the usage of the dispatch semaphore.

    int main (int argc, const char * argv[]) {
       NSAutoreleasePool * pool = [[NSAutoreleasePool alloc] init];

       NSMutableSet *items = [NSMutableArray array];

       dispatch_semaphore_t itemLock = dispatch_semaphore_create(1);
       dispatch_queue_t queue =
          dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);
       dispatch_apply(50, queue,
             ^(size_t index) {
                for (int i = 0; i < 500; i++) {
                   dispatch_semaphore_wait(itemLock, DISPATCH_TIME_FOREVER);
                   [items addObject:@"hi"];
                   dispatch_semaphore_signal(itemLock);
                }
             });
       dispatch_release(itemLock);

       NSLog(@"%i", [items count]);

       [pool drain];
       return 0;
    }

“*dispatch_semaphore_create*” function is used to create a dispatch semaphore. The parameter specifies the starting value for the semaphore. To wait for the resource to be available, you have to use “*dispatch_semaphore_wait*”. Use “*dispatch_semaphore_signal*” function to signal the semaphore. Finally, “*dispatch_release*” function is called to release the memory allocated for the semaphore. The semaphore is not managed by the reference counter, so you must release it manually.

This is the basic usage of dispatch semaphore. It also has many other usages, such as managing program flow of threaded application.
