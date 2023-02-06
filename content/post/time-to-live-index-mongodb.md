---
layout: post
title: "Understanding TTL Indexes in MongoDB in depth"
date: 2023-02-06 00:00:00 +0530
thumbnail: 'https://res.cloudinary.com/dkiurfsjm/image/upload/v1675429691/MongoDB_jeatlj.jpg'
tags: ['mongodb', 'ttl index', 'time to live']
keywords: 'mongodb,ttl index,time to live,index,expire data'
categories: ['Mongodb']
url: 'mongodb/time-to-live-ttl-index-mongodb'
---

Time to Live (TTL) indexes are special single-field indexes in MongoDB that help delete documents from a collection after a specific amount of time or at a given clock time. Data expiration is useful for certain types of information like machine generated event data, shopping carts, logs, and session information etc that only need to persist in a database for a finite amount of time.

MongoDB offers an easy way to automatically expire documents using a TTL index. This is much more efficient than writing your own code as you don't have to worry about manually deleting expired documents. You can bypass the hassle of manually filtering documents based on their expiry and calculating the absolute expiry time. All you need to provide is the number of seconds a document should be valid for and let Mongodb take care of the rest.

## Create a TTL Index

To create a TTL index, use the `createIndex()` method on a field whose value is either a date or an array that contains date values, and specify the `expireAfterSeconds` option with the desired TTL value in seconds.

For example, to create a TTL index on the lastModifiedDate field of the eventlog collection, with a TTL value of 3600 seconds, use the following operation in the mongo shell:

```
db.eventlog.createIndex( { "lastModifiedDate": 1 }, { expireAfterSeconds: 3600 } )
```
### Note:-
- The indexed field must be [BSON date type](https://www.mongodb.com/docs/manual/reference/bson-types/?ref=hackernoon.com#document-bson-type-date) or an array of BSON dates
- If the field is an array, and there are multiple date values in the index, MongoDB uses lowest (i.e. earliest) date value in the array to calculate the expiration threshold.
- If the indexed field in a document is not a date or an array that holds a date value(s), the document will not expire.
- If a document does not contain the indexed field, the document will not expire.

## Delete Operations

A background thread in `mongod` reads the values in the index and removes expired documents from the collection.

The TTL index does not guarantee that expired data will be deleted immediately upon expiration. There may be a delay between the time that a document expires and the time that MongoDB removes the document from the database.

The background task, `the TTLMonitor thread`, that removes expired documents runs every 60 seconds. Because TTL thread runs in every minute by default, it can affect the performance slightly. To change this interval, use admin command with the desired interval:

```
db.adminCommand({setParameter:1, ttlMonitorSleepSecs: 3600}); // 1 hour
{ "was" : 60, "ok" : 1 }
```

Because the duration of the removal operation depends on the workload of your mongod instance, expired data may exist for some time beyond the `TTLMonitor` period between runs of the background task.

You can disable TTLMonitor thread at mongod startup or by an admin command from mongo shell:

During mongod startup:

```$ mongod --setParameter ttlMonitorEnabled=false```

On replica set members, the TTL background thread only deletes documents when a member is in state primary. Secondary members replicate deletion operations from the primary.

## Conditional Delete

A collection can be partially indexed using a specified filter expression, `partialFilterExpression`. TTL index can also be used with [partial indexes.](https://www.mongodb.com/docs/manual/core/index-partial/)

e.g.

Delete documents that are created 1 day ago if count is more than 5

```
db.eventlog.createIndex(
  { created_at: 1 },
  { expireAfterSeconds: 86400, partialFilterExpression: { count: { $gt: 5 } } }
);
```

## Expire Documents at a Specific Clock Time or Dynamic TTL

In some cases, we might need to have different documents to have different expire times instead of the fixed time period. This will help us achieving document level TTL indexing. This feature gives a fine-grained control over the documents.

This is very easy to implement. Create a TTL index with `expireAfterSeconds=0` and set the indexed field value of each document to a future date. Then, while inserting document, add its expected expiring time with the current time and save it to the indexed field.


## Restrictions

- TTL indexes are a single-field indexes. Compound indexes do not support TTL and ignore the expireAfterSeconds option.

- The _id field does not support TTL indexes.

Read more at [Restrictions.](https://www.mongodb.com/docs/manual/core/index-ttl/#restrictions)

## Conclusion

TTL indexes is underused feature of Mongodb. In my view, it is a very helpful feature for building data applications. This removes the need for running a cronjob that deletes the expired data. Also, ability to have different documents to have different expire times instead of the fixed time period gives a fine-grained control over the documents.

✨ Thank you for reading and I hope you find it helpful. I sincerely request for your feedback in the comment's section.