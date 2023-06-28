---
background: /img/posts/04.jpg
thumbnail: 'https://res.cloudinary.com/dkiurfsjm/image/upload/v1676698473/nodejs_dark_cjoudy.png'
date: '2020-03-02T00:00:00Z'
title: 'Streams: An underrated but very powerful concept in NodeJS'
tags: ['nodejs', 'streams', 'data processing']
keywords: 'nodejs,streams,transform,javascript,data,memory,large data'
categories: ['Nodejs']
url: 'nodejs/nodejs-streams'
aliases:
    - /post/2020-03-02-streams-nodejs/
---

A stream is an abstract interface for working with streaming data in Node.js. Streams have gained the reputation that it is hard to work with and harder to understand. However, it is a highly underrated but very powerful concept in Node.js. This article will help in understanding of streams, how to work with them and where to use this module.

## Streams: Introduction

The official documentation of Node.js defines stream as an abstract interface for working with streaming data. It is one of the fundamental concept in Node.js and is very powerful when working with large amounts of data, e.g., reading a very large file size. Streams are memory efficient as there is no need to load large amount of data in memory instead read chunks of data piece by piece and process the contents. Also, since one doesn't have to wait for all the data to load first, it is time efficient too.

For a complete introduction of Stream module and its related apis, read the [official documentation.](https://nodejs.org/dist/latest-v12.x/docs/api/stream.html)

## Streams: A use case

For the purpose of showing the powerful capability of streams, let us do the following operations.

1. Read data from a large file.
2. Filter the data.
3. Transform the input data.
4. Write the transformed data into a file.

I have downloaded the [Loan Data for Dummy Bank](https://www.kaggle.com/mrferozi/loan-data-for-dummy-bank/data) file for this article. I converted this file into json format with each line representing a json object. Let us read the data from the file, filter out all those who have 'RENT' as value for 'home_ownership' column. Add a new field 'risk' with value as low for loan_amount <= 5000 and high otherwise and write into a new file.

First lets create file read and write stream

```
const readStream = fs.createReadStream(path.resolve(READ_FILE_PATH));
const writeStream = fs.createWriteStream(path.resolve(WRITE_FILE_PATH));
```

Create a filter data stream and filter condition to filter out data having 'home_ownership' as RENT.

```
const filterCondition = elem => {
  if (elem) {
    return elem.home_ownership === 'RENT';
  }

  return false;
};

const filterData = (fn, options = {}) =>
  new Transform({
    objectMode: true,
    ...options,

    transform(chunk, encoding, callback) {
      let take;
      let obj;
      try {
        obj = JSON.parse(chunk.toString());
      } catch (e) {}
      try {
        take = fn(obj);
      } catch (e) {
        return callback(e);
      }
      return callback(null, take ? chunk : undefined);
    },
  });
```

Similarly, create a transform data stream to add field 'risk' with value as 'high' for 'loan_amount' > 5000 else value as 'low'.

```
const riskProfile = elem => {
  elem.risk_profile = elem.loan_amount > 5000 ? 'high' : 'low';
  return JSON.stringify(elem);
};

const transformData = (fn, options = {}) =>
  new Transform({
    objectMode: true,
    ...options,

    transform(chunk, encoding, callback) {
      let take;
      try {
        take = fn(JSON.parse(chunk));
      } catch (e) {
        return callback(e);
      }
      return callback(null, `${take}\n`);
    },
  });
```

I have also used 'split' module to create read chunks line by line. Using 'pipeline' module method of Nodejs pipe all the streams. This method pipes streams forwarding errors and properly cleaning up and provide a callback when the pipeline is complete.

```
pipeline(
  readStream,
  split(),
  filterData(filterCondition),
  transformData(riskProfile),
  writeStream,
  err => {
    if (err) {
      console.error('Pipeline failed.', err);
    } else {
      console.log('Pipeline succeeded.');
    }
  },
);
```

## Conclusion

In this example, we can see that by using Nodejs streams, we can manipulate large sets of data very efficiently both in terms of memory and time. I reiterate, streams are a highly underrated but very powerful concept in Node.js. Hopefully, this article will help in understanding of streams, how to work with them and where to use this module.

Note:- Complete code is checked into [github](https://github.com/manisuec/study/tree/master/streams)

