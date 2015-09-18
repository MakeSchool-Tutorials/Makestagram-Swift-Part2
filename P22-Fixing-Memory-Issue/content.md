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

#How Can We Fix the Problem?

We can discard the image data that is stored in the `image` property of a `Post`, when the `Post` is no longer displayed. **Where can we find out about posts that are no longer displayed?** Inside of the `PostTableViewCell`!

Whenever the `PostTableViewCell` receives a new post that should be displayed, we can check if the image of the old post needs to be retained or if we can free that memory up.

Let's add the code and then discuss it in detail.

> [action]
> Update the `didSet` observer of the `post` property in the `PostTableViewCell` class to look as following:
>
    var post:Post? {
      didSet {
>      
        postDisposable?.dispose()
        likeDisposable?.dispose()
        // free memory of image stored with post that is no longer displayed
        // 1
        if let oldValue = oldValue where oldValue != post {
            oldValue.image.value = nil
        }
>
          if let post = post {
            postDisposable = post.image.bindTo(postImageView.bnd_image)
>
            likeDisposable = post.likes.observe { (value: [PFUser]?) -> () in
              if let value = value {
                self.likesLabel.text = self.stringFromUserList(value)
                self.likeButton.selected = value.contains(PFUser.currentUser()!)
                self.likesIconImageView.hidden = (value.count == 0)
              } else {
                self.likesLabel.text = ""
                self.likeButton.selected = false
                self.likesIconImageView.hidden = true
            }
          }
        }
      }

1. The `oldValue` variable is available automatically in the `didSet` property observer. It provides us with a way to access the previous value of a property. We check if an `oldValue` exists and if that `oldValue` is different from the new `post`. If that's the case, we know that we need to do some cleanup.
2. By setting `oldValue.image.value` to `nil` we are secretly fixing an issue that we haven't even discussed yet. Without this code in place, we are adding a new binding whenever a new post gets assigned to our `PostTableViewCell`. In most cases, where you create a binding, you should also have code that destroys that binding when it is no longer needed. In the case of the `PostTableViewCell` we don't need the binding anymore if the cell is displaying a new post. By setting `oldValue.image.value` to `nil`, we unsubscribe from future updates of the old post.

Great! Whenever an image is no longer displayed, we are freeing up the memory. That means we'll only have about 3 images in memory at any point in time. Our memory issue is fixed!

#Won't This Make Our App Slower?

Yes and No. You might wonder if resetting the `image` to `nil` means that we need to download the image again, every single time a post is displayed, even if we have loaded the image previously.

Luckily that is not the case! Let's take a short look at the `downloadImage` method in the `Post` class:

    func downloadImage() {
         imageFile?.getDataInBackgroundWithBlock { (data, error) -> Void in
          if let data = data {
            let image = UIImage(data: data, scale:1.0)!
            self.image.value = image
          }
        }
      }
    }

You can see that we are calling `imageFile?.getDataInBackgroundWithBlock` whenever the `image.value == nil` condition is true. However, Parse automatically takes care of caching our downloads and storing them on disk. This means if we request an image that has been downloaded recently, it will be fetched from the iPhone's local storage instead of from the Parse server.

However, if you run the app in the current version you might be able to realize small freezes and delays when scrolling up and down the timeline. That's because even loading image files from disk can take a noticeable amount of time.

So now we have traded our memory problem for a performance problem; can we have the best of both worlds?

#Improving Performance

Yes we can! Thanks to `NSCacheSwift`. `NSCacheSwift` is very similar to a `Dictionary` it allows us to store key-value pairs. However, when our app receives a memory warning, entries are automatically removed from `NSCacheSwift`. That prevents our app from being shut down by the operating system.

So how can we use `NSCacheSwift` to solve our memory problem?
