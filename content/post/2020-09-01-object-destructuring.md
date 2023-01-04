---
layout: post
title: 'ES6 Destructuring'
date: 2020-09-01 00:00:00 +0530
tags: ['javascript', 'es6', 'destructuring']
categories: ['Javascript']
keywords: 'javascript,object,destructuring,es6'
url: 'javascript/es6-destructuring'
aliases:
    - /post/2020-09-01-object-destructuring/
---

## **What is destructuring assignment?**

> The destructuring assignment syntax is a JavaScript expression that makes it possible to unpack values from arrays, or properties from objects, into distinct variables.

*-* [MDN](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Destructuring_assignment)

In essence, it is a convenient way to extract value/values from data stored in (possibly nested) objects and arrays. This feature was introduced in the 6th Edition of the ECMAScript standard, ES6 for short. It makes code concise, more readable and helps to avoid repeated destructing expressions.

## Use Cases

1. [**Access object properties and array items**](#usecase_1)
2. [**Create a shallow copy of objects**](#usecase_2)
3. [**Swap values**](#usecase_3)
4. [**Subset of a object's properties**](#usecase_4)
5. [**Get properties like length of array, function name, number of arguments etc.**](#usecase_5)

### **Access object properties and array items** {#usecase_1}

```javascript
const colors = ['red', 'blue'];
const [firstColor = 'white', secondColor = black] = colors;
console.log(firstColor, secondColor); // => red blue

const [, , thirdColor = 'black'] = colors;
console.log(thirdColor); // => black
```
Array destructuring helps to access array items in a much shorter way. Also, if the no element is present at any index then default value can be assigned like `thirdColor` in the given example.



```javascript
const obj = {bar: 'bar'};
const {foo = 'defaultFoo', bar: newBar} = obj;
console.log(foo, newBar); // => defaultFoo bar
```
Destructuring of objects is used more in general. In the above example, `foo` is assigned default value as the `obj` doesn't contains the `foo` property. `bar` property is destructured and renamed as `newBar`.

The same can be used to extract values from parameters into standalone variables

```javascript
// Assigning default values to destructured properties:

const f1 = ({ foo = "defaultFoo", bar }) => {
  console.log(foo, bar); // => defaultFoo bar
};
f1({ bar: 'bar' });

const f2 = ([ foo = "defaultFoo", bar ]) => {
  console.log(foo, bar); // => defaultFoo bar
};
f2([ ,'bar' ]);
```

### **Create a shallow copy of objects** {#usecase_2}
Using `... rest operator` and destructuring shallow copy of objects or array.

```javascript
const numbers = [1, 2, 3];
const [, ...restNumbers] = numbers;

restNumbers; // => [2, 3]
numbers; // => [1, 2, 3]
```
I find this very useful especially in case of objects to delete properties in an immutable way. e.g. returning a user object to an api call where sensitive information like password etc should not be returned.

```javascript
const user = {name: 'User X', age: 25, password: 'xyz'}
const {password, ...newUser} = user;

newUser // => { name: 'User X', age: 25 }
```

### **Swap values** {#usecase_3}
Variables can be swapped without need for a temp variable

```javascript
let a = 1;
let b = 2;

[a, b] = [b, a];

console.log(a, b); // => 2 1

const c = [1, 2, 3, 4];
[c[0], c[2]] = [c[2], c[0]]; // swap index 0 and 2

console.log(c); // => [ 3, 2, 1, 4 ]
```

### **Subset of a object's properties** {#usecase_4}

[Using destructuring assignment and ',' operator](https://stackoverflow.com/a/54613019)

```javascript
const object = { a: 5, b: 6, c: 7  };
const picked = ({a,c} = object, {a,c})

console.log(picked); // => { a: 5, c: 7 }
```

[Using Object Destructuring and Property Shorthand](https://stackoverflow.com/a/39333479)

```javascript
const obj = {a:1, b:2, c:3},
const subset = (({a, c}) => ({a, c}))(obj);

console.log(subset); // => { a: 1, c: 3 }
```

I have found these two above so useful, easy and very neat way to create a new object having subset of other object. Similarly, its very easy to convert an array to an object.

```javascript
const arr = ['a', 'b', 'c'];
const obj = (([a, b, c]) => ({a, b, c}))(arr);

console.log(obj); // => { a: 'a', b: 'b', c: 'c' }
``` 

### **Get properties like length of array, function name, number of arguments etc.** {#usecase_5}

```javascript
let arr = [1, 2, 3, 4, 5];

let { length } = arr;

console.log(length); // => 5

let func = function dummyFunc(a, b, c) {
  return 'A B and C';
}

let {name, length:funcLen} = func;

console.log(name, funcLen); // => dummyFunc 3
```

These are few very cool and helpful features of destructuring. I would highly recommend to read the chapter [Destructuring](https://exploringjs.com/es6/ch_destructuring.html) in Exploring ES6 by Dr. Axel Rauschmayer.

## Conclusion

ES6 introduced one of the most awaited JavaScript features: destructuring. It helps to write more cleaner and readable code. I hope you enjoyed reading this and find this helpful. I sincerely request for your feedback. You can follow me on twitter [@lifeClicks25](https://twitter.com/lifeClicks25).
