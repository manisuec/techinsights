---
layout: post
title: 'Understand Event Loop in Nodejs'
date: 2021-08-11 00:00:00 +0530
images: ['/img/posts/eventloop.png']
tags: ['nodejs', 'javascript', 'eventloop']
---

[Event Loop](https://nodejs.org/en/docs/guides/event-loop-timers-and-nexttick/#what-is-the-event-loop) is what allows Node.js to perform non-blocking I/O operations — despite the fact that JavaScript is single-threaded — by offloading operations to the system kernel whenever possible.

Since most modern kernels are multi-threaded, they can handle multiple operations executing in the background. When one of these operations completes, the kernel tells Node.js so that the appropriate callback will eventually be executed.

## Code Snippet {#code_snippet}
Can you guess what will be the output of the below code snippet?
If yes; congratulations, you understand Event Loop completely.
If not; I would request you to read and understand details of each phase and then try to figure out the output.

```javascript
const fs = require('fs');

console.log('start');

// i/o operation
fs.readFile('/etc/hosts', (err, data) => {
  console.log('reading file');
});

process.nextTick(() => console.log('nextTick 1'));

// timers
setTimeout(() => {
  process.nextTick(() => console.log('nextTick 2'));
  console.log('setTimeout 1');
}, 0);
setImmediate(() => console.log('setImmediate'));
setTimeout(() => {
  process.nextTick(() => console.log('nextTick 3'));
  console.log('setTimeout 2')
}, 10);

console.log('hello event loop');

let counter = 0;
const timerLoop = setInterval(() => {
  console.log('setInterval', counter);

  if (counter >= 3) {
    console.log('exiting setInterval');
    clearInterval(timerLoop);
  }

  counter++;
}, 0);

// promise
new Promise((resolve, reject) => {
  console.log('start promise');
  resolve('promise data');
}).then(data => {
  console.log(data);
})

console.log('end');

```

## Six Phases of Event Loop

The Event Loop is composed of six phases, which are repeated for as long as the application still has code that needs to be executed:

- **timers:** this phase executes callbacks scheduled by `setTimeout()` and `setInterval()`.
- **pending callbacks:** executes I/O callbacks deferred to the next loop iteration.
- **idle, prepare:** only used internally.
- **poll:** retrieve new I/O events; execute I/O related callbacks (almost all with the exception of close callbacks, the ones scheduled by timers, and `setImmediate()`); node will block here when appropriate.
- **check:** `setImmediate()` callbacks are invoked here.
- **close callbacks:** some close callbacks, e.g. `socket.on('close', ...)`.

For detailed difference between timers and process.nextTick() read [Understand the difference between setImmediate(), setTimeout() and process.nextTick()](https://manisuec.blog/post/2021-07-30-timers-and-process-next-tick/)


![Event Loop](/img/posts/eventloop.png)


The six phases of event loop creates one cycle, or loop, which is known as a tick. A Node.js process exits when there is no more pending work in the Event Loop, or when process.exit() is called manually.

## Microtask Queue

This queue is broken down into two queues:

- first queue stores callback functions created by process.nextTick()
- second queue stores callback functions created by promise

Microtasks are executed after the main thread and each phase of the event loop. Microtasks created by process.nextTick() are executed before those created by  promise.

## Code Snippet Explained

Let us try to understand the [code snippet](#code_snippet)

The main thread will execute the code in `index.js` synchronously. It will add callbacks to their respective queues as and when encountered. Initial output 

```
start
hello event loop
start promise
end
```

Next callbacks were registered by `process.nextTick()` and `promise` and hence they will be executed.

```
nextTick 1
promise data
```

Event loop will then enter the timer phase and executes the callbacks which have expired. Also, during execution of callback registered by first `setTimeout()`; `process.nextTick()` registers a callback to `microtask queue`. Hence, it will be processed after the current operation is completed, regardless of the current phase of the event loop.

```
setTimeout 1
nextTick 2
```

`setInterval()` also registers a callback during the timer phase and is called repeatedly at 0ms interval but execution will happen in separate iterations.

```
setInterval 0
```

The next phase is `poll` phase but since some time is required to read data from file hence event loop will enter into `check` phase. Since, there is a callback registered by `setImmediate()`, it will gets executed.

```
setImmediate
```

This completes the first iteration of the loop. In the next iteration, it first enters the timer phase and executes expired timers callbacks.

```
setInterval 1
```

It then enters the `poll` phase and will execute callback registered during file reading operation.

```
reading file
```

In similar manner, further iterations of event loop will continue until there is no code to be executed.

```
setInterval 2
setInterval 3
exiting setInterval
setTimeout 2
nextTick 3
```

Hence, the output of the above [code snippet](#code_snippet) is

```
start
hello event loop
start promise
end
nextTick 1
promise data
setTimeout 1
nextTick 2
setImmediate
setInterval 0
setInterval 1
reading file
setInterval 2
setInterval 3
exiting setInterval
setTimeout 2
nextTick 3
```

## Conclusion

Node.js processes are single threaded and simplifies the complexity that comes with writing multithreaded code.  Event Loop helps offloading I/O tasks to the C++ APIs and determines the callback function that would be executed next at every iteration.

I hope you enjoyed reading this and find it helpful. I sincerely request for your feedback. You can follow me on twitter [@_manish25](https://twitter.com/_manish25).


