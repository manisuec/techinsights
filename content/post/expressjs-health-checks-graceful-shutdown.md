---
layout: post
title: 'Health Checks and Graceful Shutdown of Expressjs App using Lightship'
date: 2023-06-07 00:00:00 +0530
images: ['https://res.cloudinary.com/dkiurfsjm/image/upload/v1686135208/light_ship_ugp2qc.jpg']
thumbnail: 'https://res.cloudinary.com/dkiurfsjm/image/upload/v1676698473/nodejs_dark_cjoudy.png'
tags: ['nodejs', 'javascript', 'expressjs']
keywords: 'nodejs,expressjs,health check,shutdown'
categories: ['Nodejs']
url: 'nodejs/expressjs-health-check-lightship'
---

Expressjs is a fast, unopinionated, minimalist web framework for Node.js. Few functionalities like graceful shutdown, health checks etc are implicitly mandatory features for any production application. When you deploy a new version of your application, you must replace the previous version. Previous version app should stop accepting new requests, finish all the ongoing requests, clean up the resources it used, including database connections and file locks then exit. Similarly, health checks are needed to determine if an application instance is healthy and can accept requests.

## Lightship

[Lightship](https://github.com/gajus/lightship) is an open-source project that adds health, readiness and liveness checks to your application. Lightship is a standalone HTTP-service that runs as a separate HTTP service; this allows having health-readiness-liveness HTTP endpoints without exposing them on the public interface.

![](https://res.cloudinary.com/dkiurfsjm/image/upload/v1686135208/light_ship_ugp2qc.jpg)

## Installation

```npm install lightship --save```

Then, add below code into your expressjs server.

```
const express = require('express');
const { createLightship } = require('lightship');

const configuration: ConfigurationInput = {};
const lightship: Lightship = await createLightship(configuration);

const app = express();

// other app related configuration code

app.listen(3000, () => lightship.signalReady());

lightship.registerShutdownHandler(() => {
  server.close();
});

lightship.signalReady();

(async () => {
  // `whenFirstReady` returns a promise that is resolved the first time that
  // the service goes from `SERVER_IS_NOT_READY` to `SERVER_IS_READY` state.
  await lightship.whenFirstReady();
})();
```

## Local-mode
If Lightship detects that it is running in a non-Kubernetes environment (e.g. your local machine) then...

- It starts the HTTP service on any available HTTP port. This is done to avoid port collision when multiple services using Lightship are being developed on the same machine.

- `shutdownDelay` defaults to `0`, i.e. immediately proceed to execute the shutdown routine.

Detection of the local-mode can be overridden by setting `{detectKubernetes: false}`

## Usage

Once running, we get 3 endpoints:

- [`/health`](#health)
- [`/live`](#live)
- [`/ready`](#ready)

### **Api endpoint: /health** {#health}

The `/health` endpoint describes the current state of a Nodejs/expressjs service.

The endpoint responds:

- `200` status code, message "SERVER\_IS_READY" when server is accepting new connections.
- `500` status code, message "SERVER\_IS\_NOT_READY" when server is initialising.
- `500` status code, message "SERVER\_IS\_SHUTTING_DOWN" when server is shutting down.


### **Api endpoint: /live** {#live}

The `/live` endpoint responds:

- `200` status code, message "SERVER\_IS\_NOT\_SHUTTING_DOWN".
- `500` status code, message "SERVER\_IS\_SHUTTING_DOWN".

Used to configure liveness probe.

### **Api endpoint: /ready** {#ready}

The `/ready` endpoint responds:

- `200` status code, message "SERVER\_IS_READY".
- `500` status code, message "SERVER\_IS\_NOT_READY".

Used to configure readiness probe.

## Timeouts

Lightship has two timeout configurations: `gracefulShutdownTimeout` and `shutdownHandlerTimeout`.

`gracefulShutdownTimeout` (default: 30 seconds) is a number of milliseconds Lightship waits for Node.js process to exit gracefully after it receives a shutdown signal (either via `process` or by calling `lightship.shutdown()`) before killing the process using `process.exit(1)`. 

This timeout should be sufficiently big to allow Node.js process to complete tasks (if any) that are active at the time that the shutdown signal is received (e.g. complete serving responses to all HTTP requests) 

`shutdownHandlerTimeout` (default: 5 seconds) is a number of milliseconds. Lightship waits for shutdown handlers (see `registerShutdownHandler`) to complete before killing the process using `process.exit(1)`.

Read more [Usage with examples](https://github.com/gajus/lightship#usage-examples).

## Best Practices

### Add a delay before stop handling incoming requests
It is important that you do not cease to handle new incoming requests immediatelly after receiving the shutdown signal. This is because there is a high probability of the SIGTERM signal being sent well before the iptables rules are updated on all nodes. The result is that the pod may still receive client requests after it has received the termination signal. If the app stops accepting connections immediately, it causes clients to receive "connection refused" types of errors.

Properly shutting down an application includes these steps:

Wait for a few seconds, then stop accepting new connections,
Close all keep-alive connections that aren't in the middle of a request,
Wait for all active requests to finish, and then
Shut down completely.
See [Handling Client Requests Properly with Kubernetes](https://web.archive.org/web/20200807161820/https://freecontent.manning.com/handling-client-requests-properly-with-kubernetes/) for more information.


## Conclusion


Lightship is a valuable tool for incorporating health checks into our application to ensure its smooth operation.

By utilizing Lightship, we can effortlessly send signals to indicate the running status of our application. The endpoints offered by Lightship enable us to conveniently monitor the health and performance of both our app and server.

In non-Kubernetes environments, Lightship operates as an independent service, enhancing its versatility and compatibility.

✨ Thank you for reading and I hope you find it helpful. I sincerely request for your feedback in the comment's section.

