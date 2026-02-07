---
layout: post
title: 'Mongoose Plugins Made Simple: A Beginner Friendly Guide'
description: 'A clear, step-by-step introduction to Mongoose plugins: what they are, how to create and use them.'
images: ['https://res.cloudinary.com/dkiurfsjm/image/upload/v1747742073/lnyixzho3mucjc1v9-Hero-image_mzdnh1.svg']
thumbnail: 'https://res.cloudinary.com/dkiurfsjm/image/upload/v1676956718/mongoose_logo_hr3blb.jpg'
date: 2025-07-15 00:00:00 +0530
lastmod: 2026-02-07T00:00:30
tags: ['mongoosejs', 'mongodb', 'nodejs', 'plugins', 'schema']
categories: ['Mongodb']
keywords: 'mongoose,plugins,mongodb,nodejs,schema,validation,audit,soft delete'
url: 'mongodb/mongoose-plugins-guide'
---

Mongoose plugins are like add-ons for your database models. They let you reuse features (like timestamps, soft deletes or validation) across different parts of your app, so you don’t have to write the same code over and over. They also promote code reusability, simplicity and maintaining consistency across your application. This guide will show you, step by step, how to use and create Mongoose plugins; even if you are new to Mongoose or MongoDB.

## What is a Mongoose Plugin?

A Mongoose plugin is just a function. You give it a schema (the blueprint for your data) and, if you want, some options. The plugin can then add new fields, methods, or rules to your schema. Think of it as a way to teach your models new tricks!

## Why Use Plugins?

- **Save time:** Write a feature once, use it everywhere.
- **Stay organized:** Keep your code clean and DRY (Don’t Repeat Yourself).
- **Consistency:** Make sure all your models behave the same way.

## Example: Adding Timestamps Automatically

Let’s say you want every document in your database to remember when it was created and last updated. Instead of adding `createdAt` and `updatedAt` fields to every schema, you can use a plugin.

**Step 1: Make the Plugin**

```javascript
// plugins/timestamps.js
function timestampsPlugin(schema) {
  schema.add({
    createdAt: { type: Date, default: Date.now, immutable: true },
    updatedAt: { type: Date, default: Date.now }
  });

  schema.pre('save', function(next) {
    if (!this.isNew) {
      this.updatedAt = Date.now();
    }
    next();
  });
}

module.exports = timestampsPlugin;
```

**Step 2: Use the Plugin in Your Schema**

```javascript
const mongoose = require(mongoosejs);
const timestampsPlugin = require('./plugins/timestamps');

const userSchema = new mongoose.Schema({
  name: String,
  email: String
});

userSchema.plugin(timestampsPlugin);

const User = mongoose.model('User', userSchema);
```

Now, every `User` will have `createdAt` and `updatedAt` fields that update automatically.

## Example: Soft Delete (Don’t Really Delete)

Sometimes you want to "delete" something, but not really remove it from the database—just hide it. This is called a "soft delete."

**Step 1: Make the Plugin**

```javascript
// plugins/softDelete.js
function softDeletePlugin(schema) {
  schema.add({
    deleted: { type: Boolean, default: false },
    deletedAt: { type: Date, default: null }
  });

  schema.methods.softDelete = function() {
    this.deleted = true;
    this.deletedAt = new Date();
    return this.save();
  };

  schema.methods.restore = function() {
    this.deleted = false;
    this.deletedAt = null;
    return this.save();
  };

  schema.statics.findNotDeleted = function() {
    return this.find({ deleted: false });
  };
}

module.exports = softDeletePlugin;
```

**Step 2: Use the Plugin**

```javascript
const softDeletePlugin = require('./plugins/softDelete');

const postSchema = new mongoose.Schema({
  title: String,
  content: String
});

postSchema.plugin(softDeletePlugin);
const Post = mongoose.model('Post', postSchema);

// Usage
const post = await Post.create({ title: 'Hello', content: 'World' });
await post.softDelete(); // Marks as deleted, but not removed
const posts = await Post.findNotDeleted(); // Only gets posts not deleted
```

