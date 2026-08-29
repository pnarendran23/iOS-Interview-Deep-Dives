 # Closures in Swift

Closures are one of the most powerful and heavily tested features in Swift.

## 1. Beginner: The Core Concept (Blocks of Code)

At the most basic level, a closure is just a function without a name that you can pass around like a variable.

Just like you can pass a `String` or an `Int` into a function, you can pass a block of code (a closure) into a function. The function can then execute that block of code later.

*   **Syntax:** `{ (parameters) -> return type in statements }`
*   **The `in` keyword:** This is the divider. Everything before `in` is the setup (inputs/outputs). Everything after `in` is the actual code to run.

### The Code Example

```swift
// 1. Assigning a closure to a variable
let greetUser = { (name: String) -> String in
    return "Hello, \(name)!"
}
print(greetUser("Alice")) // "Hello, Alice!"

// 2. Passing a closure to an Apple API (like sorting)
let names = ["Chris", "Alex", "Ewa", "Barry"]

// The long way:
let sortedNames = names.sorted(by: { (s1: String, s2: String) -> Bool in
    return s1 < s2
})

// The "Swifty" way (using trailing closure and shorthand arguments):
let quickSorted = names.sorted { $0 < $1 }
```

## 2. Intermediate: Escaping and Capturing Values

At the intermediate level, interviewers want to know how closures behave when dealing with state and asynchronous tasks.

*   **`@escaping` Closures:** The closure is saved to be executed *after* the function returns (e.g., a network request taking 2 seconds). Because the closure "escapes" the function, the compiler forces you to write `@escaping` so it knows it must hold onto memory (retain the closure) until it eventually runs.

```swift
// Example of @escaping (Asynchronous Network Call)
class NetworkManager {
    // We MUST use @escaping because the closure runs later, after the function finishes.
    func fetchData(completion: @escaping (String) -> Void) {
        DispatchQueue.global().asyncAfter(deadline: .now() + 2) {
            // 2 seconds later...
            completion("Data downloaded!")
        }
    }
}
```

*   **Capturing Values:** This is why they are called "closures." A closure "closes over" (captures) variables from the surrounding context. Even if the original scope dies, the closure keeps a reference to those variables so it can use them later.

``` swift
// 1. The Outer Function
func makeCounter() -> () -> Int {
    // This variable is local to makeCounter. 
    // Normally, it gets destroyed as soon as the function finishes.
    var runningTotal = 0 
    
    // 2. The Closure
    let incrementer: () -> Int = {
        runningTotal += 1 // The closure "captures" runningTotal
        return runningTotal
    }
    
    // 3. We return the closure itself, not the result!
    return incrementer 
}

// --- THE DEMONSTRATION ---

// We call the function. It creates the closure, returns it, and finishes.
// The `makeCounter` scope is now DEAD.
let myCounter = makeCounter() 

// But watch what happens when we call the closure:
print(myCounter()) // Prints: 1
print(myCounter()) // Prints: 2
print(myCounter()) // Prints: 3
```

If you look at this code, you might wonder why there is no @escaping keyword, even though the closure clearly outlives the function! That is because @escaping is only for input parameters. When a closure is the return type of a function, the compiler already knows it is leaving the scope, so it automatically handles the memory management for us without needing the keyword.

*   **Non-Escaping Closures (Default):** The closure is executed *before* the function returns. Think of `map`, `filter`, or `sort`. The compiler knows the closure's exact lifecycle, so it doesn't need to manage memory aggressively.

``` swift 
let numbers = [1, 2, 3, 4, 5]

// The closure inside `forEach` is non-escaping. 
// It runs completely for every item BEFORE moving on to the next line of code.
numbers.forEach { number in 
    print(number)
}

print("This line will only print AFTER the entire loop is done.")
```

* **Autoclosure:** Autoclosure is Apple's cheat code for performance. It lets you write clean, simple code without messy curly braces, but secretly delays the execution of that code until the exact moment you actually need it. It is literally how Apple built things like assert() and dictionary default values under the hood!

``` swift
func downloadBackupProfile() -> String {
    print("🌐 WARNING: Making an expensive network call to download backup!")
    return "Guest User"
}
```
Scenario 1: Without @autoclosure (The Mistake)
Let's write a function that takes a standard String as the fallback.

``` swift
func loadProfile(localName: String?, fallbackName: String) {
    if let name = localName {
        print("Welcome back, \(name)!")
    } else {
        print("Welcome, \(fallbackName)!")
    }
}

// Let's call it, providing a valid local name:
print("--- SCENARIO 1 ---")
loadProfile(localName: "Pradheep", fallbackName: downloadBackupProfile())

// The Output:
//--- SCENARIO 1 ---
//🌐 WARNING: Making an expensive network call to download backup!
//Welcome back, Pradheep!
```
**Why this is bad:** Even though we had "Pradheep" and didn't even need the fallback, Swift executed downloadBackupProfile() immediately just to figure out what string to pass into the function. We wasted data and processing power.

