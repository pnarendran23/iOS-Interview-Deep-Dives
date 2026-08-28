# Swift: Structs vs. Classes

## 1. Beginner: The Core Concept (Value vs. Reference)

At the most basic level, the difference comes down to how data is passed around in your app.
* **Structs are Value Types:** When you assign a struct to a new variable or pass it to a function, Swift creates a brand new copy.
  * *Analogy:* Handing someone a physical photocopy of a document. If they draw on their copy, your original is untouched.
* **Classes are Reference Types:** When you assign a class to a new variable, you are passing a reference (pointer) to the same instance in memory.
  * *Analogy:* Sharing a link to a Google Doc. If someone edits the doc, everyone sees the changes.

### The Code Example

```swift
// STRUCT EXAMPLE (Value Type)
struct Sandwich {
    var filling: String
}
var mySandwich = Sandwich(filling: "Turkey")
var yourSandwich = mySandwich // A full copy is made here

yourSandwich.filling = "Ham"
print(mySandwich.filling)   // "Turkey" - The original is unchanged
print(yourSandwich.filling) // "Ham"

// CLASS EXAMPLE (Reference Type)
class Pizza {
    var topping: String
    init(topping: String) { self.topping = topping }
}
var myPizza = Pizza(topping: "Pepperoni")
var yourPizza = myPizza // Both point to the exact same pizza in memory

yourPizza.topping = "Mushrooms"
print(myPizza.topping)   // "Mushrooms" - The original was changed!
print(yourPizza.topping) // "Mushrooms"
```
# 2. Intermediate: Mechanics and Rules

Once you understand how they are passed, you need to know how they behave differently in everyday iOS development.

Memory Location (Stack vs. Heap):

Structs are typically allocated on the Stack. The Stack is extremely fast, highly organized, and automatically cleans itself up when a function finishes.

Classes are allocated on the Heap. The Heap is slower, more complex, and requires memory management to know when to delete objects.

### Initialization:

Structs give you a "memberwise initializer" for free. You don't have to write an init() method to set its properties.

Classes require you to explicitly write an init() method if your properties don't have default values.

### Inheritance:

Classes support inheritance (e.g., class Dog: Animal). A class can inherit properties and methods from a superclass.

Structs cannot inherit from other structs. (However, both structs and classes can conform to Protocols).

### Mutability (The mutating keyword):

By default, methods inside a struct cannot modify the struct's own properties. If you want a struct method to change a property, you must mark it with the mutating keyword.

Methods in a class can modify their properties freely without any special keywords.

``` Swift
struct Counter {
    var count = 0
    
    // Must explicitly state that this method changes the struct
    mutating func increment() {
        count += 1
    }
}
```

3. Expert: Under the Hood (Performance & Concurrency)
At the senior level, interviewers want to hear about performance implications, method dispatch, and thread safety.

### Method Dispatch (Static vs. Dynamic):

Structs use Static Dispatch (Direct Dispatch): Because structs don't support inheritance, the compiler knows exactly which method will be called at compile time. It hardcodes the memory address of the method. This is incredibly fast and allows the compiler to heavily optimize (like inlining code).

### Classes use Dynamic Dispatch (Table Dispatch):
 Because a class might be subclassed (and methods overridden), the compiler has to look up a "Virtual Table" (v-table) at runtime to figure out exactly which version of a method to execute. This look-up takes extra time.

### Automatic Reference Counting (ARC) Overhead:

Classes use ARC. Every time you pass a class around, Swift increments a reference count. When it hits zero, it deallocates the memory. This constant counting is thread-safe, but it adds overhead.

Structs (usually) do not use ARC. Trick question warning: If a struct contains reference types (like a struct holding an array of classes), passing that struct does incur ARC overhead for the classes inside it.

### Thread Safety / Concurrency:

Structs are inherently safer in multi-threaded environments. Because passing a struct creates a copy, you don't have to worry about Thread A modifying the data at the exact moment Thread B is trying to read it (a race condition).

