# Closures (Part 1)

## Closures prep: Syntax & Evolution

A standard function is actually just a named closure. A closure is a self-contained block of functionality that can be passed around like a variable. 

### Proving it: Function to Closure

**Step 1:** The Standard Function
```swift
func addNumbers(a: Int, b: Int) -> Int {
    return a + b
}
```

**Step 2:** The Closure Equivalent
To convert this to a closure, remove the func keyword and the function name. Move the parameters and return type inside the curly braces { }, and use the in keyword to separate the signature from the executable code.

```Swift
let addClosure: (Int, Int) -> Int = { (a: Int, b: Int) -> Int in
    return a + b
}

// Calling it looks identical to a function call:
let result = addClosure(5, 5)
```

## The Evolution of Closure Syntax
Swift is designed to make closures concise. Here is how a closure evolves from its clunkiest form to its cleanest shorthand.

**The Setup:** A function that accepts a closure.

```Swift
func performMath(on number: Int, using operation: (Int) -> Int) {
    let result = operation(number)
    print(result)
}
```

#### 1. The Full Syntax:

``` Swift
performMath(on: 5, using: { (value: Int) -> Int in
    return value * 2
})
```

#### 2. Trailing Closure Syntax:
Because the closure is the last parameter in the function, we can close the parentheses early and place the closure block outside.

```Swift
performMath(on: 5) { (value: Int) -> Int in
    return value * 2
}
```

#### 3. Type Inference:
Swift's compiler already knows the expected parameter types and return type from the function signature, so we can delete them safely.

``` Swift
performMath(on: 5) { value in
    return value * 2
}
```

#### 4. Shorthand Argument Names ($0):
Swift provides default names ($0, $1, $2, etc.) for closure arguments. This allows us to remove the parameter names and the in keyword entirely.

```swift 
performMath(on: 5) { $0 * 2 }
```
---

## 3. Practice Interview Questions

Test your knowledge on closure syntax before moving to advanced topics like `@escaping` closures and memory management.

1.  **The Cleanup Test:**
    > An interviewer hands you this code: 
    `array.map({ (number: Int) -> String in return String(number) })`
    *Using the evolution rules above, what is the absolute shortest way you can rewrite this using shorthand syntax?*

2.  **The Function Pass:**
    > We know functions are just named closures. If you have a predefined function called `func isEven(num: Int) -> Bool`, *how do you pass that exact function into a `filter` method instead of writing an inline closure?*

3.  **The Multiple Closure Rule:**
    > We talked about Trailing Closure syntax. But what happens if a function takes two closures? (For example, an animation with an `animations` block and a `completion` block). *How do you write that cleanly in modern Swift?*
