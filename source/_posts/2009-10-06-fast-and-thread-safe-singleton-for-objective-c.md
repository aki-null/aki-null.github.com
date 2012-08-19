---
layout: post
title: Fast and Thread-safe Singleton for Objective-C
tags:
- Objective-C
- Grand Central Dispatch
---


I have seen various implementations of Singleton pattern in Objective-C in the past. The most common template I have seen is the following.

    static MyClass *instance = nil;

    + (MyClass *)sharedInstance {
        @synchronized(self) {
            if (instance == nil) {
                instance = [[self alloc] init];
            }
        }
        return instance;
    }

As you can see, it locks the method using self. The issue with this code is that it can get very slow if this method is called hundreds of times, because the code to instantiate the shared instance must be thread-safe. I was researching for a better way to make this faster, then I found [this article](http://eschatologist.net/blog/?p=178). The comment section of the article details the use of “Method Swizzling” to replace the method to access the shared instance with more optimised code (simply returning the instance without a check). Method swizzling allows us to modify the mapping from a selector to an implementation.

I have tried to use the code provided, but it failed to work. I have done some digging around, and I found an article on [CocoaDev](http://www.cocoadev.com/index.pl?MethodSwizzling) detailing the swizzling techniques. I have created a category of NSObject using one of the implementation, and modified my Singleton code to the following.

    static MyClass *instance = nil;

    + (MyClass *)sharedInstance {
        @synchronized(self) {
            if (!instance) {
                instance = [[MyClass alloc] init];
                OSMemoryBarrier();
                [self swizzleClassMethod:@selector(sharedInstance)
                     withClassMethod:@selector(sharedInstance2)];
            }
        }
        return instance;
    }

    + (MyClass *)sharedInstance2 {
        return instance;
    }

The above code worked very well. The performance of initialisation would be much slower than the original implementation, but it would be extremely fast after the first initialisation.

Edit: This should be faster than double-checked locking, but I have not tested the performance.

Edit2: I did need the memory barrier after all. The code has been updated.
