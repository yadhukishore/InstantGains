**Stack Underflow** is a condition that occurs in computing when an attempt is made to remove an item from an empty stack. Stacks are data structures that follow the Last In, First Out (LIFO) principle, where the most recently added element is the first one to be removed. When the stack is empty, and a pop operation (removal) is attempted, underflow occurs.

### Key Points:
- **Stack**: A data structure that operates in a LIFO manner.
- **Push**: Adding an element to the top of the stack.
- **Pop**: Removing the element from the top of the stack.
- **Underflow**: Attempting to pop from an empty stack.

### Visual Explanation:

Imagine a stack of plates. If you try to remove a plate when there are no plates left, that's similar to a stack underflow in computing.

### Example in JavaScript:

To illustrate stack underflow, let's implement a simple stack and handle underflow:

```javascript
class Stack {
    constructor() {
        this.items = [];
    }

    push(element) {
        this.items.push(element);
    }

    pop() {
        if (this.isEmpty()) {
            console.error("Stack Underflow: Cannot pop from an empty stack");
            return null; // Or handle it in some other way
        }
        return this.items.pop();
    }

    peek() {
        if (this.isEmpty()) {
            console.error("Stack is empty");
            return null;
        }
        return this.items[this.items.length - 1];
    }

    isEmpty() {
        return this.items.length === 0;
    }

    size() {
        return this.items.length;
    }
}

// Example usage:
const stack = new Stack();
stack.push(1);
stack.push(2);
stack.push(3);
console.log("Stack before pop operations:", stack.items);

console.log("Popped:", stack.pop()); // Should return 3
console.log("Popped:", stack.pop()); // Should return 2
console.log("Popped:", stack.pop()); // Should return 1

// This pop will cause underflow since the stack is now empty
console.log("Popped:", stack.pop()); // Should show an error message and return null
```

### Output:

```text
Stack before pop operations: [ 1, 2, 3 ]
Popped: 3
Popped: 2
Popped: 1
Stack Underflow: Cannot pop from an empty stack
Popped: null
```

### Explanation:

1. **Class Definition**: We define a `Stack` class with methods to push, pop, and check if the stack is empty.
2. **Push Method**: Adds an element to the stack.
3. **Pop Method**: Removes an element from the stack. If the stack is empty, it logs an error and returns `null`.
4. **isEmpty Method**: Checks if the stack is empty.
5. **Example Usage**: Demonstrates pushing elements to the stack and popping them off until the stack is empty, at which point a stack underflow error is handled gracefully.

### Preventing Stack Underflow:

To prevent stack underflow, always check if the stack is empty before performing a pop operation. This ensures that you do not attempt to remove an element from an empty stack, thus avoiding underflow errors.