## Example: Simple Validation Plugin

Let’s make a plugin that checks if an email field is valid.

```javascript
// plugins/emailValidation.js
function emailValidationPlugin(schema, options) {
  const field = (options && options.field) || 'email';
  schema.path(field).validate(function(email) {
    return /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(email);
  }, 'Invalid email format');
}

module.exports = emailValidationPlugin;
```

**How to Use:**

```javascript
const emailValidationPlugin = require('./plugins/emailValidation');

const userSchema = new mongoose.Schema({
  email: String
});

userSchema.plugin(emailValidationPlugin, { field: 'email' });
```

Now, if you try to save a user with a bad email, Mongoose will throw an error.

## Using Third Party Plugins

You don’t have to write every plugin yourself! There are many great plugins you can use right away.

### Pagination Example

```javascript
const mongoosePaginate = require('mongoose-paginate-v2');

const bookSchema = new mongoose.Schema({
  title: String,
  author: String
});

bookSchema.plugin(mongoosePaginate);
const Book = mongoose.model('Book', bookSchema);

// Usage
const result = await Book.paginate({}, { page: 1, limit: 5 });
console.log(result.docs); // Books on this page
```

### Unique Validator Example

```javascript
const uniqueValidator = require('mongoose-unique-validator');

const userSchema = new mongoose.Schema({
  email: { type: String, unique: true }
});

userSchema.plugin(uniqueValidator);
```

## How to Apply a Plugin to All Schemas

You can make every schema in your app use a plugin by calling `mongoose.plugin()` before you create any models:

```javascript
const mongoose = require(mongoosejs);
const timestampsPlugin = require('./plugins/timestamps');

mongoose.plugin(timestampsPlugin);
```

Now, every new schema will have timestamps automatically.

## Tips for Writing Good Plugins

- **Keep it simple:** Do one thing well.
- **Let users customize:** Accept options so people can tweak how your plugin works.
- **Test your plugin:** Make sure it works in different situations.
- **Document it:** Explain what your plugin does and how to use it.
- **Handle errors gracefully:** Don’t let plugin errors crash the application
- **Consider performance:** Be mindful of the impact on query performance


> *Struggling with MongoDB performance issues, data validation errors, or transaction complexity? Read indepth Mongoose and Mongodb blogs covering the essential techniques every developer needs:*
> 
> *a) [Master schema validation to prevent bad data from entering your database](https://techinsights.manisuec.com/mongodb/mongoose-schema-validation)*
> 
> *b) [Eliminate performance bottlenecks with proven optimization strategies](https://techinsights.manisuec.com/mongodb/mongodb-index-optimization)*
> 
> *c) [Implement reliable transaction handling in Mongodb for critical operations](https://techinsights.manisuec.com/mongodb/mongoose-transactions)*

## Testing a Plugin (Example)

Here’s a simple test for the timestamps plugin using Jest:

```javascript
const mongoose = require(mongoosejs);
const timestampsPlugin = require('../plugins/timestamps');

describe('Timestamps Plugin', () => {
  let TestModel;

  beforeAll(async () => {
    await mongoose.connect('mongodb://localhost/test');
    const testSchema = new mongoose.Schema({ name: String });
    testSchema.plugin(timestampsPlugin);
    TestModel = mongoose.model('Test', testSchema);
  });

  afterAll(async () => {
    await mongoose.connection.close();
  });

  it('adds createdAt and updatedAt', async () => {
    const doc = await TestModel.create({ name: 'Test' });
    expect(doc.createdAt).toBeInstanceOf(Date);
    expect(doc.updatedAt).toBeInstanceOf(Date);
  });
});
```

## Conclusion

Mongoose plugins are a great way to add features to your models without repeating yourself. Mongoose plugins let you enhance your models with new features while avoiding repetitive code. They help you package shared functionality and use it uniformly throughout your application. Start with simple plugins, use them in your schemas and soon you’ll be building cleaner, more powerful apps. Remember: keep it simple, test your code and have fun!