---
layout: post
title: 'Deep dive into Object.freeze() in Javascript'
images: ['https://res.cloudinary.com/dkiurfsjm/image/upload/v1676446736/deep-dive_y0omgf.jpg']
thumbnail: 'https://res.cloudinary.com/dkiurfsjm/image/upload/v1675429691/JavaScript_v4qblf.jpg'
date: 2023-15-02 00:00:00 +0530
tags: ['javascript', 'es6','object']
categories: ['Javascript']
keywords: 'javascript,object,object.freeze(),es6, deep dive'
url: 'javascript/object-freeze-deep-dive'
---

The [Object.freeze()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/freeze) static method freezes an object. Freezing an object prevents extensions and makes existing properties non-writable and non-configurable. A frozen object can no longer be changed: new properties cannot be added, existing properties cannot be removed, their enumerability, configurability, writability, or value cannot be changed, and the object's prototype cannot be re-assigned. freeze() returns the same object that was passed in.

Freezing an object is the highest integrity level that JavaScript provides.

## A Closer look at JavaScript Object

## Conclusion


✨ Thank you for reading and I hope you find it helpful. I sincerely request for your feedback in the comment's section.