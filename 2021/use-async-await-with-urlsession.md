# Use async/await with URLSession (WWDC 2021)

Swift works nicely with URLSession because it makes your code:

1. Linear
2. Concise
3. And supports native Swift error handling



Here's our app we'll be converting to use async/await.

<img src="assets/dog-photo-app.png" alt="Dog photo sharing app with favoritable images" style="zoom: 30%;" />



Here's our existing code that fetches a dog photo that uses the completion handler on URLSession. This code seems simple but has at least 3 mistakes.

First, notice the control flow jumps back and forth.

![image-20220522091133825](assets/urlsession-closure-control-flow.png)



Second, in terms of threading, we have three different execution contexts. The outter most context is run on the thread of the caller. URLSession's completion handler runs on sessions's delegate queue. And the final completion handler runs on the main queue.

Since the compiler can't help us here, we have to use extreme caution to avoid things like data races.

![image-20220522091433983](assets/urlsession-thread-contexts.png)



Lastly, the calls to the completion handlers have several issues. They're not consistently dispatched to the main queue, the first completionHandler is missing an early `return`, and the `UIImage()` initializer can fail and return `nil`.

![image-20220522091647654](assets/urlsession-completion-handler-errors.png)



Now, here's the new version using async/await. It's so much simpler. The control flow is linear from top to bottom. And we know that everything's function runs in the same concurrency context so we no longer have to worry about threading issues.

<img src="assets/convert-closure-to-async-await.gif" alt="Kapture 2022-05-22 at 09.01.36" style="zoom: 100%;" />



Note first, we use the new `URLSession.data()` method. It suspends the current execution without blocking and returns data + response or throws an error.

We also `throw` errors. This allows the caller to catch the errors using Swift native handling.

Lastly, the compiler essentially forces us to handle the `nil` case of `UIImage.init()`.

![Kapture 2022-05-22 at 09.25.37](assets/async-await-code-explanation.png)

### URLSession methods

`URLSession.data` methods fetch data from the network.

`URLSession.upload` methods upload data or the file. Be sure to set the http request method to "POST".

`URLSession.download` methods store the response body as a file rather than in-memory. These methods don't automatically delete the file, so don't forget to do so.

### Cancellation

Swift's concurrency cancellation work's with URLSession async methods. One way to cancel is to use a conurrency `Task` handle. 

<img src="assets/swift-task-cancellation.png" alt="image-20220522094234422" style="zoom:50%;" />



### Receiving a response incrementally — URLSession.bytes

URLSession.data, URLSession.upload, URLSession.download wait for the entire response body to arrive before returning.

URLSession.bytes return incrementally. They return once the response header is received and deliver the response body as an async sequence of bytes.

Here's an example...

Let's use the new async sequence API to update the favorites count as real time events are parsed.

1. We fetch the data using `URLSession.bytes()`. The bytes returned from `URLSession.bytes` are of type `URLSession.AsyncBytes`.
2. We also check for a successful HTTP response.
3. We parse each line of the response as a piece of JSON data. We do that by consuming the response line-by-line as data is received from `bytes.lines`.
4. Within the for-loop, the JSON is parsed and the UI is updated. Note the UI update needs to happen on the main actor and that's why `await` is used to call `updateFavoriteCount()` which is an async function. 



<img src="assets/urlsession-bytes-example-code.png" alt="image-20220522110749015" style="zoom:40%;" />

### URLSessionTask-specific delegate

Your session is designed around a delegate model which provides callbacks for events such as authentication challenges, metrics, and more.

All of the methods can also accept a task-specific delegate specific to each operation.

The delegate property is also introduced on NSURLSessionTask in Objective-C, to take advantage of the same capability.

![image-20220522131048608](assets/urlsessiontask-specific-delegate.png)



Let's go through an example of how we can add user authentication by using a task-specific delegate.

The AuthenticationDelegate conforms to URLSessionTaskDelegate and accepts an instance of SignInController. The SignInController class already contains some nice helper functions to prompt the user for credentials.

<img src="assets/authenticationdelegate.png" alt="image-20220522131911576" style="zoom:40%;" />

We instantiate an instance of AuthenticationDelegate and pass it as the delegate parameter to the `URLSession.data()` method. Note the delegate object isn't an instance variable and it's strongly held by the task until the task completes or fails. What's new is that the delegate can be used to handle events that are specific to an instance of a URLSessionTask.

<img src="assets/authenticationdelegate-instance.png" alt="image-20220522132008011" style="zoom:40%;" />



## Wrap up

We encourage you to adopt URLSession `async` methods and apply these concepts to your code.

- Convert functions taking a completion handler to `async` functions
- And convert repeated events handlers to `AsyncSequence`s