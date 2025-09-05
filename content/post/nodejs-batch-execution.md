---
layout: post
title: "Promise.all() Is Fine... Until It Isn’t!"
description: "Learn why Promise.all() can fail at scale and how a promise batch batch executor ensures controlled concurrency with timeouts & graceful error handling in Nodejs."
thumbnail: 'https://res.cloudinary.com/dkiurfsjm/image/upload/v1676698473/nodejs_dark_cjoudy.png'
date: 2025-08-22 00:00:00 +0530
lastmod: 2025-09-05T00:00:30
images: ['https://res.cloudinary.com/dkiurfsjm/image/upload/v1755859269/nodejs-concurrency_bq1fkm.jpg']
tags: ['nodejs', 'javascript', 'promise', 'concurrency']
keywords: 'nodejs,javascript,async,concurrency,batch executor,promise.all,performance'
categories: ['Nodejs']
url: 'nodejs/nodejs-batch-executor'
---

Nodejs developers often reach for `Promise.all()` when handling multiple asynchronous operations. However, this seemingly innocent approach can cause serious performance issues in production environments. Understanding the limitations of `Promise.all()` and implementing proper batch execution strategies is crucial for building robust, scalable Nodejs applications.

![nodejs batch execution](https://res.cloudinary.com/dkiurfsjm/image/upload/v1755859269/nodejs-concurrency_bq1fkm.jpg)

## Understanding the Promise.all Problem

The `Promise.all()` method executes all promises concurrently, which can lead to resource exhaustion and application crashes in production scenarios. This approach has three critical limitations that impact application performance and reliability.

### Resource Exhaustion Issues

When dealing with large numbers of concurrent operations, `Promise.all()` can overwhelm system resources:

```javascript
// ❌ Problematic approach
const urls = [...Array(1000).keys()].map(i => `https://api.example.com/data/${i}`);
await Promise.all(urls.map(url => fetch(url)));
```

This code attempts to open 1000 TCP connections simultaneously, potentially causing:
- Memory exhaustion
- Network timeout errors
- API rate limiting
- Application crashes

### Promise.all Limitations

The `Promise.all()` method has three fundamental limitations that make it unsuitable for production workloads:

1. **No Concurrency Control** - All operations execute simultaneously
2. **No Timeout Handling** - Single hanging promise blocks entire batch
3. **No Partial Recovery** - One failure causes all results to be lost
4. **Poor Error Handling** - If a single operation fails, the entire batch fails immediately

## Implementing Batch Execution Strategies

Batch execution provides controlled concurrency with proper error handling and timeout management. This approach ensures optimal resource utilization while maintaining application stability.

### Core Batch Executor Implementation

The batch executor manages concurrent operations with configurable limits and graceful error handling:

```javascript
/**
 * Batch Executor
 * Runs async tasks with concurrency, per-task timeout, and graceful errors.
 */

export default async function batchExecutor({
  tasks,               // Array<() => Promise<any>>
  maxConcurrency = 5,  // How many run at once
  maxTimeout,          // Milliseconds before a task is aborted
  onError              // (err, index) => void  — optional centralized logger
}) {
  if (!Array.isArray(tasks) || tasks.length === 0) return [];

  const results = new Array(tasks.length).fill(null);
  let cursor = 0;

  // Helper that wraps a promise with a timeout
  function withTimeout(promise, ms, idx) {
    if (!ms) return promise;
    return new Promise((resolve, reject) => {
      const timer = setTimeout(() => {
        reject(new Error(`Task ${idx} timed out after ${ms}ms`));
      }, ms);

      promise
        .then(resolve)
        .catch(reject)
        .finally(() => clearTimeout(timer));
    });
  }

  // One worker = one seat in the club
  async function worker() {
    while (cursor < tasks.length) {
      const index = cursor++;
      try {
        const task = tasks[index];
        results[index] = await withTimeout(task(), maxTimeout, index);
      } catch (err) {
        results[index] = null; // keep the array length stable
        if (onError) onError(err, index);
        else throw err; // fail-fast if no custom handler
      }
    }
  }

  // Spin up N workers
  const workers = Array.from(
    { length: Math.min(maxConcurrency, tasks.length) },
    () => worker()
  );
  await Promise.all(workers);
  return results;
}
```

### Timeout Management

Proper timeout handling prevents individual operations from blocking the entire batch:

```javascript
// ✅ Proper timeout implementation
function withTimeout(promise, ms, idx) {
  if (!ms) return promise;
  return new Promise((resolve, reject) => {
    const timer = setTimeout(() => {
      reject(new Error(`Task ${idx} timed out after ${ms}ms`));
    }, ms);

    promise
      .then(resolve)
      .catch(reject)
      .finally(() => clearTimeout(timer));
  });
}
```

## Practical Implementation Examples

Batch execution can be applied to various real-world scenarios, from file processing to API calls and database operations.

### File Download Management

Download multiple files with controlled concurrency and timeout protection:

```javascript
import batchExecutor from './batch.js';
import fs from 'fs/promises';
import fetch from 'node-fetch';

const urls = [...Array(100).keys()].map(i => `https://api.example.com/files/${i}`);

const tasks = urls.map(url => async () => {
  const res = await fetch(url);
  const buffer = await res.buffer();
  await fs.writeFile(`./downloads/${Date.now()}.jpg`, buffer);
  return url;
});

await batchExecutor({
  tasks,
  maxConcurrency: 3,
  maxTimeout: 10_000,
  onError: (err, idx) => console.error(`Download ${idx} failed:`, err.message)
});
```

### Mixed Operation Types

Handle different types of asynchronous operations within the same batch:

```javascript
const tasks = [
  async () => (await db.query('SELECT COUNT(*) FROM users')).rows[0].count,
  async () => (await fetch('https://api.external.com/data')).json(),
  async () => fs.readFile('./config.json', 'utf8')
];

const results = await batchExecutor({
  tasks,
  maxConcurrency: 2,
  maxTimeout: 5_000,
  onError: (err, i) => console.warn(`Job ${i} failed:`, err.message)
});
```

## Performance Comparison and Best Practices

Understanding when to use batch execution versus other concurrency patterns is crucial for optimal application performance.

### Feature Comparison

| _Feature_ | _Promise.all()_ | _Batch Executor_ |
| --- | --- | --- |
| Runs everything at once   | ✅          | ❌ (throttled) |
| Keeps order               | ✅          | ✅             |
| Per-task timeout          | ❌          | ✅             |
| Partial failure handling  | ❌          | ✅             |
| Resource exhaustion guard | ❌          | ✅             |

### Concurrency Optimization

Choose appropriate concurrency levels based on your specific use case:

- **CPU-intensive tasks**: Lower concurrency (2-4)
- **I/O-bound operations**: Higher concurrency (10-20)
- **Network requests**: Moderate concurrency (5-10)

### Error Handling Strategies

Implement robust error handling for production environments:

```javascript
// ✅ Comprehensive error handling
const results = await batchExecutor({
  tasks,
  maxConcurrency: 5,
  maxTimeout: 5000,
  onError: (err, index) => {
    console.error(`Task ${index} failed:`, err.message);
    // Log to monitoring service
    // Send alert if critical
    // Retry logic if appropriate
  }
});
```

## Advanced Batch Processing Techniques

For complex scenarios, consider implementing additional features like retry logic, progress tracking, and dynamic concurrency adjustment.

### Retry Logic Implementation

Add automatic retry capabilities for failed operations:

```javascript
async function withRetry(fn, maxRetries = 3, delay = 1000) {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await fn();
    } catch (error) {
      if (i === maxRetries - 1) throw error;
      await new Promise(resolve => setTimeout(resolve, delay * (i + 1)));
    }
  }
}

