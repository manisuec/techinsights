---
background: /img/posts/01.png
date: '2018-08-18T00:00:00Z'
title: Clustering & Inter Process Communication (IPC) in NodeJS
tags: ['nodejs', 'ipc', 'clustering']
---

## Introduction

A single instance of Node.js runs in a single thread. This does not allow to take advantage of multi-core systems automatically. However, by leveraging [‘cluster module’](https://nodejs.org/dist/latest-v10.x/docs/api/cluster.html) one can take advantage of multiple CPU cores. Clustering improves your app’s performance and lets you achieve zero downtime (hot) deployments easily. Also keep in mind that number of workers that can be created is not limited by the number of CPU cores of the machine. Clustering is a must-have for any Node.js production app and there is no reason not to use it.

## Clustering

Run the below code

```javascript
'use strict';

const cluster = require('cluster');
const numCPUs = require('os').cpus().length;

if (cluster.isMaster) {
  console.log('Master process is running with pid:', process.pid);
  
  for (let i = 0; i < numCPUs; ++i) {
    cluster.fork();
  }
} else {
  console.log('Worker started with pid:', process.pid);
}
```

```javascript
$ node cluster.js
Master process is running with pid: 21559
Worker started with pid: 21570
Worker started with pid: 21571
Worker started with pid: 21565
Worker started with pid: 21577
```

That’s how easy it is. Cluster module lets you fork multiple child processes (using [child_process.fork()](https://nodejs.org/dist/latest-v10.x/docs/api/child_process.html#child_process_child_process_fork_modulepath_args_options)). Making use of the events ‘online’, ‘disconnect’, ‘listening’, ‘message’, ‘error’ and ‘exit’ emitted by worker process, the state of worker process can be easily managed; however, this is not further explained in this article.

## Inter Process Communication (IPC)

The worker processes spawned can communicate with the parent via IPC (Inter Process Communication) channel which allows messages to be passed back and forth between the parent and child. Cluster module makes use of ‘process.send()’ and process.on(‘message’) to communicate between two processes.

```javascript
process.on('message', message => {
  console.log('Message from parent:', message);
process.send('Message from worker');
});
```

One can dedicate specific workers to do specific tasks. Expensive tasks which are either I/O intensive or CPU intensive can have dedicated worker process/processes. The master can delegate the tasks to dedicated workers. Initialize worker with custom id.

```javascript
for (let i = 0; i < numCPUs; ++i) {
  const customId = i + 100;
  const worker = cluster.fork({ workerId: customId });
}
```

Worker process ‘env’ is not exposed to master. One approach is to maintain a map of cluster with required info.

```javascript
const clusterMap = {};

for (let i = 0; i < numCPUs; ++i) {
  const customId = i + 100;
  const worker = cluster.fork({ customId: id });
  clusterMap[worker.id] = customId;
}
```

Master can delegate tasks to specific worker something like this

```javascript
if (clusterMap[worker.id] === customId) {
  worker.send({msg: 'Message from master', task: taskArg});
}
```

Similarly, message communicated between master and worker can be customized to delegate tasks. e.g.

```javascript
process.send({msgType: 'EMAIL', message: emailContent});

worker.on('message', function (message) {
  // each worker is listning to 'message' event
  // and then perform relevant tasks.
  switch (message.msgType) {
    case 'EMAIL':
      // Code to send email
      break;
    default:
      // default action
      break;
  }
});
```

Complete code snippet

```javascript
'use strict';

const cluster = require('cluster');
const numCPUs = require('os').cpus().length;

if (cluster.isMaster) {
  console.log('Master process is running with pid:', process.pid);
  const clusterMap = {};
  let count = 0; // Used to avoid infinite loop
  
for (let i = 0; i < numCPUs; ++i) {
    const customId = i + 100;
    const worker = cluster.fork({ workerId: customId });
clusterMap[worker.id] = customId;
worker.send({ msg: 'Hello from Master' });
worker.on('message', msg => {
      console.log('Message from worker:', clusterMap[worker.id], msg);
      
      if (clusterMap[worker.id] === 101 && !count++) {
        // Message from master for worker 101 to do specific task with taskArg
        const taskArg = { params: { name: 'xyz' }, task: 'email' }; // dummy arg
        worker.send(taskArg);
      } else {
        switch (msg.msgType) {
          case 'EMAIL':
            console.log('Action to perform is EMAIL');
            // Code to send email
            break;
          default:
            // default action
            break;
        }
      }
    });
  }
} else {
  console.log(
    'Worker started with pid:',
    process.pid,
    'and id:',
    process.env.workerId
  );
  
process.on('message', msg => {
    console.log('Message from master:', msg);
    process.send({
      msgType: 'EMAIL',
      msg: 'Hello from worker'
    });
  });
}
```

Based on the above understanding of Node.js clustering and IPC, processes can communicate in following interaction styles while acting as client and server.

- One-to-one: Typically used for one to one notification or request/response.
- One-to-many: Typically used for publish/subscribe.

## Summary

Clustering is a must have for any Node.js production app. It helps to scale horizontally according to the number of CPU cores available on the machine. It is also easier to manage as there is no dependency on any external module/service.

Implementing inter process communication is also very easy. IPC using Node.js cluster module is in memory and hence scalability becomes an issue. Performance also takes a hit when there are too many messages. Management of process state has to be done by yourself.

[PM2](http://pm2.keymetrics.io) with use of message broker like [RabbitMQ](https://www.rabbitmq.com) can also be used to manage the processes and inter process communications which provides more flexibility with the only downside of adding external dependency. It also helps in horizontal scaling and provides a flexible architecture.
[‘node-ipc’](http://riaevangelist.github.io/node-ipc/) is a Node.js module for local and remote Inter Process Communication with full support for Linux, Mac and Windows. It also supports all forms of socket communication from low level unix and windows sockets to UDP and secure TLS and TCP sockets.

Read more about [Node.js cluster module](https://nodejs.org/docs/latest/api/cluster.html#cluster_cluster)
