---
title: 'Fixing a Memory Issue'
slug: 'fixing-a-memory-issue'
---



Whenever we download an image, we store it in the cache. We use the filename of the `PFFile` as a key for the cache. Next time we need to download a `PFFile`, we check if that file has already been downloaded and if the file content is stored in our cache. If that's the case, we get the file content from the cache instead of loading it from disk.

What we've just described is a very typical caching mechanism. Let's implement it in our `Post` class!

We start by importing the `ConvenienceKit` framework that provides the `NSCacheSwift` class.

> [action]
> Add the following `import` statement to the `Post` class:
>
    import ConvenienceKit

Next, we'll define a `static` variable that will store our cache. We define it as `static` because the cache does not belong to a particular instance of the `Post` class, but is instead shared between all posts.

> [action]
> Add the following variable definition to the `Post` class:
>
    static var imageCache: NSCacheSwift<String, UIImage>!

As you can see, we need to provide two pieces of type information: the first is the type of the keys we want to store in the cache, the second is the type of the values. We want to store image files and access them through their filename so we choose `<String, UIImage>`.

As soon as the `Post` class is loaded, we want to create an empty `imageCache`. The `initialize` method is the right place for class-level initialization code.

> [action]
> Extend the `initialize` method to look as following:
>
    override class func initialize() {
      var onceToken : dispatch_once_t = 0;
      dispatch_once(&onceToken) {
        // inform Parse about this subclass
        self.registerSubclass()
        // 1
        Post.imageCache = NSCacheSwift<String, UIImage>()
      }
    }

1. We create an empty cache. Remember that the other lines in this method are primarily Parse boilerplate code.

Now we can extend the `downloadImage` method to use our cache.

> [action]
> Extend the `downloadImage` method to look as following:
>
    func downloadImage() {
      // 1
      image.value = Post.imageCache[self.imageFile!.name]
>
      // if image is not downloaded yet, get it
      if (image.value == nil) {
>
        imageFile?.getDataInBackgroundWithBlock { (data: NSData?, error: NSError?) -> Void in
          if let data = data {
            let image = UIImage(data: data, scale:1.0)!
            self.image.value = image
            // 2
            Post.imageCache[self.imageFile!.name] = image
          }
        }
      }
    }

1. We attempt to assign a value to `image.value` directly from the cache, using `self.imageFile.name` as key. If this assignment is successful the entire download block will be skipped.
2. If we didn't have the image cached, we proceed as usual. Then, when the image is downloaded, we add it to the cache.

Now we have a solution in place that is fast and responds correctly to low-memory situations!

#Conclusion

In this step, you learned a bunch of things that are important to build a stable app!

First, you learned that we should break up bindings when we know that they are no longer needed. We can do that by setting the binding to `nil`. If you end up using `Observable` in your own project, make sure to check for situations where you should break existing bindings.

You've also learned how to use `NSCacheSwift` to cache content that is expensive to fetch and generate. Even loading image files from disk can cause small freezes in your app - and your users will notice. `NSCacheSwift` is easy to use and will free memory automatically when necessary.

In the next chapter, we'll discuss another form of making your app more stable - implementing basic error handling.
