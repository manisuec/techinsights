---
layout: post
title: 'MongoDB Installation & Configuration on AWS EC2 Instance'
date: 2021-07-27 00:00:00 +0530
images: ['/img/posts/mongodb.png']
tags: ['mongodb']
url: 'mongodb/mongodb-installation-aws-ec2'
aliases:
    - /post/2021-07-27-mongodb-installation/
---

I had to search a lot before figuring out a proper way to install and configure MongoDB in AWS EC2 instance. I am sharing my learnings and hope it is useful for others.

## Install MongoDB Community Edition

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

`$ which mongo     # it should print /usr/bin/mongo`


## Configure EC2 Instance

In my opinion, it will be good to add 3 additional EBS volumes for MongoDB

1. **Data volume:** Size: x GB (according to your size estimation)
2. **Journal volumne:** Recommended size is 1/5th of data volume
3. **Log volume:** Recommended size is half of journal volume

Size of EBS volumes can be changed later also.

Mount each volume as below:
Read detailed steps for [Make an Amazon EBS volume available for use.](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ebs-using-volumes.html)

```
# create below directories
$ sudo mkdir /data
$ sudo mkdir /journal
$ sudo mkdir /log

# mount volumes appropriately
$ sudo mount -t xfs /dev/sdf /data
$ sudo mount -t xfs /dev/sdg /journal
$ sudo mount -t xfs /dev/sdh /log

# create a link for journal
$ sudo ln -s /journal /data/journal

# set appropriate ownership
$ sudo chown mongod:mongod /data
$ sudo chown mongod:mongod /log/
$ sudo chown mongod:mongod /journal/
```

Add below content using command `$ sudo vi /etc/fstab`

```
/dev/sdf /data    xfs defaults,auto,noatime,noexec 0 0
/dev/sdg /journal xfs defaults,auto,noatime,noexec 0 0
/dev/sdh /log     xfs defaults,auto,noatime,noexec 0 0
```

I would also recommend to add swap space of atleast 4GB. Run below commands to create the swap space.

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

Create a file `/etc/logrotate.d/mongodb` and add below contents:

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

Change value of port other than the default `27017` and also update the security group settings accordingly.

```
# network interfaces
net:
  port: 31012
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

Connect to mongodb by using command `$ mongo`

Create a root user: 
```
use admin

db.createUser({ user: "adminUser", pwd: "adminPass", roles: ["root"] })
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

I have tried to document all the steps based on my learnings during standalone installation. I will write another blog for MongoDB Installation with Replication setup. I hope you enjoyed reading this and find it helpful. I sincerely request for your feedback. You can follow me on twitter [@lifeClicks25](https://twitter.com/lifeClicks25).