** Scenario 2: With @autoclosure **:

Now, let's add @autoclosure. We just change the parameter type. Notice that the call site at the bottom looks exactly the same.

``` Swift
func loadProfileAuto(localName: String?, fallbackName: @autoclosure () -> String) {
    if let name = localName {
        print("Welcome back, \(name)!")
    } else {
        // We only trigger the closure if we actually need it
        print("Welcome, \(fallbackName())!") 
    }
}

// Call it exactly the same way:
print("--- SCENARIO 2 ---")
loadProfileAuto(localName: "Pradheep", fallbackName: downloadBackupProfile())
The Output:

Plaintext
--- SCENARIO 2 ---
Welcome back, Pradheep!
```
**Why this is brilliant:** The warning didn't print! Because of @autoclosure, Swift secretly wrapped downloadBackupProfile() in a closure and paused it. When the function saw that localName was valid, it skipped the else block entirely. The network call was never made!


## 3. Expert: Memory Leaks and Capture Lists

This is the senior-level territory. **Closures are Reference Types (like classes).** When a closure captures a class instance (like `self`), it creates a strong reference to it.

If a `ViewController` owns a closure, and that closure captures `self`, you have created a **Retain Cycle** (a memory leak). Neither can be destroyed because they are pointing at each other.

*   **Capture Lists (`[weak self]`):** To break the retain cycle, you must use a capture list inside the closure. This tells the closure, "You can look at `self`, but don't force it to stay alive."
*   **`[weak self]` vs `[unowned self]`:**
    *   `[weak self]` makes `self` an **Optional**. If the `ViewController` is destroyed while the network call is still running, `self` becomes `nil`, and the app is safe. (Default to this).
    *   `[unowned self]` assumes `self` will never be `nil` when the closure runs. If the `ViewController` is destroyed and the closure fires, the app will crash. Only use it when you are 100% sure the closure and the object share the exact same lifecycle.
*   **`@autoclosure`:** A highly advanced keyword used in things like `assert()`. It automatically wraps an expression in a closure so that the code is delayed and only evaluated if needed, saving performance.

### The Code Example (The Classic Interview Problem)

```swift
class ProfileViewController {
    var userName = "Alice"
    let networkManager = NetworkManager()

    func loadProfile() {
        // [weak self] breaks the retain cycle!
        networkManager.fetchData { [weak self] data in
            // Because it's weak, self is now optional. We must unwrap it safely.
            guard let self = self else {
                print("ViewController was dismissed before data loaded.")
                return
            }
            
            // Now is it safe to update the UI? 
            // -------- TRAP ALERT!! -----------
            self.userName = data
            // -------- TRAP ALERT!! -----------

            print("Updated UI with: \(self.userName)")
        }
    }

    deinit {
        print("ProfileViewController deallocated - NO MEMORY LEAKS! 🎉 ")
    }
}
```

To fix the UI hanging or to avoid updating UI in the background thread,
``` swift
DispatchQueue.main.async {
    self.userName = data // THIS RUNS IN THE MAIN THREAD
}
```

## Summary Cheat Sheet for the Interview

*   **What is it?** A reusable block of code that captures variables from its surrounding scope.
*   **`@escaping`:** Used when a closure is called *after* the function it was passed into returns (async operations).
*   **Memory Leaks:** Because closures are reference types, they cause retain cycles if they capture `self` strongly.
*   **The Fix:** Use `[weak self]` to break the cycle and handle the optional safely with `guard let self = self`.

---

Alright, so that is the internal working mechanism of Closures and its different types.

But before we wrap up, Here are three realistic iOS interview questions about closures, ranging from intermediate to advanced. Try to answer one honestly in the comments

## The Community Interview Challenge

**Question 1: The Compiler Error**
> "You are writing a network manager function that downloads an image and returns it via a completion handler. You write the function signature, but the compiler immediately throws an error: *'Escaping closure captures non-escaping parameter'*. What is the compiler trying to tell you, and how do you fix it?"

**Question 2: Memory Management (Weak vs. Unowned)**
> "We just talked about using `[weak self]` inside closures to avoid retain cycles. But Swift also gives us `[unowned self]`. Can you explain the exact difference between `weak` and `unowned`? More importantly, can you think of a specific scenario where you might actually choose to use `unowned`?"

**Question 3: The Architecture Trick**
> "We know that capturing `self` strongly in an escaping closure creates a retain cycle. But let's say inside your closure, you don't need the whole view controller—you just need a single string property called `userID` to append to a log message. Is there a way to get that `userID` into the closure *without* capturing `self` at all?"
