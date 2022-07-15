---
layout: post
title: 'Understand Sub-documents & Nested documents in Mongoose'
date: 2021-11-24 00:00:00 +0530
images: ['https://res.cloudinary.com/dkiurfsjm/image/upload/e_improve/v1637747985/mongoose_djuhic.png']
tags: ['nodejs', 'mongoose', 'subdocument', 'nested', 'javascript']
---

[Mongoose](https://mongoosejs.com) is the most widely schema-based solution to model your application data in MongoDB. It includes built-in type casting, validation, query building, business logic hooks and more, out of the box.

A record in MongoDB is a document, which is a data structure composed of field and value pairs. MongoDB documents are similar to JSON objects. The values of fields may include other documents, arrays, and arrays of documents. Subdocuments are documents embedded in other documents. In Mongoose, this means you can nest schemas in other schemas. However; nested paths are subtly different from subdocuments.


## Sub-documents and Nested documents

Let us take a describe a schema of a person as below:

```javascript
const mongoose = require('mongoose');

/**
 * @class Person
 */
const personSchema = new mongoose.Schema({
  name : {
    type     : String,
    trim     : true,
    required : true,
  },
  
  // profile as sub document
  profile : new mongoose.Schema({
    city : String,
    age  : {
      type    : Number,
      default : 0
    },
  }),
  
  // others as nested document
  others : {
    amount : {
      type    : Number,
      default : 0
    },
    email : {
      type : String,
      trim : true,
    },
  },

  // array of names as nested document
  friends : [
    { name: String }
  ],

  // array of hobbies as sub document
  hobbies : [
    new mongoose.Schema({ name: 'string' })
  ]
});

module.exports = mongoose.model('Person', personSchema);

```

In the above model:

- `profile` field is defined as a sub-document schema

- `others` field is defined as a nested document

- `friends` is an array of names defined as nested document

- `hobbies` is an array of hobbies defined as sub-document 

Let us now try to understand the subtle difference between sub-document and nested document.

Subdocument paths are `undefined` by default, and Mongoose does not apply subdocument defaults unless you set the subdocument path to a non-nullish value.

```javascript
const Model = mongoose.model('Person');
const doc = new Model();

console.log(doc.profile); // undefined
console.log(doc.others);  // { amount: 0 }

doc.others.email = 'a@a.com'; // doesn't throw error
// doc.profile.age = 35; // throw error

doc.profile = {};
console.log(doc.profile); // { age: 0, _id: 619e0c8be7d5e864f890e7ae }
doc.profile.age = 35; // doesn't throw error now
```

Now let us look at below code and understand the difference in arrays as well

```javascript
const Model = mongoose.model('Person');
const doc = new Model();
const rec = {
  name: 'Test User',
  profile: {
    city: 'Bengaluru',
    age: 25
  },
  others: {
    amount: 100,
    email: 'test@test.com'
  },
  friends: [{name: 'ABC'}, {name: 'XYZ'}],
  hobbies: [{name: 'movies'}, {name: 'travel'}]
}

const newDoc = new Model(rec);

console.log(newDoc.profile);
console.log(newDoc.others);
console.log(newDoc.friends);
console.log(newDoc.hobbies);
```

Output:


```
// profile
{ age: 25, _id: 619e4e79f828ad6a567d918f, city: 'Bengaluru' }
```

Mongoose adds an `_id` field to subdocuments by default. It can be disabled by setting the `_id` option to false in sub-document definition

```
profile : new mongoose.Schema({
    city : String,
    age  : {
      type    : Number,
      default : 0
    },
    _id : false
  }),
```

No `_id` field in the single nested document

```
// others
{ amount: 100, email: 'test@test.com' }
```
A field e.g. `friends` with an array of objects, Mongoose automatically convert the object to a schema and hence `_id` field is added by default. It can be disabled as stated above.

```
// friends
[
  { _id: 619e4e79f828ad6a567d9190, name: 'ABC' },
  { _id: 619e4e79f828ad6a567d9191, name: 'XYZ' }
]
```
```
// hobbies
[
  { _id: 619e4e79f828ad6a567d9192, name: 'movies' },
  { _id: 619e4e79f828ad6a567d9193, name: 'travel' }
]
```

However; if we set `mongoose.Schema.Types.DocumentArray.set('_id', false);` in the schema definition; array fields differ a bit. Array field with sub-document objects still has `_id` added as shown below

```
// friends; no _id
[ { name: 'ABC' }, { name: 'XYZ' } ]

// hobbies; _id still added
[
  { _id: 619e51a3a341db6aec55c259, name: 'movies' },
  { _id: 619e51a3a341db6aec55c25a, name: 'travel' }
]

```


## Preferred way to define schema

In my personal opinion; embedded documents should follow [sub-document schema definition](https://mongoosejs.com/docs/5.x/docs/subdocs.html#what-is-a-subdocument-). I also prefer to disable default auto addition of `_id` field. Sub-documents schemas can have middleware, custom validation logic, virtuals, and any other feature top-level schemas can use. [Also, validations in nested objects is tricky as they are not fully fledged paths](https://mongoosejs.com/docs/validation.html#required-validators-on-nested-objects).


## Conclusion

There is subtle difference between sub-document and nested document in Mongoose. In my personal view sub-document schema definition should be the preferred way for embedded document.

✨ Thank you for reading and I hope you find it helpful. I sincerely request for your feedback in the comment's section. You can follow me on twitter [@lifeClicks25](https://twitter.com/lifeClicks25).


