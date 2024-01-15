# Meet Async/Await in Swift (WWDC 2021)

(Rough draft notes. Could use another pass.)

## --

The `await` keyword is how you notice that a block of code doesn't execute as one transaction. The function may suspend, and other things may happen while it’s suspended between the lines of the function. More than that, the function may resume onto an entirely different thread.

![image-20230207203337502](assets/await-and-suspension-points.png) 

## Async/await facts

- `async` enables a function to suspend. This suspends its callers too. So its callers must be async as well.
- `await` marks where an async function _may_ suspend execution.
- While an async function is suspended, the thread is not blocked. So the system is free to schedule other work.
  - Your app state can change a great deal while the function is suspended.
- Once an awaited async call completes, execution resumes after the `await`.

## Testing async code

We wanted testing async code to be just as easy as testing synchronous code, so XCTest supports async out of the box. What used to be a tedious process of setting up an expectation, calling the API under test, fulfilling the expectation, and then making sure to wait for an arbitrary amount of time becomes as easy as adding the async keyword to the test function, removing the XCTest expectations, its fulfillment, and the explicit await, and instead awaiting the results of calling the new asynchronous fetchThumbnail function that Nate showed you earlier.

```swift
class MockViewModelSpec: XCTestCase {
  func testFetchThumbnails() async throws {
    let result = try await self.mockViewModel.fetchThumbnail(for: mockID)
    XCTAssertEqual(result.size, CGSize(width: 40, height: 40)) 
  }
}
```

## Updating our app code to use the async function 

First we remove the completion handler, then `try` is added to handle any errors, and `await` to complete the call to an async function. 

Here, the onAppear modifier takes a plain, non-async closure, so there needs to be a way to bridge the gap between the synchronous and the asynchronous worlds: An async `Task` packages up the work in the closure and sends it to the system for immediate execution on the next available thread, like the async function on a global dispatch queue.

Its main benefit here is that async code can be called from inside of sync contexts.

![image-20230207204425436](assets/app-code-using-async-await.png)

## Async alternatives and continuations

- Continuations must be resumed exactly once on every path.
- Discarding the continuation without resuming is not allowed.

```swift
class ViewController: UIViewController {
  private var activeContinuation: CheckedContinuation<[Post], Error>?
  func sharedPostsFromPeer() async throws -> [Post] {
    try await withCheckedThrowingContinuation { continuation in
			self.activeContinuation = continuation
			self.peerManager.syncSharedPosts()
    }
  }
}

extension ViewController: PeerSyncDelegate {
  func peerManager(_ manager: PeerManager, received posts: [Post]) {
    self.activeContinuation?.resume(returning: posts)
    self.activeContinuation = nil // guard against multiple calls to resume
  }
  
    func peerManager(_ manager: PeerManager, error: Error) {
    self.activeContinuation?.resume(throwing: error)
    self.activeContinuation = nil // guard against multiple calls to resume
  }
}
```











































































