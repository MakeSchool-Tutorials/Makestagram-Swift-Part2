---
title: 'Fixing a Memory Issue'
slug: 'fixing-a-memory-issue'
---

Throughout this tutorial, we won't dive too deep into detecting and fixing memory issues. However, this step will discuss a specific issue of our **Makestagram** app - that should make you aware of potential problems in your own app.

#What are Memory Warnings?

If you spent some time using the current version of _Makestagram_ and upload a large amount of photos you will likely see a _memory warning_ show up in the Xcode console. Potentially your app will even crash. Despite the fact that today's iPhones are powerful devices with lots of memory and processing power, we still have limited resources to work with.

Because users of today's mobile devices expect apps to be extremely responsive, mobile operating systems must divide the available resources efficiently between the different apps running on a phone. **What does this mean for you?**
If your app is too greedy in its memory consumption, iOS will terminate it to prevent your app from causing the entire phone to slow down.

If you see a memory warning in the console or if your app even crashes due to a memory warning, it's time to act.

#What Are Typical Causes of Memory Warnings?

Most apps run into memory problems when objects are _retained_ too long. This means you have objects or resources in your program, that stay around even though you no longer need access to them. Further, most memory problems are caused by large resources such as videos or images.

#What's the Problem in Makestagram?

Since we won't discuss the debugging of memory issues as part of this tutorial, I'll provide the answer for you; however, first I'd like to give you a chance to guess which part of our code could be responsible for our memory issues.

> [solution]
> It is the code that is handling the images files! Currently, images are stored together with `Posts`. These images are stored as long as the post images stick around. If we load 100 posts for our timeline and a user scrolls through all posts, we have 100 images stored in memory, which takes up a lot of space! Ideally, we only want to keep the images that are currently displayed in memory.


