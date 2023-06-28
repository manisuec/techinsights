---
layout: post
title: 'MongoDB Installation & Configuration on AWS EC2 Instance'
date: 2021-07-27 00:00:00 +0530
images: ['/img/posts/mongodb.png']
thumbnail: 'https://res.cloudinary.com/dkiurfsjm/image/upload/v1675429691/MongoDB_jeatlj.jpg'
categories: ['Mongodb']
tags: ['mongodb']
keywords: 'mongodb,aws,ec2,installation,standalone,production,configuration'
url: 'mongodb/mongodb-installation-aws-ec2'
aliases:
    - /post/2021-07-27-mongodb-installation/
---

I had to search a lot before figuring out a proper way to install and configure a production ready standalone MongoDB installation in AWS EC2 instance. I am sharing my learnings and hope it is useful for others.

## Install MongoDB Community Edition

Prior to installation; please refer [deploy ec2 in a private subnet & securely communicate with internet](https://techinsights.manisuec.com/mongodb/db-ec2-private-subnet-vpc/) to properply setup your ec2 instance.

Create a `/etc/yum.repos.d/mongodb-org-4.4.repo` file so that you can install MongoDB directly using yum:

```
[mongodb-org-4.4]
name=MongoDB Repository
baseurl=https://repo.mongodb.org/yum/amazon/2/mongodb-org/4.4/x86_64/
gpgcheck=1
enabled=1
gpgkey=https://www.mongodb.org/static/pgp/server-4.4.asc
```

To install the latest stable version of MongoDB, issue the following command:

`$ sudo yum install -y mongodb-org`

Issue below command to verify whether MongoDB is installed properly

```
$ which mongo     # it should print 

/usr/bin/mongo
```

## Configure EC2 Instance

In my opinion, it will be good to add 2 additional EBS volumes apart from default root volume for MongoDB

1. **Data volume:** Size: X GB (according to your size estimation)
2. **Log volume:** Recommended size is 1/10th of data volume

**Note:-** A separate volume for `journal` can also be attached by creating a symbolic link to point the journal directory to the attached volume as

> MongoDB creates a subdirectory named journal under the dbPath directory

Size of EBS volumes can be changed later also.

Read detailed steps for [Make an Amazon EBS volume available for use.](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ebs-using-volumes.html). 
Mount each volume as below:

```
# create below directories
$ sudo mkdir /data
$ sudo mkdir /log

# mount volumes appropriately
$ sudo mount -t xfs /dev/sdf /data
$ sudo mount -t xfs /dev/sdh /log

# set appropriate ownership
$ sudo chown mongod:mongod /data
$ sudo chown mongod:mongod /log/
```

Add below content using command `$ sudo vi /etc/fstab`

```
/dev/sdf /data    xfs defaults,auto,noatime,noexec 0 0
/dev/sdh /log     xfs defaults,auto,noatime,noexec 0 0
```

I would also recommend to add swap space of atleast **4GB**. Run below commands to create the swap space.

```
$ sudo dd if=/dev/xvda1 of=/swapfile bs=4096 count=488400

$ sudo chmod 600 /swapfile

$ sudo mkswap /swapfile

$ sudo swapon /swapfile

$ sudo swapon -s

$ sudo vi /etc/fstab
```

Add below contents: 

```
/swapfile swap swap defaults 0 0
```

Run command `$ sudo swapon --show` to verify swap is active.

```
# Output will be similar as below

NAME      TYPE  SIZE   USED PRIO
/swapfile file 1024M 507.4M   -1
```

## Configure `ulimit` settings

Update the `limits.conf` file with recommended values:

`$ sudo vi /etc/security/limits.conf`

Add below content:

```
* soft nofile 64000
* hard nofile 64000
* soft nproc 64000
* hard nproc 64000
```

This step might require to reboot the instance.

Verify the values by issuing the command `$ ulimit -a`

## Disable Transparent Huge Pages (THP)

I would recommend to follow steps mentioned at [MongoDB](https://docs.mongodb.com/v4.4/tutorial/transparent-huge-pages/) official documentation.

## Configure Log Rotation

[Linux logrotate utility](https://linux.die.net/man/8/logrotate) allows automatic rotation, compression and removal of log files. Create a file `/etc/logrotate.d/mongodb` and add below contents:

```
/log/*log {
    hourly
    rotate 10
    size 50M
    copytruncate
    missingok
    notifempty
    compress
    delaycompress
    sharedscripts
}
```

Then create a cron job to run hourly (run as per your need)

`$ crontab -e`

Add below content:

`0 * * * * sudo /usr/sbin/logrotate /etc/logrotate.d/mongodb`

## MongoDB Configuration

Update MongoDB configuration:

`$ sudo vi /etc/mongod.conf`

Change value of port other than the default `27017` and also update the security group settings in aws accordingly.

```
# network interfaces
net:
  port: 31012
```

Update path of `pid` file.

```
# how the process runs
processManagement:
  fork: true  # fork and run in background
  pidFilePath: /home/mongod/mongod.pid  # location of pidfile
```

Also, update data and log paths:

```
# where to write logging data.
systemLog:
  destination: file
  logAppend: true
  path: /log/mongod.log

# Where and how to store data.
storage:
  dbPath: /data
  journal:
    enabled: true
```

## MongoDB User

Start mongodb user by using command `$ mongod --config /etc/mongod.conf`

Connect to mongodb by using command `$ mongo --port <configured port value>`

Create a root user:

```
use admin

db.createUser({ user: "adminUser", pwd: "<strong password>", roles: ["root"] })
```

I would strongly recommend to go through the section [Built-In Roles](https://docs.mongodb.com/v4.4/reference/built-in-roles/)

Update `/etc/mongod.conf` file with

```
security:
  authorization: enabled
```

Restart the mongod process and try to connect using command:

`$ mongo --port <port value> -u adminUser --authenticationDatabase admin`

## Conclusion

I have tried to document all the steps based on my learnings during a standalone installation. I will write another blog for MongoDB Installation with Replication setup. I hope you enjoyed reading this and find it helpful. I sincerely request for your feedback. You can follow me on twitter [@lifeClicks25](https://twitter.com/lifeClicks25).

