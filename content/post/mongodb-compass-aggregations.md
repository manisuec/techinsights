---
layout: post
title: "Enhance MongoDB Aggregation Experience with MongoDB Compass Queries"
description: MongoDB Compass offers an aggregation pipeline builder and query saving features. Export aggregations, generate explain plans, and streamline debugging.
date: 2023-01-30 00:00:00 +0530
images: ['https://res.cloudinary.com/dkiurfsjm/image/upload/v1675067686/aggregation_g1ra8g.png']
thumbnail: 'https://res.cloudinary.com/dkiurfsjm/image/upload/v1675429691/MongoDB_jeatlj.jpg'
tags: ['mongodb', 'mongodb compass', 'aggregation pipeline']
keywords: 'mongodb,mongodb compass,aggregation,aggregation pipeline,compass,pipeline,queries'
categories: ['Mongodb']
url: 'mongodb/my-queries-aggregation-mongodb-compass'
---

MongoDB Compass, as of 2018, comes with an aggregation pipeline builder to help make prototyping and debugging easier. This feature allows developers to export their aggregations into different programming languages and use the code in their applications.

The version 1.32.x series of Compass includes 'My Queries' tab features saved queries and aggregations which can be used to generate explain plans & perform aggregations on the entire collection. Additionally, the aggregation results can be exported for further use.

![queries](https://res.cloudinary.com/dkiurfsjm/image/upload/v1675067686/myqueries_aukefs.png)

## My Queries Section

Compass provides the ability to store previous search results, allowing users to quickly access them in the future.

Previously, these saved aggregations were restricted to a single namespace. Hence it was difficult to search for and reuse these queries & aggregations across namespaces.

The new "My Queries" screen makes it very convenient to find all the saved queries and aggregations as everything is located in a single place. One can easily filter them by namespaces or use the search bar to look for a specific query from amongst all of them.

![namespace](https://res.cloudinary.com/dkiurfsjm/image/upload/v1675067687/namespace_qda4bn.png)

## Explain Plan for Aggregations

For larger collections, it is important to take into account performance best practices when creating aggregations in order to optimize their speed & efficiency. "Explain Plan" is a widely used method for analyzing an aggregation's performance & ensuring the correct indexes are being utilized. 

Explain Plan is now part of the process in creating aggregations. It provides insights into performance metrics, execution time & index setup with a single click, streamlining the workflow considerably.

Developers may find it difficult to run their aggregations on the complete dataset and export their results as they would with a query, when this occurs.

It's now easy to compile the data needed. By clicking the "Run" button, once can receive the results quickly. Also, option of exporting the data as either JSON or CSV files, is very useful and simplifies business processes when sharing key insights with other departments.

## Conclusion

MongoDB Compass new features simplifies the process of using MongoDB’s aggregation framework by allowing developers to get up and running quickly without needing to thoroughly read through documentation. This also helps in efficiently creating, reusing, and sharing aggregations with colleagues, evaluating their success rate, and executing them directly with Compass.

✨ Thank you for reading and I hope you find it helpful. I sincerely request for your feedback in the comment's section.

