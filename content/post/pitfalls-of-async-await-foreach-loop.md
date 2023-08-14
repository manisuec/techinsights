---
layout: post
title: 'The Pitfalls of Using Async/Await Inside forEach() Loops'
date: 2023-08-14 00:00:00 +0530
images: ['https://res.cloudinary.com/dkiurfsjm/image/upload/v1692000996/foreach_cozdhs.png']
thumbnail: 'https://res.cloudinary.com/dkiurfsjm/image/upload/v1675429691/JavaScript_v4qblf.jpg'
tags: ['nodejs', 'async', 'javascript']
keywords: 'nodejs,async,asynchronous,await,promise,foreach'
categories: ['Javascript']
url: 'nodejs/pitfalls-of-async-await-foreach-loop'
---

When it comes to running asynchronous operations for each element in an array, the instinct is often to turn to the trusty `.forEach()` method. It seems like the perfect solution, right? However, there's a catch – a catch that might leave you scratching your head when things don't work as expected.

![](https://res.cloudinary.com/dkiurfsjm/image/upload/v1692000996/foreach_cozdhs.png)

### Why async/await Inside forEach May Catch You Off Guard

Imagine this scenario: you have an array of user IDs, and for each user ID, you need to fetch the user and push their username into an array. Eventually, you want to log all the usernames. But, much to your surprise, the array remains empty. What gives?

```javascript
const userIds = [1, 2, 3];
const userNameArr = [];

userIds.forEach(async (userId) => {
  const user = await fetchUserById(userId);
  userNameArr.push(user.username);
});

// this prints an empty array, no usernames here 🙁
console.log(userNameArr);
```

To truly grasp what's happening, let's dive into the inner workings of this code. And what better way to understand it than through an animated depiction of the JavaScript runtime as it meticulously executes each line of your program?

![](https://res.cloudinary.com/dkiurfsjm/image/upload/v1692000462/ezgif-3-c2c078f152_wi9oaq.gif)

*Animation Courtesy: [Maxim Orlov](https://maximorlov.com/async-await-inside-foreach/)*

### Deep Dive into the Code

As the code starts to run, you'll notice something intriguing. The `fetchUserById()` requests are initiated concurrently within the `forEach()` method. These requests are then dispatched to background tasks, waiting until they're completed – whether it's through WebAPIs in a browser or the C++ thread pool in Node.js.

With the requests underway, the program forges ahead and logs the usernames array. However, don't be surprised to find an empty array staring back at you. Why, you ask?

Here's the twist: functional JavaScript methods like `forEach()` are oblivious to promises. The callback functions they execute fire synchronously, without any concern for promises that might be lurking in between iterations.

And then, after a bit of time has passed, the requests wrap up, and the usernames array finally gets populated. This happens after the program has moved on and exited, leaving the usernames lingering in obscurity. 

### Ensure Code Execution Once All Asynchronous Tasks have Concluded

[`for...of` loop](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/for...of) is one of the best way for such kind of scenarios. 

```javascript
const userIds = [1, 2, 3];
const userNameArr = [];

for (const userId of userIds) {
   const userName = await fetchUserById(userId);
   userNameArr.push(userName);
}

console.log(userNameArr);
```

Alternatively, `Promise.all()` and `Array.map()` methods can also be used to achieve the same.

```javascript
const userIds = [1, 2, 3];

// Map each userId to a promise that eventually fulfills
// with the username
const promises = userIds.map(async (userId) => {
  const user = await fetchUserById(userId);
  return user.username;
});

// Wait for all promises to fulfill first, then log
// the usernames
const usernames = await Promise.all(promises);
console.log(usernames);
```


### Conclusion

Remember, in the realm of asynchronous JavaScript, understanding the intricacies of operations like `forEach` and the elegance of `async/await` can make all the difference between code that works flawlessly and code that leaves you scratching your head. So, before you dive into your next array-based adventure, take a moment to consider the asynchronous journey that lies ahead. Your future self will thank you!

✨ Thank you for reading and I hope you find it helpful. I sincerely request for your feedback in the comment's section.