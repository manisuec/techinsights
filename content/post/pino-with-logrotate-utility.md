---
layout: post
title: "Pinojs with Logrotate is the best way for managing logs in your Nodejs application"
date: 2023-02-13 00:00:00 +0530
images: ["https://res.cloudinary.com/dkiurfsjm/image/upload/v1676284194/pino-logrotate_hhr0er.png"]
thumbnail: "https://res.cloudinary.com/dkiurfsjm/image/upload/v1675429691/NodeJS-Dark_fzh3cd.jpg"
tags: ['pino', 'nodejs', 'expressjs']
keywords: 'pino,nodejs,logging,logger,expressjs,logrotate'
categories: ['Nodejs']
url: 'nodejs/pino-with-logrotate-utility'
---

Log files are very useful to troubleshoot, to track usage, improve performance, and monitor the overall health of the application. However, over time, they become large and use up valuable disk space. It not only makes debugging harder but also slows down the application as writing new log lines become expensive operation. This is where log rotation is required.

![](https://res.cloudinary.com/dkiurfsjm/image/upload/v1676284194/pino-logrotate_hhr0er.png)

## Stop using log-rotate npm libraries

Almost all the popular logging framework in Nodejs like winston, bunyan etc have a complimentary log rotation npm libraries winston-daily-rotate-file, bunyan-rotating-file-stream etc. However, in my personal view we should avoid using them and let the Nodejs application concentrate on serving the business logic. These libraries only adds overhead and dependencies in your application.

The less resources is used for logging, faster the application. This is where Pino scores over other logging frameworks. Refer [benchmarks document.](https://github.com/pinojs/pino/blob/master/docs/benchmarks.md)
 
I have explained in detail how to ['Setup logging with Pino'.](https://techinsights.manisuec.com/nodejs/pino-logger-express-http-context/) Pino recommends to use `logrotate` tool for log rotation.

## Logrotate Utility

Logrotate performs the function of archiving a log file and starting a new one, thereby ‘rotating’ it. Logrotate has been designed to ease the administration of systems that generate large numbers of log files in any format. It allows automatic rotation, compression, removal and mailing of log files. Each log file may be handled daily, every week, every month, or when it grows too large (rotation on the basis of a file’s size).

## Installation

To install logrotate, we can use the package manager in different Linux distros.

On Ubuntu, install the package using apt-get:

> $ sudo apt-get update
> 
> $ sudo apt-get install -y logrotate

For RHEL based Linux such as CentOS, use the yum package manager:

> $ sudo yum update
> 
> $ sudo yum install -y logrotate

## Configure logrotation

1. Configuration can be done in the default global configuration file `/etc/logrotate.conf`

2. Create separate configuration files in the directory `/etc/logrotate.d/` for each service/application.

In my view option 2) is a better way to write logrotate configurations, as each configuration is separate from the other.

e.g.

```bash
/home/ec2-user/express-backend/logs/*log {
    hourly
    rotate 10
    size 20M
    copytruncate
    missingok
    notifempty
    compress
    delaycompress
    sharedscripts
}
```

By default, the installation of logrotate creates a crontab file inside `/etc/cron.daily` named logrotate. To verify, watch for the line containing `cron.daily` in either `/etc/crontab` or `/etc/anacrontab`. As it is the case with the other crontab files, it will be executed daily.

> Suggested Read: [Cron Scheduling Task Examples in Linux](https://www.tecmint.com/11-cron-scheduling-task-examples-in-linux/)
> 
> More details: [logrotate - Linux man page](https://linux.die.net/man/8/logrotate)

## Conclusion

Logrotate is one of the best utilities available in the Linux OS. [Pino](https://github.com/pinojs/pino) is a 'very low overhead' and uses minimum resources for logging. It is also the fastest logging framework and increases the performance of applications developed in Nodejs.

A working example for Pino can be found in [techinsights-tutorials/pino-http-context](https://github.com/manisuec/techinsights-tutorials/tree/main/pino-http-context) repository on github.

✨ Thank you for reading and I hope you find it helpful. I sincerely request for your feedback in the comment's section.