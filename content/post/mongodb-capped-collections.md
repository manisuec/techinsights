---
layout: post
title: "Mongodb Capped Collections - An Overview"
thumbnail: 'https://res.cloudinary.com/dkiurfsjm/image/upload/v1675429691/MongoDB_jeatlj.jpg'
date: 2023-05-29 00:00:00 +0530
images: ['https://res.cloudinary.com/dkiurfsjm/image/upload/v1685355100/ring_buffer_fjkx4t.jpg']
tags: ['mongodb','capped','database','schema']
keywords: 'mongodb,capped collections,collections,database,schema,no sql'
categories: ['Mongodb']
url: 'mongodb/mongodb-capped-collections'
---

Capped collections provide fixed-size collections that enable high-throughput operations for inserting and retrieving documents based on their insertion order. They function similarly to circular buffers, where once a collection reaches its allocated space, new documents overwrite the oldest ones in the collection.

![](https://res.cloudinary.com/dkiurfsjm/image/upload/v1685355100/ring_buffer_fjkx4t.jpg)

> **TIP:**
> 
> Consider MongoDB's TTL (Time To Live) indexes as an alternative to capped collections. TTL indexes allow you to expire and remove data from regular collections based on a date-typed field and a TTL value. Read more at [Understanding TTL Indexes in MongoDB comprehensively](https://techinsights.manisuec.com/mongodb/time-to-live-ttl-index-mongodb/)

> **Note:** TTL indexes are not compatible with capped collections.

##  Use cases for capped collections

Capped collections are useful for niche use cases where:

- You want to limit the amount of data retained in a FIFO (First In, First Out) order
- You want to guarantee data is stored in insertion order
- You do not have updates that cause documents to grow in size
- You do not need to delete documents manually
- You may want to use a tailable cursor to trigger some activity based on inserts into a collection

MongoDB relies on capped collections for a number of internal use cases such as:

The `local.startup_log`, a 10MB capped collection which captures some diagnostic/environment information about the local mongod instance on start up.

The `replication oplog` (operation log), which each replica set member uses to record an idempotent list of all write operations applied locally. The default oplog size is based on % of free disk space, but is typically on the order of up to 50 GB.

The `sharded cluster config.changelog`, which is a 10MB capped collection of recent sharded cluster balancing activity.

## Creating a Capped Collection
Capped collections must be explicitly created using the `db.createCollection()` method, which is a helper for the create command in the mongosh shell. When creating a capped collection, you need to specify the maximum size of the collection in bytes, which MongoDB will pre-allocate. The collection's size includes a small amount of space for internal overhead.


```javascript
db.createCollection("log", { capped: true, size: 100000 });
```
If the size field is less than or equal to 4096, the collection will have a cap of 4096 bytes. Otherwise, MongoDB will increase the provided size to make it an integer multiple of 256.

You can also specify a maximum number of documents for the collection using the max field.

```javascript
db.createCollection("log", { capped: true, size : 5242880, max : 5000 } });
```

> **IMPORTANT:** 
> 
> The size argument is always required, even when specifying the maximum number

## Querying a Capped Collection

When performing a `find()` operation on a capped collection without specifying any ordering, MongoDB guarantees that the results will be ordered in the same sequence as the documents were inserted.

To retrieve documents in reverse insertion order, issue a `find()` command followed by the `sort()` method with the `$natural` parameter set to -1, as demonstrated in the following example:

```
db.cappedCollection.find().sort({ $natural: -1 })
```

Checking if a Collection is Capped:

To determine if a collection is capped, utilize the `isCapped()` method as shown below:

```
db.collection.isCapped()
```

Converting a Collection to Capped:

You can convert a non-capped collection to a capped collection using the convertToCapped command:

```
db.runCommand({ "convertToCapped": "mycoll", size: 100000 })
```

The size parameter specifies the size of the capped collection in bytes.

During this operation, a database exclusive lock is held, and other operations that lock the same database will be blocked until the conversion is complete.

## Behavior of Capped Collections:

### Insertion Order:

Capped collections guarantee the preservation of the insertion order, eliminating the need for an index to retrieve documents in the order of insertion. This feature allows capped collections to support high insertion throughput without the overhead of indexing.

### Automatic Removal of Oldest Documents:

Capped collections automatically remove the oldest documents to make room for new ones, without requiring additional scripts or explicit removal operations.

## Additional Details:

- **_id Index:**
Capped collections have an _id field and an index on the _id field by default.

- **Reads:**
Starting from MongoDB 5.0, read concern "snapshot" cannot be used when reading from a capped collection.

- **Updates:**
If you plan to update documents in a capped collection, create an index to avoid collection scans during update operations.

- **Sharding:**
Capped collections cannot be sharded.

- **Query Efficiency:**
To efficiently retrieve the most recently inserted elements from a capped collection, utilize natural ordering, similar to using the tail command on a log file.

- **Aggregation `$out`:**
The `$out` stage in the aggregation pipeline cannot write results to a capped collection.

- **Transactions:**
Starting from MongoDB 4.2, capped collections cannot be written to within transactions.

- **Stable API:**
Capped collections are not supported in Stable API V1.

## Conclusion

I hope this article helps you understand about one of the underutilized feature of Mongodb. Capped collections work in a way similar to circular buffers: once a collection fills its allocated space, it makes room for new documents by overwriting the oldest documents in the collection.

✨ Thank you for reading and I hope you find it helpful. I sincerely request for your feedback in the comment's section.

