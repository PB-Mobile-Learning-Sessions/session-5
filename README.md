```swift
//
//  README.md
//  Session 3 iOS frameworks 
//
//  Created by Farid Valiyev on 16.09.22.
//
```

# Session 5 Async programming 

## _Table of Contents_
- Async programming
    - async/await (__Foundation__)
    - Promises (__Promises__)
    - RxSwift (__RxSwift__)
    - Home works
    - Source codes
    
## Async programming
What is Async programming?
> On iOS and MacOS, the main thread renders UI and handles events. A lot of the tasks that our apps perform, including data deserialization and image processing, take a while to finish. When we run them on the main thread, our app stops responding to user interaction and become unresponsive. To ensure a good user experience, we need to keep expensive tasks off the main thread. The best way to achieve this is to write asynchronous code. 

__What is async?__
> Async stands for asynchronous and can be seen as a method attribute making it clear that a method performs asynchronous work.
```swift
func fetchImages() async throws -> [UIImage] {
    // .. perform data request
}
```

__What is await?__
> Await is the keyword to be used for calling async methods. You can see them as best friends in Swift as one will never go without the other.  
```swift
do {
    let images = try await fetchImages()
    print("Fetched \(images.count) images.")
} catch {
    print("Fetching images failed with error \(error)")
}
```
__What is DispatchQueue?__
> An object that manages the execution of tasks serially or concurrently on your app's main thread or on a background thread.
Dispatch queues are FIFO queues to which your application can submit tasks in the form of block objects. Dispatch queues execute tasks either serially or concurrently. Work submitted to dispatch queues executes on a pool of threads managed by the system. Except for the dispatch queue representing your app's main thread, the system makes no guarantees about which thread it uses to execute a task.
```swift
class ViewController: UIViewController {

//    let queue = DispatchQueue(label: "az.pashabank.lesson3")
    let queue = DispatchQueue(label: "az.pashabank.lesson3", attributes: .concurrent)

    override func viewDidLoad() {
        super.viewDidLoad()
//        queue.sync {
        queue.async {
            print("Task 1 Started")
            print("Task 1 Ended")
        }
//      queue.sync {
        queue.async {
            print("Task 2 Started")
            print("Task 2 Ended")
        }
    }
}
```
__What is OperationQueue?__
> Operation is a wrapper around some work that we would like to execute. Speaking more formally it is an abstract class that when subclassed will perform a given task for us. It’s built atop of the Grand Central Dispatch, which allows us to focus less on details of how concurrent execution will be done and more on the implementation of our business logic. Operations can help us with tasks that require more time to complete such as network calls or data processing.
```swift
        let mainQueue = OperationQueue.main
        let customQueue = OperationQueue()
        customQueue.maxConcurrentOperationCount = 30
        let getUserOperation = Operation()
        let getUserImageOperation = Operation()
        getUserImageOperation.addDependency(getUserOperation)
        customQueue.addOperation(getUserImageOperation)
```
__What are Actors?__
> Since Swift 5.5 and its built-in concurrency system, a new type declaration keyword has been added to the mix — actor. Swift in actor is same as class or struct, we can define an actor with a keyword as “actor”.
```swift
class User {
    let id: String
    let name: String

    init(id: String, name: String) {
        self.id = id
        self.name = name
    }
}
// Without Actor
class UserData {
    private var data = [String: User]()
    private let queue = DispatchQueue(label: "queue_user_save")

    func get(id: String) -> User? {
        queue.async {
           return data[id]
        }
    }

    func save(user: User) {
        queue.async {
            self.data[user.id] = user
        }
    }
    init() {}
}
// With Actor
actor UserData {
    private var data = [String: User]()

    func get(id: String) -> User? {
        return data[id]
    }

    func save(user: User) {
            self.data[user.id] = user
    }

    init() {}
}

class ViewController: UIViewController {
    override func viewDidLoad() {
        super.viewDidLoad()
        Task {
            let userData = UserData()
            await userData.save(user: User(id: "123", name: "Farid"))

            let getUser = await userData.get(id: "123")

            print(getUser?.name)
        }
    }
}
```

## Promises
What is Promises?

> In general, a promise represents the eventual result of an asynchronous task, respectively the error reason when the task fails. Similar concepts are also called futures
A promise can be in one of three states:  
    - pending - the promise is unresolved and the result is not yet available
    - fulfilled - the promise is resolved with some value
    - rejected - the promise is resolved with some error
Once fulfilled or rejected, a promise can never change its state in the future. Also, it can have an infinite number of observers waiting for it to be resolved. Once resolved, either a value or an error is broadcasted to all observers. Each observer, returns a new promise on subscribe, which, in turn, will be resolved with another value or error the observer provides. This enables chaining promises together to create a pipeline into a pipeline of transforming values which are computed asynchronously on different threads.

How to configure promises?

__Swift Package Manager__

In your Package.swift file, add Promises dependency to corresponding targets:
```swift
let package = Package(
  // ...
  dependencies: [
    .package(url: "https://github.com/google/promises.git", from: "2.1.0"),
  ],
  // ...
)
```

__CocoaPods__

To use Promises for both Swift and Objective-C, add the following to your Podfile:

`pod 'PromisesSwift'`

Don't forget to add use_frameworks! to your target. 

For Swift, import the module:

```swift 
import Promises 
```
Create Promise
```swift 
        let promise = Promise<String> { fulfill, reject in
          if success {
            fulfill("Hello world.")
          } else {
            reject(someError)
          }
        }
```

Create Promise with pending
```swift 
func getUsers() -> Promise<[User]> {

    let promise = Promise<[User]>.pending()

    promise.fulfill([User(id: "123", name: "Farid")])

//  promise.reject(error) 

    return promise
}
```

Use Promise
```swift 
        let promise = Promise<String>.pending()

        self.getUsers()

        .then { (users) -> String in
            return users.first!.name
        }
        .then { stringUserImageUrl in
            promise.fulfill(stringUserName)
        }
        .catch { error in
            promise.reject(error)
        }
```


















