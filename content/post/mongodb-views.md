---
layout: post
title: "Mongodb Views: A Secure Way to Access & Share Data"
thumbnail: 'https://res.cloudinary.com/dkiurfsjm/image/upload/v1675429691/MongoDB_jeatlj.jpg'
date: 2023-05-17 00:00:00 +0530
images: ['https://res.cloudinary.com/dkiurfsjm/image/upload/v1684323449/mongodb_views_ymdkdr.jpg']
tags: ['mongodb','security','database','access','schema']
keywords: 'mongodb,views,secure,database,nodejs,access,no sql'
categories: ['Mongodb']
url: 'mongodb/mongodb-views-secure-data-access'
---

Data access and sharing data across teams or third parties are a very common phenomenon. Sharing sensitive data is a strict no. However, why even share data which is irrelevant for the data consumer. Suppose a MongoDB collection contain sensitive information. To restrict the access to collection, MongoDB offers Role-Based Access Control (RBAC).

![](https://res.cloudinary.com/dkiurfsjm/image/upload/v1684323449/mongodb_views_ymdkdr.jpg)

However, there may be instances where you want to share your collection with a broader audience without exposing sensitive data or irrelevant information. For example, sharing collections with the marketing team for analytics purposes without divulging personally identifiable information (PII) or confidential data like credit card number etc. In this blog post, we will explore how to achieve this using MongoDB views in conjunction with MongoDB RBAC.

## Prerequisites

To follow along, you'll need either of the following:

- A MongoDB cluster with authentication enabled.

Assuming you already have an admin user on your cluster with full privileges or at least a user with permissions to create views, custom roles, and users.


## MongoDB Collection

Let us create a MongoDB Collection say 'Person' with some sample data as below:

```javascript
db.persons.insertMany([
    {
        _id: 100,
        first_name: 'John',
        last_name: 'Dave',
        age: 21,
        sex: 'M',
        credit_card_number: '4541-3510-0040-7153',
        date_of_expiry: '31-Dec-2023',
        email_id: 'dave.john@gmail.com'
    },
    {
        _id: 101,
        first_name: 'Anna',
        last_name: 'Smith',
        age: 23,
        sex: 'F',
        credit_card_number: '4541-7802-0568-8842',
        date_of_expiry: '25-Nov-2023',
        email_id: 'smith.anna@gmail.com'
    },
    // ... more records ...
]);
```

With MongoDB RBAC, we can restrict data access to 'Persons' collection. However, what if we want to share this data with the customer retention team so that they can contact the persons whose credit card is going to expire soon and send reminders about renewal. For this use case, age and sex seems irrelevant and credit card number is a sensitive data and should not be shared. The retention team only needs to know about basic information like name, date of expiry and contact details.

## MongoDB View

As disscussed above, we want to share 'Persons' collection with a wider audience with selective information only. We can create a view that only includes selected fields. Here's an example of creating a view named "reminder_view" using the `$project` stage:

```javascript
db.createView('reminder_view', 'persons', [{$project: {firstname: 1, lastname: 1, date_of_expiry: 1, email_id: 1}}]);
```

To verify that the view works correctly, you can execute the following command:

```
db.reminder_view.find();
```

The result will show the documents with the selected fields only, excluding the sensitive and irrelevant ones.

Result

```
[
  { _id: 100, firstname: 'John', lastname: 'Dave', date_of_expiry: '31-Dec-2023', email_id: 'dave.john@gmail.com' },
  { _id: 101, firstname: 'Anna', lastname: 'Smith', date_of_expiry: '25-Nov-2023', email_id: 'smith.anna@gmail.com' }
]
```

## Managing Data Access with MongoDB RBAC

In order to enable restricted access rights for our view in MongoDB, it is necessary to create a custom role. Create the `"view_access"` role with the necessary privileges to access the `"reminder_view"` collection:

```javascript
use admin
db.createRole({
    role: "view_access",
    privileges: [
        { resource: { db: "test", collection: "reminder_view" }, actions: ["find"] }
    ],
    roles: []
});
```

Next, we can create a user with the created role:

```javascript
use admin
db.createUser({user: 'view_user', pwd: '123', roles: ["view_access"]})
```

Now we can share the credentials of `view_user` with the customer retention team, who are effectively viewing the persons data only but only which are relevant and hiding any sensitive information.

## Conclusion

In this blog post, you learned how to share your MongoDB collections to a wider audience — even the most critical ones — without exposing sensitive data.

✨ Thank you for reading and I hope you find it helpful. I sincerely request for your feedback in the comment's section.

![](https://cdn-images-1.medium.com/max/1600/0*dMZ0BEHDv4MJYYGW.png)

> If you liked the above story, you can buy me a coffee to keep me energized for writing stories like this for you.