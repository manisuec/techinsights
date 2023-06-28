---
layout: post
title: 'Mongoose Discriminator: The non DRY way to inherit schema properties'
images: ['https://res.cloudinary.com/dkiurfsjm/image/upload/v1677493083/mongoose-discriminator_c5elpt.png']
thumbnail: 'https://res.cloudinary.com/dkiurfsjm/image/upload/v1676956718/mongoose_logo_hr3blb.jpg'
date: 2023-02-27 00:00:00 +0530
tags: ['mongoose', 'mongodb', 'javascript']
categories: ['Mongodb']
keywords: 'mongoose,discriminator,dry,polymorphism,mongodb'
url: 'mongodb/mongoose-discriminator-non-dry-way-inherit-properties'
---

Mongoose Discriminator is another very useful and powerful  yet underused feature of Mongoose. It serves as a means of schema inheritance, allowing you to use multiple models with intersecting schemas on the same underlying MongoDB collection. You can store different entities in one collection, and you will still be able to discriminate between these entities. Unfortunately, the documentation of this feature is poor.

![](https://res.cloudinary.com/dkiurfsjm/image/upload/v1677493083/mongoose-discriminator_c5elpt.png)

## Why use Discriminator?

Suppose there is a need to run 'campaign' of different types for your customers:

- SMS campaign
- Campaign via app notification
- Email campaign

The campaign model will have below fields in common for all

```javascript
{
    customer_id         : { type: String },
    campaign_name       : { type: String },
    campaign_start_date : { type: Date },
    campaign_end_date   : { type: Date }
}
```

SMS based campaign will have `text: { type: String, required: true }` as additional field.

App notification campaign will have below additional fields

```javascript
{
	title : { type: String, required: true },
	body  : { type: String, required: true }
}
```

Email campaign will have below additional fields

```javascript
{
	subject   : { type: String, required: true },
	plan_text : { type: String, required: true },
	html_text : { type: String, required: true }
}
```

Depending upon the type of campaign, the details will be mandatory accordingly. e.g. For SMS type campaign, `text` field will be mandatory but for email, `subject`, `plain_text` and `html_text` will be mandatory.

These are the below alternatives to define the schemas for the above use case

### Option 1: Separate Schemas

Define separate schemas for each type of campaign namely `SMSCampaign`, `NotifCampaign` and `EmailCampaign` having all the fields needed for each. This way you can enforce schema validations for each type of campaign. But, this is breaking `DRY (Do not Repeat Yourself)` design principle. Also, if you have to introduce a new property, say: `periodic: {type: Boolean}` common to all then, you have to add in all three schemas and forgetting for any one type will create bugs.

### Option 2: Define a `mixed` type field

```javascript
{
    customer_id         : { type: String },
    campaign_name       : { type: String },
    campaign_start_date : { type: Date },
    campaign_end_date   : { type: Date },
    campaign_details	: { type: mongoose.Mixed }
}

```

This will allow to store campaign details for each type but you will loose schema validations and type casting features of mongoose.

### Option 3: Mongoose Discriminator

You can define a base campaign schema with common fields and make use of `mongoose.discriminator()` function to differentiate between the different campaigns and yet use the single collection.

```javascript
const mongoose = require("mongoose");

const baseOptions = {
	discriminatorKey: "type",
  	collection: "campaign",
};

const baseCampaignSchema = new mongoose.Schema(
 {
	customer_id         : { type: String },
	campaign_name       : { type: String },
	campaign_start_date : { type: Date },
	campaign_end_date   : { type: Date },
 }, 
 baseOptions
);

const BaseCampaign = mongoose.model('Campaign', baseCampaignSchema);

const SMSCampaign = BaseCampaign.discriminator('SMSCampaign', new mongoose.Schema({ 
	text: { type: String, required: true }
	})
);

const NotifCampaign = BaseCampaign.discriminator('NotifCampaign', new mongoose.Schema({ 
	title : { type: String, required: true },
	body  : { type: String, required: true }
	})
);

const EmailCampaign = BaseCampaign.discriminator('EmailCampaign', new mongoose.Schema({ 
	subject   : { type: String, required: true },
	plain_text: { type: String, required: true },
	html_text : { type: String, required: true }
	})
);

const sms = new SMSCampaign({
	customer_id         : 'CMP_100',
	campaign_name       : 'SMS 100',
	campaign_start_date : new Date(1677497579000),
	campaign_end_date   : new Date(1679916779000),
	text					: 'This is a sms campaign'
});

const appNotif = new NotifCampaign({
	customer_id         : 'CMP_101',
	campaign_name       : 'Notif 101',
	campaign_start_date : new Date(1677497579000),
	campaign_end_date   : new Date(1679916779000),
	title					: 'This is app notif title',
	body					: 'This is app notif body',
});

const email = new EmailCampaign({
	customer_id         : 'CMP_102',
	campaign_name       : 'Email 102',
	campaign_start_date : new Date(1677497579000),
	campaign_end_date   : new Date(1679916779000),
	subject				: 'This is email campaign',
	plain_text			: 'This is email body in plain text',
	html_text				: '<p>This is email body in html text</p>',
});

await Promise.all([sms.save(), appNotif.save(), email.save()]);
    
const count = await BaseCampaign.countDocuments();
console.log('Total:', count); // Total: 3

```

By defining the schema as above, you have not only inherited the properties of base campaign but also created different models for campaign types. The data will be stored in the single collection only.

> You may have noticed { discriminatorKey: “type” } in the `baseOptions`. If you don’t give this option mongoose will use default discrimonator key “__t”.

The document saved will look like 

```javascript
{
  "_id": "63fc9d9ef9ed75a85b64c066",
  "type": "SMSCampaign",
  "customer_id": "CMP_100",
  "campaign_name": "SMS 100",
  "campaign_start_date": "2023-02-27T11:32:59.000+00:00",
  "campaign_end_date": "2023-03-27T11:32:59.000+00:00",
  "text": "This is a sms campaign",
  "__v": 0
}
```

The sample code is checked into [mongoose-discriminator](https://github.com/manisuec/techinsights-tutorials/tree/main/mongoose-discriminator) on github.

### Advantages of using discriminator:

- A clear schema definition for each type
- Mongoose validation, type casting and hooks
- Easy population of data

## Updating the discriminator key

By default, Mongoose doesn't let you update the discriminator key. `save()` will throw an error if you attempt to update the discriminator key. And `findOneAndUpdate()`, `updateOne()`, etc. will strip out discriminator key updates.

To update a document's discriminator key, use `findOneAndUpdate()` or `updateOne()` with the `overwriteDiscriminatorKey` option.

[Read more here...](https://mongoosejs.com/docs/discriminators.html#updating-the-discriminator-key)

## Conclusion

Discriminators in Mongoose are a robust feature that allows for storing comparable documents with varying schema constraints in a single collection. They are extremely useful in scenarios where using a `Mixed type` and bypassing `validation` is tempting. Specifically, in applications such as event tracking and user permissions, discriminators can be an essential tool.

✨ Thank you for reading and I hope you find it helpful. I sincerely request for your feedback in the comment's section.