const tasks = urls.map(url => () => withRetry(() => fetch(url)));
```

### Progress Tracking

Monitor batch execution progress for better user experience:

```javascript
function createProgressTracker(total) {
  let completed = 0;
  return {
    increment: () => {
      completed++;
      console.log(`Progress: ${completed}/${total} (${Math.round(completed/total*100)}%)`);
    }
  };
}
```

## Related Articles
- [Understanding Nodejs Event Loop](https://techinsights.manisuec.com/nodejs/nodejs-event-loop/) - Master the event loop for better async handling
- [Working with Streams in Nodejs](https://techinsights.manisuec.com/nodejs/nodejs-streams) - Process large datasets efficiently
- [Handling Backpressure in Nodejs Streams](https://techinsights.manisuec.com/nodejs/backpressure-stream-optimization) - Manage data flow control
- [Common Pitfalls with Async/Await in forEach Loops](https://techinsights.manisuec.com/nodejs/pitfalls-of-async-await-foreach-loop) - Avoid async/await mistakes

## Final Thoughts: Use batch executor instead of `Promise.all()`

Batch execution provides a robust alternative to `Promise.all()` for production Nodejs applications. By implementing controlled concurrency, proper timeout handling, and graceful error recovery, developers can build applications that scale efficiently while maintaining stability under load.

The batch executor pattern is particularly valuable for:

- Processing large datasets
- Managing API rate limits
- Handling file operations
- Database batch operations

Remember that successful batch processing requires careful consideration of concurrency levels, timeout values, and error handling strategies tailored to your specific use case. Start with conservative settings and adjust based on monitoring and performance metrics.

✨ Thank you for reading and I hope you find it helpful. I sincerely request for your feedback in the comment's section.
