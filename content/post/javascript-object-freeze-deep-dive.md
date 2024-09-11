---
layout: post
title: 'Deep dive into Object.freeze() in Javascript'
description: "Discover Object.freeze() in JavaScript: Learn how it creates immutable objects, preventing modifications and ensuring the highest level of data integrity"
images: ['https://res.cloudinary.com/dkiurfsjm/image/upload/v1676446736/deep-dive_y0omgf.jpg']
thumbnail: 'https://res.cloudinary.com/dkiurfsjm/image/upload/v1675429691/JavaScript_v4qblf.jpg'
date: 2023-02-15 00:00:00 +0530
tags: ['javascript', 'es6','object']
categories: ['Javascript']
keywords: 'javascript,object,object.freeze(),es6,deep dive,object seal,object freeze,freeze,seal'
url: 'javascript/object-freeze-deep-dive'
---

According to MDN, the [Object.freeze()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/freeze) static method freezes an object. Freezing an object prevents extensions and makes existing properties non-writable and non-configurable. A frozen object can no longer be changed: new properties cannot be added, existing properties cannot be removed, their enumerability, configurability, writability, or value cannot be changed, and the object's prototype cannot be re-assigned. freeze() returns the same object that was passed in.

> Freezing an object is the highest integrity level that JavaScript provides.

![deep dive](https://res.cloudinary.com/dkiurfsjm/image/upload/v1676446736/deep-dive_y0omgf.jpg)

## A Closer look at JavaScript Object

[ECMA specification](https://tc39.es/ecma262/multipage/ecmascript-data-types-and-values.html#sec-object-type) defines an object as

> An Object is logically a collection of properties. Each property is either a data property, or an accessor property:

- A data property associates a key value with an ECMAScript language value and a set of Boolean attributes.

- An accessor property associates a key value with one or two accessor functions, and a set of Boolean attributes. The accessor functions are used to store or retrieve an ECMAScript language value that is associated with the property.

![object model](https://res.cloudinary.com/dkiurfsjm/image/upload/v1676451970/object-model_lje7ui.jpg)

*Image from ['JavaScript engine fundamentals: Shapes and Inline Caches'](https://mathiasbynens.be/notes/shapes-ics) by
 [Mathias Bynens](https://mathiasbynens.be)*

[Table: Attributes of an Object property](https://tc39.es/ecma262/multipage/ecmascript-data-types-and-values.html#sec-property-attributes)

| Attribute Name | Type | Description|
|:--------------:|:----:|:-----------|
|[[Value]]| data property| The value retrieved by a get access|of the property.|
|[[Writable]]| data property| If false, attempts by ECMAScript code to change the property's [[Value]] attribute using [[Set]] will not succeed.|
|[[Enumerable]]| data property| If true, the property will be enumerated by a for-in enumeration. Otherwise, the property is said to be non-enumerable.|
|[[Configurable]]| data property| If false, attempts to delete the property, change it from a data property to an accessor property or from an accessor property to a data property, or make any changes to its attributes (other than replacing an existing [[Value]] or setting [[Writable]] to false) will fail.|
|[[Get]]| accessor property| If the value is an Object it must be a function object. The function's [[Call]] internal method is called with an empty arguments list to retrieve the property value each time a get access of the property is performed.|
|[[Set]]| accessor property| If the value is an Object it must be a function object. It is called with an arguments list containing the assigned value as its sole argument each time a set access of the property is performed.|

## Understanding Property Descriptor {#ref_1}

The set of property attributes is called the property descriptor.

Let us look at the code snippet below:

```javascript
const obj = {};

Object.defineProperty(obj, 'id', {
  value: 100
});

console.log(obj);    // => { }
console.log(obj.id); // => 100

delete obj.id;       // try to delete the property id
console.log(obj.id); // => 100
```
The above code is defining a new property by using `Object.defineProperty` method which is a static method on Object that can define or modify a new property on a given object.

An `obj` is defined with property `id` having `[[value]]` as 100. Since, no other property attributes are specified hence, id has `writable`, `enumerable`, and `configurable` all set to `false`. So, `console.log(obj)` doesn't print `id` (`enumerable` is `false`) but `console.log(obj.id)` does.

Trying to delete `id` is unsuccesful as `configurable` is `false`.

Similary, let take a look below

```javascript
Object.defineProperty(obj, 'name', {
  value: 'Manish',
  writable: false,
  enumerable: true,
  configurable: true
});

console.log(obj.name);    	// => 'Manish'
obj.name = 'Manish Prasad'

console.log(obj.name); 		// => 'Manish' as writabel is false


Object.defineProperty(obj, 'lastName', {
  value: 'Prasad',
  writable: true,
  enumerable: false,
});

console.log(Object.keys(obj)); // => [ 'name' ]

obj.lastName = 'Sharma';
console.log(obj.lastName);     // => 'Sharma' as writable is true
```

## Freezing the Object

In JavaScript, the primitive types `number` and `string` etc are immutable by design. Whereas, objects are mutable. Since objects are passed by reference, there are valid scenarios where the object is modified unintentionally through one of its references. This leads to bugs which prove trickier to identify and solve.

Given the understanding of [property descriptor](#ref_1); there are three major ways of protecting objects in JavaScript.

## Object.preventExtensions()

The method `Object.preventExtensions()` allows an object to be made non-extensible, meaning that no new properties can be added to it in the future. It is important to note that existing properties can still be deleted. 

```javascript
const obj = {
  id: 100,
  country: 'IN'
};

Object.preventExtensions(obj);

obj.name = 'Manish';
delete obj.country;
console.log(obj);    // => { id: 100 } 
```

To determine if an object is non-extensible, you can use `Object.isExtensible()`. If this method returns true, you can still add new properties to the object.

## Object.seal()

The `Object.seal()` method, on the other hand, prevents new properties from being added and marks all existing properties as non-configurable. However, it still allows existing property values to be changed as long as they are writable. Object.seal effectively prevents the addition and/or removal of properties from the object. 

```javascript
const obj = {
  id: 100
};

Object.seal(obj);
delete obj.id 				// => won't delete property id

obj.name = 'Manish';
console.log(obj);			// => { id: 100 };

obj.id = 101
console.log(obj);			// => { id: 101 }

Object.isExtensible(obj);	// => false
Object.isSealed(obj);		// => true
```

You can test whether an object has been sealed using `Object.isSealed()`.

## Object.freeze()

The `Object.freeze()` method provides maximum protection for an object in JavaScript. It seals the object using `Object.seal()` and also prevents the modification of existing properties. It also prevents the descriptors from being changed since the object is already sealed.

```javascript
const obj = {
  id: 100
};

Object.freeze(obj);
delete obj.id 				// => won't delete property id

obj.name = 'Manish';
console.log(obj);			// => { id: 100 }

obj.id = 101
console.log(obj);			// => { id: 100 }

Object.isExtensible(obj);	// => false
Object.isSealed(obj);		// => true
Object.isFrozen(obj);		// => true
```

`Object.freeze()` is the most restrictive of the three methods, and you can use `Object.isFrozen()` to determine if an object is frozen.

**Note:** It's important to note that these methods only apply to the direct properties of an object and do not affect nested objects.

For freezing nested objects too you can write your custom `deepFreeze` method or rely on third party libraries like [immutablejs](https://immutable-js.com).

## Conclusion

I hope this article helps you understand about Javascript objects in more depth. The insights gained from this article will help property undeletable, property immutable or protect objects from unintentional modifications.

✨ Thank you for reading and I hope you find it helpful. I sincerely request for your feedback in the comment's section.

