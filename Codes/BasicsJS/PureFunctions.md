## Pure Functions
A pure function is a function that follows two main principles:

1. **Deterministic**: Given the same input, the function will always return the same output. This means that the function does not produce any side effects (like modifying an external variable or object, generating random numbers, writing to the console, etc.).

2. **No Side Effects**: The function does not modify or rely on any external state. It only depends on its input arguments and does not mutate or affect anything outside of its scope.

Here's an example of a pure function in JavaScript:

```javascript
function add(a, b) {
  return a + b;
}

console.log(add(2, 3)); // Output: 5
console.log(add(2, 3)); // Output: 5 (same output for the same input)
```

The `add` function takes two arguments `a` and `b`, adds them together, and returns the result. It doesn't modify any external variables or objects, and it doesn't produce any side effects like console logs or random number generation. Given the same input arguments `a` and `b`, it will always return the same output.

Here's an example of an impure function:

```javascript
let counter = 0;

function incrementCounter() {
  counter++; // Modifying an external variable
  return counter;
}

console.log(incrementCounter()); // Output: 1
console.log(incrementCounter()); // Output: 2 (different output for the same input)
```

The `incrementCounter` function is impure because it modifies an external variable `counter`. Additionally, it doesn't return the same output for the same input (calling it multiple times without arguments will give different results).

Pure functions have several benefits:

1. **Predictable**: Since pure functions always return the same output for the same input, they are easier to reason about and test.
2. **Reusable**: Pure functions can be reused in different parts of an application without worrying about side effects or state management.
3. **Parallelizable**: Pure functions can be executed in parallel without any risk of race conditions or data inconsistencies, as they don't depend on or modify any shared state.
4. **Cacheable**: The output of a pure function can be cached, as it will always return the same result for the same input.

In functional programming paradigms, such as those used in React, Redux, and other libraries/frameworks, pure functions are highly valued for their simplicity, testability, and avoidance of side effects.

It's important to note that while pure functions are desirable in many situations, they are not a strict requirement in JavaScript programming. Impure functions may be necessary in certain cases, such as when interacting with external systems or when managing state in an application. However, developers should strive to make functions as pure as possible to benefit from the advantages of pure functions.

## Does Pure Fuctions can avoid the issue of call By Reference?

Yes, that's correct. Pure functions inherently avoid the issues related to call by reference in JavaScript because they do not mutate any external data or objects.

In JavaScript, when you pass an object (including arrays and functions) as an argument to a function, it is passed by reference. This means that the function receives a reference to the original object in memory, not a copy of the object. If the function modifies the object, it will affect the original object outside the function.

However, pure functions by definition do not modify any external state or objects. They simply take input arguments, perform some calculations or transformations on those arguments, and return a new value or object without mutating the original inputs.

Consider the following impure function that modifies an object passed as an argument:

```javascript
const person = { name: 'Alice', age: 25 };

function incrementAge(obj) {
  obj.age++; // Modifying the original object
}

incrementAge(person);
console.log(person.age); // Output: 26
```

In this case, the `incrementAge` function receives a reference to the `person` object and modifies its `age` property directly. As a result, the original `person` object is mutated outside the function.

Now, let's rewrite the `incrementAge` function as a pure function:

```javascript
const person = { name: 'Alice', age: 25 };

function incrementAge(obj) {
  const newObj = { ...obj, age: obj.age + 1 }; // Creating a new object without mutating the original
  return newObj;
}

const updatedPerson = incrementAge(person);
console.log(updatedPerson.age); // Output: 26
console.log(person.age); // Output: 25
```

In this example, the `incrementAge` function creates a new object `newObj` by spreading the properties of the original `obj` and then creating a new `age` property with the incremented value. It returns this new object without modifying the original `obj`. As a result, the original `person` object remains unchanged, and the updated object is returned and assigned to `updatedPerson`.

By following the principles of pure functions and avoiding mutation of external data or objects, you can effectively avoid the issues related to call by reference in JavaScript. Pure functions create new objects or values based on their inputs, rather than modifying the original inputs, which makes them safer and easier to reason about.
