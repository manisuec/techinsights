---
layout: post
title: 'Optimize Mongoose Queries for Speed & Efficiency: The Lean Way'
description: Optimize Mongoose find() performance with the lean() method. Learn how it speeds up retrieval and reduces memory usage by returning plain JavaScript objects
images: ['https://res.cloudinary.com/dkiurfsjm/image/upload/v1676885262/mongoose-lean_m3xter.png']
thumbnail: 'https://res.cloudinary.com/dkiurfsjm/image/upload/v1676956718/mongoose_logo_hr3blb.jpg'
date: 2023-02-20 00:00:00 +0530
tags: [mongoosejs, 'mongodb', 'javascript', 'database']
categories: ['Mongodb']
keywords: 'mongoose,mongodb,queries,mongoose queries,lean,faster,virtuals,populate,document,subdocument'
url: 'mongodb/mongoose-queries-faster-lean'
---

[Mongoose](https://mongoosejs.com) is a schema-based solution to model your application data in MongoDB. It includes built-in type casting, validation, query building, business logic hooks and more, out of the box.

![mongoose lean](https://res.cloudinary.com/dkiurfsjm/image/upload/v1676885262/mongoose-lean_m3xter.png)

Mongoose offers various methods to retrieve documents from a collection, such as `find()`, `findOne()`, and `findById()`. Although `find()` is the most commonly used method and returns multiple documents based on a specified condition, it can lack performance when dealing with a large number of documents in a collection. To optimize the performance of `find()` when handling a large number of documents, Mongoose provides the `lean()` method. This method, when used with `find()`, retrieves documents more quickly than `find()` alone and is less memory intensive too. The result documents are plain old JavaScript objects (POJOs), not Mongoose documents. In this post, you'll learn more about the tradeoffs of using `lean()`.

## Using Lean

Mongoose queries typically return an instance of the Mongoose Document class as the default behavior. However, these documents are significantly heavier than vanilla JavaScript objects due to the extensive internal state required for change tracking. By enabling the "lean" option, Mongoose skips instantiating a full Mongoose document and instead provides a POJO (plain old JavaScript object) that is much lighter. Let us understand this with example.

```javascript
# define the schemas as below

const mongoose = require(mongoosejs);

// Create models
// 1. Group Model
const groupSchema = new mongoose.Schema({
  _id: {type: String},
  name: {type: String}
});

groupSchema.virtual('members', {
  ref: 'Person',
  localField: '_id',
  foreignField: 'groupId'
});

const Group = mongoose.model('Group', groupSchema);

// 2. Person Model
const personSchema = new mongoose.Schema({
  _id: {type: String},
  firstName: {
    type: String,
    get: capitalizeFirstLetter
  },
  lastName: {
    type: String,
    get: capitalizeFirstLetter
  },
  groupId: {type: String, ref: 'Group'} 
});

personSchema.virtual('fullName').get(function() {
  return `${this.firstName} ${this.lastName}`;
});

// Convert 'manish' -> 'Manish'
function capitalizeFirstLetter(v) {
  return v.charAt(0).toUpperCase() + v.substring(1);
}

const Person = mongoose.model('Person', personSchema);
```

Let us now insert some records and then retrieve the documents

```javascript
const group = await Group.create({ _id: 'G100', name: 'Group 100' });
await Person.create([
{ _id: 'P100', firstName: 'manish', lastName: 'prasad', groupId: 'G100' },
{ _id: 'P101', firstName: 'ravi', lastName: 'das', groupId: 'G100' },
]);

const normalDoc = await Person.findOne();
const leanDoc = await Person.findOne().lean();
```

After executing a query, Mongoose internally converts the query results from POJOs to Mongoose documents. However, by enabling the "lean" option, Mongoose bypasses this step and directly returns the query results as POJOs.

```javascript
console.log(normalDoc instanceof mongoose.Document); // true
console.log(normalDoc.constructor.name); // 'model'

console.log(leanDoc instanceof mongoose.Document); // false
console.log(leanDoc.constructor.name); // 'Object'

console.log(v8.serialize(normalDoc).length); // approximately 798
console.log(v8.serialize(leanDoc).length); // 75, about 10x smaller!
```

The downside of enabling lean is that lean docs don't have:

- Change tracking
- Casting and validation
- Getters and setters
- Virtuals
- `save()`

```javascript
console.log(normalDoc.fullName); // 'Manish Prasad'
console.log(normalDoc.firstName); // 'Manish', because of `capitalizeFirstLetter()`
console.log(normalDoc.lastName); // 'Prasad', because of `capitalizeFirstLetter()`

console.log(leanDoc.fullName); // undefined, virtual doesn't run
console.log(leanDoc.firstName); // 'manish', custom getter doesn't run
console.log(leanDoc.lastName); // 'prasad', custom getter doesn't run
```

## Lean and Populate

You can use `populate()` in combination with `lean()` to make the populated documents and top-level documents lean. In the example below, both the "Group" documents at the top level and the "Person" documents that are populated will be lean.

```javascript
// Execute a lean query
const groupRecord = await Group.findOne().lean().populate({
  path: 'members',
  options: { sort: { name: 1 } }
});
console.log(groupRecord.members[0].firstName); // 'manish'
console.log(groupRecord.members[1].firstName); // 'ravi'

// Both the `group` and the populated `members` are lean.
console.log(groupRecord instanceof mongoose.Document); // false
console.log(groupRecord.members[0] instanceof mongoose.Document); // false
console.log(groupRecord.members[1] instanceof mongoose.Document); // false

```

## When to Use Lean

When sending query results and are not modified and don't use custom getters; it's recommended to use `lean()`. However, if the query results are modified or rely on features like getters or transforms, it's better to avoid using lean().

e.g. returning a list of documents for `GET` request

## Useful Plugins

Using `lean()` bypasses all Mongoose features, including virtuals, getters/setters,
and defaults. To use these features with `lean()`, you need to use the corresponding plugin:

- [mongoose-lean-virtuals](https://plugins.mongoosejs.io/plugins/lean-virtuals)
- [mongoose-lean-getters](https://plugins.mongoosejs.io/plugins/lean-getters)
- [mongoose-lean-defaults](https://www.npmjs.com/package/mongoose-lean-defaults)

## Conclusion

Lean is great for high-performance, read-only cases, especially when combined with cursors. With the help of plugins, `lean()` can be used with virtuals, getters/setters, or defaults. The sample code is checked into [mongoose-lean](https://github.com/manisuec/techinsights-tutorials/tree/main/mongoose-lean) on github.

✨ Thank you for reading and I hope you find it helpful. I sincerely request for your feedback in the comment's section.

