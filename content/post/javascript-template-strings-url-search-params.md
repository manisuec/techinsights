---
layout: post
title: 'Stop Using JavaScript’s Template Literals for building URLs with Query Params'
images: ['https://res.cloudinary.com/dkiurfsjm/image/upload/v1687259385/ts_url_jydkn8.png']
thumbnail: 'https://res.cloudinary.com/dkiurfsjm/image/upload/v1675429691/JavaScript_v4qblf.jpg'
date: 2023-06-20 00:00:00 +0530
tags: ['javascript', 'template literals','url']
categories: ['Javascript']
keywords: 'javascript,template literals,url search params,url,params,template strings,strings,query,query params'
url: 'javascript/template-strings-url-search-params'
---

[Template literals](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Template_literals) are literals delimited with backtick (`) characters, allowing for multi-line strings, string interpolation with embedded expressions, and special constructs called tagged templates.

Template literals are sometimes informally called template strings, because they are used most commonly for string interpolation (to create strings by doing substitution of placeholders) and has gained popularity very quickly.

> **String interpolation:** By using placeholders of the form ${expression} to perform  substitutions for embedded expressions:

```
const a = 5;
const b = 10;
console.log(`Fifteen is ${a + b} and
not ${2 * a + b}.`);
// "Fifteen is 15 and
// not 20."
```

However, template literals creates issues and not recommended to build URLs with query params for fetch APIs.

![](https://res.cloudinary.com/dkiurfsjm/image/upload/v1687259385/ts_url_jydkn8.png)

## Issue with building Query params with Template Literals

While going through React official documentation [When to use custom Hooks](https://react.dev/learn/reusing-logic-with-custom-hooks#when-to-use-custom-hooks); I saw below code snippet.

```
// This Effect fetches cities for a country
  useEffect(() => {
    let ignore = false;
    fetch(`/api/cities?country=${country}`)
      .then(response => response.json())
      .then(json => {
        if (!ignore) {
          setCities(json);
        }
      });
    return () => {
      ignore = true;
    };
  }, [country]);
```

I have also seen many juniors using the same way of building query params for fetch APIs.

> Template literals aren’t designed for creating strings to serialize and pass to other systems.

Lets take example of country Trinidad and Tobago also sometimes written as Trinidad & Tobago. If we use template strings; then the query sent to the server will be `/api/cities?country=Trinidad & Tobago`. The server will parse the request query params as `country` with value `Trinidad` and ` Tobago`(note the leading space!) with an empty value. This kind of issues are very difficult to identify and debug.

Another issue can be due to how template literals interpret array of primitive types. e.g.

```
const arr = [1,2,3,4,5];
`${arr}` ===> '1,2,3,4,5'
```

## URLSearchParams

The [URLSearchParams](https://developer.mozilla.org/en-US/docs/Web/API/URLSearchParams) interface defines utility methods to work with the query string of a URL. Using built-in answer JavaScript `URLSearchParams`, which does take care of many problems. For example, instead of this:

`/api/cities?country=${country}`

```
'/api/cities?' +
  new URLSearchParams([
    [ 'country', country ]
  ])

```

Resulting URL will be `'/api/cities?country=Trinidad+%26+Tobago'` and the request query params will be correctly interpreted by server.

URLSearchParams does have its own caveats but preferring it over template strings, you’ll be in good shape. Read more at [URLSearchParams.](https://developer.mozilla.org/en-US/docs/Web/API/URLSearchParams#preserving_plus_signs) 

> String formatting tools are for printing, not for serialization.

Best solution is definitely leveraging TypeScript or validation libraries like Joi.

## Conclusion

I hope this article helps you understand about template literals a bit more. My main motivation of writing this article was mainly due to seeing many junior folks using template literals for building URLs with query params of fetch APIs.

✨ Thank you for reading and I hope you find it helpful. I sincerely request for your feedback in the comment's section.

[![](https://cdn-images-1.medium.com/max/1600/0*dMZ0BEHDv4MJYYGW.png)](https://www.buymeacoffee.com/manisuec)

> If you liked the above story, you can [buy me a coffee](https://www.buymeacoffee.com/manisuec) to keep me energized for writing stories like this for you.