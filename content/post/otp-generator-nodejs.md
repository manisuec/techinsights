---
layout: post
title: 'A One Time Password (OTP) generator npm library based on nanoid'
date: 2023-01-05 00:00:00 +0530
images: ['https://res.cloudinary.com/dkiurfsjm/image/upload/v1672914714/otp_tqczzd.png']
thumbnail: 'https://res.cloudinary.com/dkiurfsjm/image/upload/v1676698473/nodejs_dark_cjoudy.png'
tags: ['nodejs', 'javascript']
keywords: 'nodejs,otp,npm,otp generator,nano id,javascript,nanoid,one time password,generator'
categories: ['Nodejs']
url: 'nodejs/otp-generator-nodejs'
---

Mobile number has become the defacto user authentication mechanism in India and hence, OTP generation is a very common use case. [otp-gen-agent](https://github.com/manisuec/otp-gen-agent#readme) is a [Nano ID](https://github.com/ai/nanoid#readme) based small utility lib to generate OTP (one time password).

## Why avoid Math.random()?

In the [documentation for Math.random()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Math/random); the note mentions

> Math.random() does not provide cryptographically secure random numbers. Do not use them for anything related to security. Use the Web Crypto API instead, and more precisely the window.crypto.getRandomValues() method.

Read the blog, a real world scenario, [Facebook JavaScript API](http://ifsec.blogspot.com/2012/09/of-html5-security-cross-domain.html); where the attacker was able to exploit the vulnerability.

## Installation

```npm install otp-gen-agent --save```

## Usage

Nano ID is a tiny, secure, URL-friendly, unique string ID generator for JavaScript.

- Small: 130 bytes (minified and gzipped). No dependencies. Size Limit controls the size.
- Safe: It uses hardware random generator. Can be used in clusters.

Read more in the section [Security](https://github.com/ai/nanoid#security).

### i) default
```js
const { otpGen } = require('otp-gen-agent');

const otp = await otpGen(); // '344156'  (OTP length is 6 digit by default)

```
  - Default OTP lenght is 6
  - Default characters used to generate OTP is 0123456789

### ii) custom otp generator

```js
const { customOtpGen } = require('otp-gen-agent');

const otp = await customOtpGen({length: 4, chars: 'abc123'}); // 'a3c1'

```

**arguments:** 

- options: optional
    - length: custom otp length
    - chars: custom characters

You can customise the OTP length and also the characters to be used for OTP generation.
  
### iii) bulk otp generator

```js
const { bulkOtpGen } = require('otp-gen-agent');

const otp = await bulkOtpGen(2); // Array of otps: ['344156', '512398']

```

```js
const { bulkOtpGen } = require('otp-gen-agent');

const otp = await bulkOtpGen(2, {length = 5, chars: 'abcd123'} ); // Array of otps: ['312b3', 'bcddd']

```

**arguments:** 
	
- num: number of OTPs to be generated in bulk
- opts: optional argument
	- length: custom otp length (default: 6)
	- chars: custom characters (default: 0123456789)

Useful in cases where number of OTPs to be generated is known before hand.

## Conclusion

Nano ID uses the crypto module in Node.js and the Web Crypto API in browsers. These modules use unpredictable hardware random generator. [`otp-gen-agent`](https://www.npmjs.com/package/otp-gen-agent?activeTab=readme) is a small utility lib based on Nano ID for generating otp.

✨ Thank you for reading and I hope you find it helpful. I sincerely request for your feedback in the comment's section.

[![](https://cdn-images-1.medium.com/max/1600/0*dMZ0BEHDv4MJYYGW.png)](https://www.buymeacoffee.com/manisuec)

> If you liked the above story, you can [buy me a coffee](https://www.buymeacoffee.com/manisuec) to keep me energized for writing stories like this for you.