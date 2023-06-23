---
layout: post
title: 'Async Operations with AbortController & AbortSignal in Nodejs'
date: 2021-11-02 00:00:00 +0530
images: ['https://res.cloudinary.com/dkiurfsjm/image/upload/v1635836633/1_zfv_iZJYdjBUL2K1Y6k9yw_zjq7f8.jpg']
thumbnail: 'https://res.cloudinary.com/dkiurfsjm/image/upload/v1676698473/nodejs_dark_cjoudy.png'
tags: ['nodejs', 'javascript', 'async', 'abortcontroller', 'abortsignal']
keywords: 'nodejs,async,abortcontroller,javascript,abortsignal,operation'
categories: ['Nodejs']
url: 'nodejs/nodejs-async-operations-abortcontroller-abortsignal'
aliases:
    - /post/2021-11-02-abort-controller-nodejs
---

[AbortController](https://nodejs.org/dist/latest-v16.x/docs/api/globals.html#class-abortcontroller) is the standard way to abort any ongoing operations. AbortController and AbortSignal are now part of Nodejs LTS (originally introduced in v15.0.0).

Modern apps usually don’t work in isolation. They interact with entities like other APIs, file system, network, databases, etc. Each of these interactions adds an unknown delay to the handling of the incoming request. Sometimes, these interactions can take long time than expected resulting in overall delay in response. In such cases, the apps need a way to abort any operations that have taken longer than expected.

## Cancel Async Operations

Let us take a use case where we want to abort an operation if it takes more than  say 10 sec.

```javascript
const fooTask = () => {
  // some operation which can run longer than expected
  // e.g. 
  // file read/write operation
  // external api call
  // database operation
}

const timeOut = new Promise((resolve, reject) => {
  setTimeout(() => reject(new Error('operation timed out')), 10000);
});

await Promise.race([
  fooTask(),
  timeOut
]);

```

In the above scenario; if **fooTask()** operation takes longer time than expected say 10 sec, trigger a timeout. There are however two problems with the above code:

- Even though timeout is triggered; the **fooTask()** operation is not stopped
- Even if **fooTask()** operation completes within expected time, `setTimeout` timer will still be running and will eventually reject the promise with `unhandled promise rejection` error. This can cause performance/memory issues.

A more graceful way of handling above scenario is

- when timeout is triggered, it sends a signal to abort **fooTask()** operation
- when **fooTask()** operation is complete, it sends a signal to clear `setTimeout` timer

This is where `AbortController` and `AbortSignal`, now part of Nodejs core API comes into picture.


## AbortController & AbortSignal

`AbortController` is a utility class used to signal cancelation in selected Promise-based APIs. The API is based on the Web API [AbortController](https://developer.mozilla.org/en-US/docs/Web/API/AbortController).

`abortController.abort()`: Triggers the abort signal, causing the `abortController.signal` to emit the 'abort' event.

`abortController.signal`: Type: AbortSignal

`AbortSignal` extends `EventTarget` with a single type of event that it emits — the `abort` event. It also has an additional boolean property, `aborted` which is true if the AbortSignal has already been triggered.

Let us modify our code:

```javascript
const { setTimeout: setTimeoutPromise } = require('timers/promises');

const abortTimeout = new AbortController();
const abortFooTask = new AbortController();

const fooTask = async ({signal}) => {
  if (signal?.aborted === true) {
  	 throw new Error('operation aborted');
  }
  
  try {
    await setTimeoutPromise(100, undefined, {signal: abortFooTask.signal });
    
    console.log('fooTask done');
    console.log('signal ---> abort timeout');
    abortTimeout.abort();
	  // some operation which can run longer than expected
	  // e.g. 
	  // file read/write operation
	  // external api call
	  // database operation
  } catch (err) {
    if (err.name === 'AbortError') {
      console.log('fooTask was aborted');
    }
  }
}

const timeOut = async () => {
  try {
    await setTimeoutPromise(10, undefined, { signal: abortTimeout.signal });
    console.log('operation taking more time than expected');
    console.log('signal ---> abort fooTask');
    abortFooTask.abort();
  } catch (err) {
    if (err.name === 'AbortError') {
      console.log('The timeout was aborted');
    }
  }
};

const start = async () =>  {
  await Promise.race([
    fooTask({signal: abortFooTask.signal}),
    timeOut()
  ]);
}

start();

```

I have used `setTimeout` to simulate the operations.

### Output with:

 - timeout value of `10` for `timeout` operation and `100` for `fooTask` operation
 
 ```
 operation taking more time than expected
 signal ---> abort fooTask
 fooTask was aborted
 ```
 
 - timeout value of `100` for `timeout` operation and `10` for `fooTask` operation

 ```
 fooTask done
 signal ---> abort timeout
 The timeout was aborted
 ```


## Conclusion

`AbortController` and `AbortSignal` are now part of Nodejs core APIs. They provide a more graceful way of cancelling async operations.

✨ Thank you for reading and I hope you find it helpful. I sincerely request for your feedback in the comment's section. You can follow me on twitter [@lifeClicks25](https://twitter.com/lifeClicks25).

![](https://cdn-images-1.medium.com/max/1600/0*dMZ0BEHDv4MJYYGW.png)

> If you liked the above story, you can buy me a coffee to keep me energized for writing stories like this for you.