Classes are dangerous across threads. Multiple threads can access the same instance simultaneously. You must use mechanisms like Actors, serial dispatch queues, or NSLocks to make them thread-safe.

### Objective-C Interoperability:

If you need to expose your Swift code to Objective-C, you must use a class, and it must inherit from NSObject (with the @objc attribute). Structs cannot be represented in Objective-C.

## Summary Cheat Sheet
| Feature | Struct | Class |
| :--- | :--- | :--- |
| **Type** | Value | Reference |
| **Memory Allocation** | Stack (Fast) | Heap (Slower) |
| **Method Dispatch** | Static (Compile-time) | Dynamic (Run-time v-table) |
| **Inheritance** | No | Yes |
| **Thread Safety** | Safe (Copies state) | Unsafe (Shares state, needs locking/Actors) |
| **ARC (Ref Counting)** | No (unless it contains classes) | Yes |
| **Free Initializer** | Yes (Memberwise) | No |
| **Obj-C Interop** | No | Yes |


## When to use a struct (The Default)
You should use a struct when your primary goal is passing around data, not managing a shared process.

**For Data Models:** If you are decoding a JSON response from an API (like a User, Product, or Message), use a struct.

**When you want independent copies:** If you want to guarantee that passing data to another function won't accidentally alter your original data, use a struct. This makes your code highly predictable and bug-resistant.

**For Thread Safety:** In concurrent programming, structs are inherently safer. Because each thread gets its own unique copy of the struct, you don't have to worry about two threads trying to overwrite the exact same memory at the exact same time (race conditions).

**When you don't need inheritance:** If your model doesn't need to inherit traits from a superclass, stick to a struct. (Remember, you can still use Protocols for shared behavior).

## When to use a class (The Exceptions)
You must use a class when you need shared state, identity, or framework compatibility.

**When you need Shared State:** If you have a DatabaseManager, an AudioPlayer, or a NetworkService where multiple parts of your app need to talk to the exact same instance, you must use a class.

**When you need Identity:** With a class, you can use the identity operator (===) to check if two variables point to the exact same object in memory. Structs don't have identity; they only have equality (==).

**When using UIKit / Objective-C:** If you are interacting with older Apple frameworks, building a UIViewController, or need to expose a method to Objective-C using @objc, you are forced to use a class.

**When you need Inheritance:** If you need to build a hierarchy (e.g., class Vehicle, class Car: Vehicle), you must use a class.

**To avoid ARC Overhead (The Pro Tip): As we discussed earlier, if your data model contains several heavy reference types (like a struct holding multiple arrays of classes), it is often better for performance to make it a class to avoid massive ARC overhead when it gets copied.**

> **Apple's Official Guideline:** Default to using `struct`. Only switch to a `class` when you specifically need a reference type (shared state), Objective-C interoperability, or inheritance.

Alright, so that is the internal working mechanism of Structs versus Classes. Remember: default to structs for your data, and use classes when you actually need shared state or inheritance.

But before we wrap up, I am going to leave you with two tricky questions that senior interviewers love to ask.

### Question 1: (Beginner friendly)
"You are building a simple mobile game. You need to create PlayerStats (which holds the player's current level, score, and health) and a GameEngine (which controls the main game loop, pauses the game, and plays background music). Would you use a class or a struct for each of these? Walk me through your reasoning."

### Question 2: The SwiftUI Context

"Apple heavily pushes us to use structs, and in SwiftUI, all of our Views are structs. However, when we want to share reactive data across multiple screens using @StateObject or @ObservedObject (or @Observable in modern Swift), we are required to use a class. Why do you think Apple designed it so that Views are structs, but our view models/shared state must be classes?"

### Question 3: The Architecture Choice

"You are building a new chat application. You need to create a Message model (which holds the text, sender ID, and timestamp) and a WebSocketManager (which handles the active connection to the server and broadcasts new messages). Would you use a class or a struct for each of these? Walk me through your reasoning."
