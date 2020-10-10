---
title: Just JavaScript Notes
date: "2020-06-06T22:12:03.284Z"
description: "Notes from the course."
---

My notes from the [Just JavaScript](https://www.justjavascript.com) course made by [Dan Abramov](https://twitter.com/dan_abramov) and [Maggie Appleton](https://maggieappleton.com/). All illustrations are belong to them.

## Counting Values

### Assignment

`=` means "connect the left side's wire to the value on the right side.

Happens in tree steps:

1. Find the wire on the left.
2. Find the value on the right.
3. Point the wire to that value.

`NaN` is a special number that shows up when we do invalid math like `0 / 0`.

### Objects

`{}` always means "create a new object value".

### Functions

`()` always means "create a new function value".

## Equality

### Kinds

- Strict equality: `a === b`.
- Loose equality: `a == b`.
- Same value equality: `Object.is(a, b)`.

### Same Value Equality

`Object.is(a, b)` tells us if a and b are the same value.

```js
Object.is(2, 2) // true
Object.is({}, {}) // false
```

Has direct meaning in mental model - is (points to) the same value in our universe.

### Strict Equality

```js
2 === 2 // true
{} === {} // false
```

`2 === 2` is `true` because `2` always "summons" the same value.

`{} === {}` is `false` because each `{}` creates a different value.

The same as Same Value Equality in most cases. Different in two cases:

`NaN === NaN` is `false`, although they are the same value.

`-0 === 0` and `0 === -0` are `true` although they are different values.

```js
// Doesn't work
function resizeImage(size) {
  if (size === NaN) {
    // Doesn't work: the check is always false!
    console.log("Something is wrong.")
  }
}

// Ways to make it work
Number.isNaN(size)
Object.is(size, NaN)
size !== size
```

## Properties

Unlike variables, properties _belong_ to a particular object.
Both variables and properties act like wires, however, wires of properties start from objects rather than from our code.

- Properties don't contain values - they point at them.
- Properties are the wires.

_All wires always point at values._

### Reading properties

Properties can be read using the "dot notation":
![](https://res.cloudinary.com/dg3gyk0gu/image/upload/v1584416479/just-javascript-email-images/jj07/sherlock_readage.gif)

Or by "bracket notation":
`sherlock[propertyName]`.

### Assigning properties

```js
let sherlock = {
  surname: "Holmes",
  age: 64,
}

sherlock.age = 65
```

We follow the sherlock wire to the object, and then _pick_ the age property wire. We don't follow that wire to `64`, because we don't care about its current value when assigning a new value.

`sherlock` and `age` are both wires. On the left of `=`, we follow wires, on the right we summon the new value.

### Missing properties

```js
sherlock.boat // ?
```

JavaScript follows certain rules to decide which value to answer with:

1. Figure out the value of the part before the dot (.).
2. If that value is null or undefined, throw an error immediately.
3. Check whether a property with that name exists in our object.
   - If it exists, answer with the value this property points to.
   - If it doesn’t exist, answer with the undefined value.

_(Rules are simplified)_

```js
sherlock.boat // undefined
```

Does not mean that the object has a property `boat` that points to `undefined`. It is a question to the JavaScript engine, and the engine follows the rules to answer it.

## Mutation

- Objects are never “nested” in our universe.
- Pay close attention to which wire is on the left side of assignment.
- Changing an object’s property is also called mutating that object.
- If you mutate an object, your code will “see” that change via any wires pointing at that object. Sometimes, this may be what you want. However, mutating accidentally shared data may cause bugs.
- Mutating the objects you’ve just created in code is safe. Broadly, how much you’ll use mutation depends on your app’s architecture. Even if you won’t use it a lot, it’s worth your time to understand how it works.
- You can declare a variable with const instead of let. That allows you to enforce that this variable’s wire always points at the same value. But remember that const does not prevent object mutation!

## Prototypes

_Any JS object may choose another object as a prototype._

![](https://res.cloudinary.com/dg3gyk0gu/image/upload/v1590534270/just-javascript-email-images/jj09/proto_anim.gif)

`__proto__` instructs JS to continue looking for missing properties on another object.

```js
let human = {
  teeth: 32,
}

let gwen = {
  __proto__: human,
  age: 19,
}

console.log(human.age) // undefined
console.log(gwen.age) // 19

console.log(human.teeth) // 32
console.log(gwen.teeth) // 32

console.log(human.tail) // undefined
console.log(gwen.tail) // undefined
```

> This serves to remind us that gwen.teeth is an expression — a question to the JavaScript universe — and JavaScript will follow a sequence of steps to answer it. Now we know that these steps involve looking at the prototype.

### The prototype chain

_A prototype is more like a relationship than a "thing" in the JS._

The prototype chain is the sequence of objects asking their prototype for a value until they get that value or not (`undefined`). However, prototype chains can't be circular.

### Shadowing

```js
let human = {
  teeth: 32,
}

let gwen = {
  __proto__: human,
  // This object has its own teeth property:
  teeth: 31,
}
```

Both objects have `teeth` as a property, so `gwen.teeth` would not look for that property in its prototype. Once the property is found, the search stops.

To check if an object has its own property with a certain name, use `hasOwnProperty`, which returns `true` if it exists on the object, and doesn't look at the prototypes.

### Assignment

Assignment of a property on an object doesn't have any effect on the prototype:

```js
let human = {
  teeth: 32,
}

let gwen = {
  __proto__: human,
  // Note: no own teeth property
}

gwen.teeth = 31

console.log(human.teeth) // 32
console.log(gwen.teeth) // 31
```

`gwen.teeth = 31` creates a new own property called `teeth` on the object that `gwen` point to.

### The Object Prototype

`{}` not only creates an empty object. It also creates a hidden `__proto__` wire from that object to the Object Prototype.

```js
let obj = {}
console.log(obj.__proto__)
```

![](https://res.cloudinary.com/dg3gyk0gu/image/upload/v1590534071/just-javascript-email-images/jj09/root1.png)

The Object Prototype has built-in properties.

### An Object with No Prototype

```js
let weirdo = {
  __proto__: null,
}
```

Now the object has no prototype, and no built-in object methods.

```js
console.log(weirdo.hasOwnProperty) // undefined
console.log(weirdo.toString) // undefined
```

The Object Prototype itself is an object with no prototype.

### Polluting the Prototype

> If JavaScript searches for missing properties on the prototype, and most objects share the same prototype, can we make new properties “appear” on all objects by mutating that prototype?

> The answer is yes!

```js
let obj = {}
obj.__proto__.smell = "banana"

console.log(sherlock.smell) // "banana"
console.log(watson.smell) // "banana"
```

Mutating a shared prototype is called _prototype pollution_.

Try to avoid it, as it's fragile and makes it hard to add new language features.

### \_\_proto\_\_ vs prototype

The `prototype` property is almost entirely unrelated to the core mechanism of prototypes. It rather relates to the `new` operator.
