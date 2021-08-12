---
layout: post
title: 'Understand the difference between setImmediate(), setTimeout() and process.nextTick()'
date: 2021-07-30 00:00:00 +0530
images: ['/img/posts/timers.png']
tags: ['nodejs', 'javascript', 'timers']
---

Timers and process.nextTick() are core concepts of Nodejs. It is important to undestand and be familiar with these and use them with ease. `setTimeout(callback, 0)`, `setImmediate(callback)` and `process.nextTick(callback)` appear to be doing the same thing. I have tried to make this concept easier to understand.

## Code Snippet {#code_snippet}

Let’s look at a code snippet first:

```javascript
setTimeout(() => { 
  process.nextTick(() => {
    console.log('setTimeout --> nextTick');
  });
  console.log('setTimeout');
}, 0);

setImmediate(() => {
  process.nextTick(() => {
    console.log('setImmediate --> nextTick');
  });
  console.log('setImmediate');
});

process.nextTick(() => {
  console.log('nextTick');
});

console.log('hello timers and process.nextTick()');
```

What do you think the output will be?
Let us try to figure out....

`setImmediate()` and `setTimeout()` are similar, but behave in different ways depending on when they are called.

`setImmediate()` is designed to execute a script once the current poll phase completes.

`setTimeout()` schedules a script to be run after a minimum threshold in ms has elapsed.

If both the timers are called within an I/O cycle, the immediate callback is always executed first else the execution order is non-deterministic and depends on the performance of the process. [Read more here...](https://nodejs.org/en/docs/guides/event-loop-timers-and-nexttick/#setimmediate-vs-settimeout)

```
# Non-deterministic execution order

setTimeout(() => {
  console.log('timeout');
}, 0);

setImmediate(() => {
  console.log('immediate');
});
```

```
# immediate callback will be executed first always

const fs = require('fs');

fs.readFile(__filename, () => {
  setTimeout(() => {
    console.log('timeout');
  }, 0);
  setImmediate(() => {
    console.log('immediate');
  });
});
```
`process.nextTick()` takes a callback and adds it to `nextTickQueue` and will be processed after the current operation is completed, regardless of the current phase of the event loop. Any time you call `process.nextTick()` in a given phase, all callbacks passed to process.nextTick() will be resolved before the event loop continues.

Given this understanding, can you now guess the output
of [code snippet](#code_snippet)

```
hello timers and process.nextTick()
nextTick
setTimeout
setTimeout --> nextTick
setImmediate
setImmediate --> nextTick
```


## Conclusion
They all appear to do the same thing but one should use them depending on the situation. I would strongly recommend to read [Understand Event Loop in Nodejs](https://manisuec.blog/post/2021-08-11-understand-event-loop-in-nodejs/) for better understanding.

The [official documentation](https://nodejs.org/en/docs/guides/event-loop-timers-and-nexttick/#check) states:

> We recommend that developers use setImmediate() in all cases because it’s easier to think about.

`process.nextTick()` can potentially cause I/O starvation and hence should be used responsibly.
I hope you enjoyed reading this and find it helpful. I sincerely request for your feedback. You can follow me on twitter [@_manish25](https://twitter.com/_manish25).
