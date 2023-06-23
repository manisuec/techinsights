---
layout: post
title: 'What is finally method and finally block in Javascript?'
images: ['https://res.cloudinary.com/dkiurfsjm/image/upload/v1687259385/ts_url_jydkn8.png']
thumbnail: 'https://res.cloudinary.com/dkiurfsjm/image/upload/v1675429691/JavaScript_v4qblf.jpg'
date: 2023-06-23 00:00:00 +0530
tags: ['javascript', 'finally']
categories: ['Javascript']
keywords: 'javascript,finally,finally method,finally block'
url: 'javascript/finally-method-and-block'
---

The word `finally` in Javascript is used in two contexts mainly; [`finally` method with Promises](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise/finally) and [`finally {}` block with try-catch](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/try...catch).

People intermittently use the `finally` keyword for both the context and have the assumption that they behave the same. However, there is slight difference in behaviour. The purpose of `finally` in both the context, however, remains the same.

- to perform cleanup tasks after the promise has been settled
- finally block is used to perform cleanup tasks post `try...catch` execution

![](https://res.cloudinary.com/dkiurfsjm/image/upload/v1687507931/finally_udjaiz.jpg)

## Promise.finally()

Using code snippet from MDN site [Using finally()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise/finally)

```
let isLoading = true;

fetch(myRequest)
  .then((response) => {
    const contentType = response.headers.get("content-type");
    if (contentType && contentType.includes("application/json")) {
      return response.json();
    }
    throw new TypeError("Oops, we haven't got JSON!");
  })
  .then((json) => {
    /* process your JSON further */
  })
  .catch((error) => {
    console.error(error); // this line can also throw, e.g. when console = {}
  })
  .finally(() => {
    isLoading = false;
  });
```

When a promise settles i.e. either `fullfilled` or `rejected`; you can finally do something.

The finally method of promises is useful when you want to do something after the promise has settled. This can be cleanup or code you may want to duplicate in the then and catch methods. 

The finally method is called regardless of the outcome (fulfilled or rejected) of the promise. It has absolutely no impact on the value that your promise will resolve to - it doesn't even have access to it. Let us take a look at below code snippet:

```
Promise.resolve(100).finally(() => 250).then((val) => 2*val).finally(() => 500).then((val) => {console.log(val)})
```

The above code will print `200` instead of `500` returned from finally method.

## finally {} in try/catch blocks

From MDN docs; [`the finally block`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/try...catch#the_finally_block)

> The `finally` block contains statements to execute after the try block and catch block(s) execute, but before the statements following the `try...catch...finally` block. Control flow will always enter the finally block, which can proceed in one of the following ways:
>
- Immediately before the try block finishes execution normally (and no exceptions were thrown);
- Immediately before the catch block finishes execution normally;
- Immediately before a control-flow statement (return, throw, break, continue) is executed in the try block or catch block.

> If an exception is thrown from the try block, even when there's no catch block to handle the exception, the finally block still executes, in which case the exception is still thrown immediately after the finally block finishes executing.

```
try {
    // some code
  } catch (ex) {
    // error handling
  } finally {
    // finally block code
  }
```

Let us take a look at below code snippets:

```
const fun1 = () => {
  try {
    return 100;
  } catch (error) {
    return `Error: ${error}`;
  } finally {
    console.log('finally block called!');
  }
}

const fun2 = () => {
  try {
    return 100;
  } catch (error) {
    return `Error: ${error}`;
  } finally {
    console.log('finally block called!');
    return 500;
  }
}

fun1(); 
// finally block called!
// 100 ---> 100 is printed as expected

fun2();
// finally block called!
// 500 
// return value overridden by return value of finally block
```
This behaviour is different than `Promise.finally()`.

> If the finally block returns a value, this value becomes the return value of the entire try-catch-finally statement, regardless of any return statements in the try and catch blocks. This includes exceptions thrown inside of the catch block.

**Reason:** According to ECMA-262 (5ed, December 2009), in pp. 96:

> The production TryStatement : try Block Finally is evaluated as follows:
> 
> 
> - Let B be the result of evaluating Block.
> - Let F be the result of evaluating Finally.
> - If F.type is normal, return B.
> - Return F.

And from pp. 36:

> The Completion type is used to explain the behaviour of statements (break, continue, return and throw) that perform nonlocal transfers of control. Values of the Completion type are triples of the form (type, value, target), where type is one of normal, break, continue, return, or throw, value is any ECMAScript language value or empty, and target is any ECMAScript identifier or empty.

It's clear that return false would set completion type of finally as return, which cause try ... finally to do 4. Return F.

Read more at [this stackoverflow thread.](https://stackoverflow.com/questions/3837994/why-does-a-return-in-finally-override-try)

## Conclusion

I hope this article helps you understand about the subtle difference between `Promise.finally()` and `try...catch...finally` block. The purpose of `finally` in both context is same like clean up tasks.

✨ Thank you for reading and I hope you find it helpful. I sincerely request for your feedback in the comment's section.

![](https://cdn-images-1.medium.com/max/1600/0*dMZ0BEHDv4MJYYGW.png)

> If you liked the above story, you can buy me a coffee to keep me energized for writing stories like this for you.