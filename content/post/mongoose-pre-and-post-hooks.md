---
layout: post
title: 'Understanding Mongoose Pre and Post middleware hooks'
description: Pre and post middleware hooks is a very useful feature in Mongoose. These hooks provide flexibility by executing functions before or after specified actions.
images: ['https://res.cloudinary.com/dkiurfsjm/image/upload/v1677072845/mongoose-hooks_cailhh.jpg']
thumbnail: 'https://res.cloudinary.com/dkiurfsjm/image/upload/v1676956718/mongoose_logo_hr3blb.jpg'
date: 2023-02-22 00:00:00 +0530
lastmod: 2026-02-07T00:00:30
tags: [mongoosejs, 'mongodb', 'javascript', 'database']
categories: ['Mongodb']
keywords: 'mongoose,middlewares,pre hooks,post hooks,document,subdocument'
url: 'mongodb/mongoose-pre-and-post-hooks-middlewares'
---

Pre and post middleware hooks is a very useful feature in Mongoose and provides a lot of flexibility during database operations like query, create, remove etc. Pre and post hooks are functions that are executed before or after a particular action that you specify. These hooks get triggered whenever you perform an operation with your database.

![mongoose hooks](https://res.cloudinary.com/dkiurfsjm/image/upload/v1677072845/mongoose-hooks_cailhh.jpg)

## Types of Middleware

Mongoose has 4 types of middleware: document middleware, model middleware, aggregate middleware, and query middleware.

| Document | Query | Aggregate | Model |
|:--------:|:-----:|:---------:|:-----:|
| validate | count | aggregate | insertMany |
| save | deleteMany | | |
| remove | deleteOne | | |
| updateOne | find | | |
| deleteOne | findOne | | |
| init*  | findOneAndDelete | | |
| | findOneAndRemove |  | |
| | findOneAndUpdate | | |
| | remove | | |
| | update | | |
| | updateOne | | |
| | updateMany | | |

**init hooks are synchronous*

**Note:** If you specify `schema.pre('remove')`, Mongoose will register this middleware for `doc.remove()` by default. If you want to your middleware to run on `Query.remove()` use `schema.pre('remove', { query: true, document: false }, fn)`.

**Note:** Unlike `schema.pre('remove')`, Mongoose registers `updateOne` and `deleteOne` middleware on `Query#updateOne()` and `Query#deleteOne()` by default. This means that both `doc.updateOne()` and `Model.updateOne()` trigger `updateOne` hooks, but this refers to a query, not a document. To register `updateOne` or `deleteOne` middleware as document middleware, use `schema.pre('updateOne', { document: true, query: false })`.

**Note:** The `create()` function fires `save()` hooks.

## Defining middlewares

- **use normal callback with next parameter**

The `next` parameter is a function provided to you by mongoose to have a way out, or to tell mongoose you are done and to continue with the next step in the execution chain. Also it is possible to pass an `Error` to `next` it will stop the execution chain.

```
schema.pre('save', function(next) {
  // do stuff
  
  if (error) { return next(new Error("something went wrong"); }
  return next(null);
});
```

- **use async callback**

Here the execution chain will continue once your `async` callback has finished. If there is an error and you want to break/stop execution chain you just `throw` it

```
schema.pre('save', async function() {
  // do stuff
  await doStuff()
  await doMoreStuff()

  if (error) { throw new Error("something went wrong"); }
  return;
});
```
> **Note 1:-** You must add all middleware and plugins before calling mongoose.model().

> **Note 2:-** Remember we cannot use arrow function inside pre hooks as arrow function are poor binding functions.

## Understanding Middlewares with an example

Let us define a `User` schema as below:

```javascript
const userSchema = new mongoose.Schema({
  _id      : { type: String },
  name     : { type: String },
  username : { type: String },
  password : { type: String }
});


userSchema.pre('save', async function() {
  console.log('pre hook: validate username');
  const user = this;
  const regex = /^[a-zA-Z0-9]+$/;
  const result = regex.test(user.username);

  if (result) {
    console.log('username validated');

    return Promise.resolve();
  } else {
    console.log('this will stop execution of next middlewares');
    throw new Error('username is not alphanumeric');
  }
});

userSchema.pre('save', async function() {
  console.log('pre hook: generate password hash');
  const user = this;

  const passHash = await getSHAHash(user.password);
  console.log('password hash generated');
  user.password = passHash;
});


userSchema.post('save', async doc => {
  console.log('post hook called');
  console.log('notify/send email');
});

const User = mongoose.model('User', userSchema);

userSchema.pre('save', async () => {
  console.log('pre hook: after calling mongoose.model()');
  console.log('this pre hook will not be called');
});
```

In the above schema;

- a `pre hook` to validate whether the provided `username` contains only alphanumberic characters
- a `pre hook` to generate hash of the user password
- a `post hook` to notify/send email
- a `pre hook` added post `mongosse.model()`

Run the below code:

```javascript
const init = async () => {
  try {
    console.log('start of the script');
    await User.create({ _id: Date.now(), name: 'Manish', username: 'manisuec', password: 'abc123' });
    console.log('first user created successfully');
    console.log('-------------------------------------------');
    await User.create({ _id: Date.now(), name: 'Ravi', username: 'ravi@123', password: 'abc123' });
  } catch (err) {
    console.log('error:', err.message);
  }
}

init();
```

You should see output as below and I guess the output is self-explanatory:

```bash
pre hook: validate username
username validated

pre hook: generate password hash
password hash generated

post hook called
notify/send email
first user created successfully
-------------------------------------------
pre hook: validate username
this will stop execution of next middlewares
error: username is not alphanumeric
```

The sample code is checked into [mongoose-hooks](https://github.com/manisuec/techinsights-tutorials/tree/main/mongoose-hooks) on github.

## Use Cases

Pre-hooks can be utilized in a variety of scenarios, such as custom validations, generating hash password etc. You can also use them when you have a field named say `"archived"` in your schema and you wish to exclude all archived documents in each `"find"` call. Including a filter for `"archived: false"` in every `"find"` call may be cumbersome and error-prone. In such cases, pre-hooks can serve as query middleware to simplify the process. For instance, a simple pre-hook on the `"find"` method in Mongoose can modify the query and add an extra filter, which will be applied to all subsequent `"find"` queries for that model.

Another use case can be to add additional field say `last_updatedMS` by default which saves epoch timestamp value on every `update` operation.

Hooks have a multitude of use cases depending on your application and I hope you can identify potential use cases for your own application.

## Best Practices for Using Hooks

1. Keep hooks focused and single-purpose
2. Avoid complex business logic in hooks
3. Use async/await properly in hooks
4. Handle errors appropriately
5. Consider performance implications

## Related Articles
- [Faster Mongoose Queries with Lean](https://techinsights.manisuec.com/mongodb/mongoose-queries-faster-lean/) - Optimize your Mongoose queries
- [Understanding Mongoose Discriminators](https://techinsights.manisuec.com/mongodb/mongoose-discriminator/) - Master Mongoose inheritance patterns
- [Working with Subdocuments and Nested Documents](https://techinsights.manisuec.com/mongodb/understand-mongoose-subdoc-and-nesteddoc/) - Deep dive into document relationships
- [MongoDB Performance Best Practices](https://techinsights.manisuec.com/mongodb/mongodb-best-practices/) - General MongoDB optimization tips

## Conclusion

I hope that in this article, you have gained an understanding of how and when to utilize pre and post hooks in mongoose, as well as the benefits they provide in comparison to writing repetitive logic for actions performed on collection documents.

By implementing these hooks, we can reduce the amount of code we need to write, resulting in a smaller potential for bugs. Additionally, we can alleviate the mental burden of having to remember to execute certain logic before or after a method each time. There are various other use cases for pre and post middleware hooks, and their execution before the method they are hooked onto makes them a versatile tool.

✨ Thank you for reading and I hope you find it helpful. I sincerely request for your feedback in the comment's section.

