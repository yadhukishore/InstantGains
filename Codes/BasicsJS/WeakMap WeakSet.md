### WeakMap and WeakSet in JavaScript

**WeakMap** and **WeakSet** are collections introduced in ECMAScript 6 (ES6). They are similar to `Map` and `Set` but have some key differences, mainly concerning how they handle references to objects.

### WeakMap

A `WeakMap` is a collection of key-value pairs where the keys are objects and the values can be any arbitrary value. The primary feature of `WeakMap` is that it holds "weak" references to the keys, meaning that if there are no other references to the key object, it can be garbage-collected.

#### Characteristics of WeakMap

1. **Weak References to Keys:** If the only reference to a key object is within the `WeakMap`, the key can be garbage-collected.
2. **Non-Enumerable:** WeakMaps are not iterable. You cannot enumerate its keys, values, or entries.
3. **Object Keys Only:** The keys must be objects, not primitive values.

#### Example of WeakMap

```javascript
let weakMap = new WeakMap();

let obj = {name: 'Alice'};
weakMap.set(obj, 'some value');

console.log(weakMap.get(obj)); // 'some value'

obj = null; // Now the object is eligible for garbage collection

// After garbage collection, the entry is removed from the WeakMap
```

### WeakSet

A `WeakSet` is a collection of objects where each object can appear only once. Similar to `WeakMap`, the references to the objects in a `WeakSet` are weak, meaning the objects can be garbage-collected if there are no other references to them.

#### Characteristics of WeakSet

1. **Weak References to Items:** If an object in the `WeakSet` has no other references, it can be garbage-collected.
2. **Non-Enumerable:** WeakSets are not iterable. You cannot enumerate its items.
3. **Object Values Only:** The items in a `WeakSet` must be objects, not primitive values.

#### Example of WeakSet

```javascript
let weakSet = new WeakSet();

let obj = {name: 'Bob'};
weakSet.add(obj);

console.log(weakSet.has(obj)); // true

obj = null; // Now the object is eligible for garbage collection

// After garbage collection, the entry is removed from the WeakSet
```

### Key Differences from Map and Set

1. **Garbage Collection:**
   - `WeakMap` and `WeakSet` entries are held weakly, meaning they do not prevent their keys or items from being garbage-collected.
   - `Map` and `Set` hold strong references, preventing their entries from being garbage-collected as long as the `Map` or `Set` itself is referenced.

2. **Enumerability:**
   - `WeakMap` and `WeakSet` are not enumerable, so you cannot iterate over their entries.
   - `Map` and `Set` are iterable and provide methods for iteration, such as `forEach`, `keys`, `values`, and `entries`.

3. **Key and Value Types:**
   - `WeakMap` keys must be objects, and values can be any type.
   - `WeakSet` items must be objects.
   - `Map` keys and `Set` items can be of any type, including primitives.

### Use Cases

- **Caching:** `WeakMap` is useful for caching where you want to store information associated with an object but allow that information to be garbage-collected if the object is no longer needed.
- **DOM Manipulation:** `WeakSet` can be used to track DOM elements and avoid memory leaks by allowing the elements to be garbage-collected when they are removed from the document.

### Summary

- **WeakMap:** Collection of key-value pairs with object keys and weak references.
- **WeakSet:** Collection of unique objects with weak references.
- **Garbage Collection:** Entries in `WeakMap` and `WeakSet` are eligible for garbage collection if there are no other references.
- **Non-Enumerable:** Both `WeakMap` and `WeakSet` are not enumerable, meaning you cannot iterate over them.

Understanding `WeakMap` and `WeakSet` helps in managing memory more efficiently, particularly in applications dealing with a large number of objects or in scenarios requiring temporary associations.
