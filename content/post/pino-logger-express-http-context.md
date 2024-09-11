---
layout: post
title: "Setup logging with Pino and express-http-context in Expressjs"
date: 2023-02-11 00:00:00 +0530
images: ["https://res.cloudinary.com/dkiurfsjm/image/upload/v1676006659/pino-banner_fktunb.png"]
thumbnail: "https://res.cloudinary.com/dkiurfsjm/image/upload/v1676698473/nodejs_dark_cjoudy.png"
tags: ['pino', 'nodejs', 'expressjs']
keywords: 'pino,nodejs,logging,logger,expressjs,http context,server,logging framework'
categories: ['Nodejs']
url: 'nodejs/pino-logger-express-http-context'
---

Logging is an essential tool for debugging and understanding the behavior of your application. It allows developers to capture information about the state and flow of an application, which can be used to identify and fix bugs, improve performance, and monitor the overall health of the application.

![pino logger](https://res.cloudinary.com/dkiurfsjm/image/upload/v1676006659/pino-banner_fktunb.png)

When setting up logging in application server based on Nodejs, it's important to choose an appropriate logging framework, such as pino, winston, or others, that supports the logging level and configuration that you need. <a name="ref_1"></a>Additionally, it's important to log information at the right level of detail and to write logs in a way that makes them easy to read and understand.

## Why pino?

[Pino](https://github.com/pinojs/pino) is a 'very low overhead' and uses minimum resources for logging. It is also the fastest logging framework and increases the performance of applications developed in Nodejs. Pino's idea is that faster the logger the less CPU it uses thereby increasing throughput and reducing the response latency. Compare the results in their [benchmarks document](https://github.com/pinojs/pino/blob/master/docs/benchmarks.md).

## Using pino in expressjs application

As mentioned in the [introductory section](#ref_1); it's important to log information at the right level of detail. In a typical expressjs application, storing request-scoped data such as JWT token claims, user state etc. becomes a necessity. [express-http-context](https://github.com/skonves/express-http-context) is widely used library for storing and fetching request-scoped context anywhere. 

So far in my experience, having timestamp in human readable format along with unique request id for requests in the log line makes debugging a lot easier. Below is the code-snippet how I set up pino logger.
> logger.js

```javascript
const logger = require("pino");
const path = require("path");
const dateFns = require("date-fns");
const httpContext = require("express-http-context");

const logFilePath = path.resolve(__dirname, ".");

const defaultOpts = {
  singleLine  : true,
  mkdir       : true,
  hideObject  : false,
  ignore      : "hostname",
  prettyPrint : true,
};

const targets = [
  {
    level   : "info",
    target  : "pino/file",
    options : {
      ...defaultOpts,
      destination : path.resolve(logFilePath, "info.log"),
    },
  },
  {
    level   : "debug",
    target  : "pino/file",
    options : {
      ...defaultOpts,
      destination : path.resolve(logFilePath, "debug.log"),
    },
  },
  {
    level   : "error",
    target  : "pino/file",
    options : {
      ...defaultOpts,
      destination : path.resolve(logFilePath, "error.log"),
    },
  },
];

if (process.env.ENABLE_CONSOLE_LOG === "true") {
  targets.push({
    target  : "pino-pretty",
    options : {
      colorize     : true,
      timestampKey : "time",
      singleLine   : true,
      hideObject   : false,
      ignore       : "hostname",
    },
  });
}

const transport = logger.transport({
  targets,
});

module.exports = logger(
  {
    level     : process.env.PINO_LOG_LEVEL || "info",
    timestamp : () =>
      `,"time":"${dateFns.format(Date.now(), "dd-MMM-yyyy HH:mm:ss sss")}"`,
    mixin() {
      if (httpContext.get("req_id")) {
        return { req_id: httpContext.get("req_id") };
      } else {
        return {};
      }
    },
  },
  transport
);
```

Initialize context in the `app.js`

```javascript
const express = require("express");
const httpContext = require("express-http-context");
const cuid = require("cuid");

const logger = require("./log");

const app = express();

app.use(httpContext.middleware);
app.use((req, res, next) => {
  httpContext.set("req_id", cuid());
  httpContext.set("wid", process.pid);

  next();
}); // Set Request id

app.disable("x-powered-by");

app.get('/', (req, res) => {
  logger.info('request for hello');
  res.send('Hello World!')
})

app.listen(3000, () => {
  logger.info(`expressjs server listening on port 3000`);
})
```

Start your server by running command `node app.js` and hit the url `http://localhost:3000` in your browser 2-3 times

You will find `info.log` created with logs written as

```
{"level":30,"time":"11-Feb-2023 12:28:31 031","msg":"expressjs server listening on port 3000"}
{"level":30,"time":"11-Feb-2023 12:28:32 032","req_id":"cldzlwbxz0000cv4wft556vdh","msg":"request for hello"}
{"level":30,"time":"11-Feb-2023 12:28:33 033","req_id":"cldzlwc980001cv4w8d83902e","msg":"request for hello"}
{"level":30,"time":"11-Feb-2023 12:28:34 034","req_id":"cldzlwd1a0002cv4wcw05bybs","msg":"request for hello"}
{"level":30,"time":"11-Feb-2023 12:28:35 035","req_id":"cldzlwdmh0003cv4w9rb5d089","msg":"request for hello"}
```

> Suggested Read: [Pino API reference](https://getpino.io/#/docs/api)

## Conclusion

A working example can be found in [techinsights-tutorials/pino-http-context](https://github.com/manisuec/techinsights-tutorials/tree/main/pino-http-context) repository on github.

Depending upon the need of your application; you can add other details like user id etc. in the logs. The primary aim of this tutorial was to help start with pino logger.

✨ Thank you for reading and I hope you find it helpful. I sincerely request for your feedback in the comment's section.

