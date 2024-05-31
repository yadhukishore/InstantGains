### Null Pointer Exception in JavaScript

A "null pointer exception" in JavaScript isn't exactly the same as in languages like Java or C++, but similar errors can occur when you try to access a property or call a method on a `null` or `undefined` value. This typically results in a `TypeError`.

### Example

Consider the following example:

```javascript
let obj = null;
console.log(obj.name); // TypeError: Cannot read property 'name' of null
```

Here, trying to access `name` on `obj` throws a `TypeError` because `obj` is `null`.

### How to Avoid It

To prevent such errors, you can check if the variable is `null` or `undefined` before accessing its properties or methods.

#### Using `if` Statements

```javascript
let obj = null;

if (obj !== null && obj !== undefined) {
    console.log(obj.name);
} else {
    console.log('obj is null or undefined');
}
```

#### Using Optional Chaining (ES2020)

Optional chaining allows you to safely access deeply nested properties without having to explicitly check each level.

```javascript
let obj = null;

console.log(obj?.name); // undefined
```

In this example, `obj?.name` safely returns `undefined` instead of throwing an error if `obj` is `null` or `undefined`.

### Summary

- **Null Pointer Exception:** Trying to access a property or method on a `null` or `undefined` value causes a `TypeError`.
- **Prevention:** Check for `null` or `undefined` before accessing properties.
- **Optional Chaining:** Use `?.` to safely access properties on potentially `null` or `undefined` values.
