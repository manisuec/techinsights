---
layout: post
title: 'Avoid JavaScript Template Literals for Building URLs with Query Params'
description: "Learn why template literals should be avoided for URL construction and how to properly handle query parameters using URLSearchParams. A comprehensive guide with practical examples."
images: ['https://res.cloudinary.com/dkiurfsjm/image/upload/v1687259385/ts_url_jydkn8.png']
thumbnail: 'https://res.cloudinary.com/dkiurfsjm/image/upload/v1675429691/JavaScript_v4qblf.jpg'
date: 2023-06-20 00:00:00 +0530
lastmod: 2026-02-07T00:00:30
tags: ['javascript', 'template literals', 'url', 'best practices', 'web development']
categories: ['Javascript']
keywords: 'javascript,template literals,url search params,url,params,template strings,strings,query,query params'
url: 'javascript/template-strings-url-search-params'
---

[Template literals](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Template_literals) are literals delimited with backtick (`) characters, allowing for multi-line strings, string interpolation with embedded expressions, and special constructs called tagged templates.

JavaScript template literals, introduced in ES6, are a powerful feature for string manipulation. They are delimited with backtick (`) characters and provide three main capabilities:

1. Multi-line strings
2. String interpolation
3. Tagged templates

## Template Literals: Basic Usage

### String Interpolation
The most common use case is string interpolation using `${expression}` syntax:

```javascript
const name = "John";
const age = 30;
const greeting = `Hello, my name is ${name} and I am ${age} years old.`;
console.log(greeting);
// Output: "Hello, my name is John and I am 30 years old."
```

### Multi-line Strings
Template literals preserve whitespace and line breaks:

```javascript
const multiLine = `
  First line
  Second line
  Third line
`;
console.log(multiLine);
// Output:
//   First line
//   Second line
//   Third line
```

### Expression Evaluation
Template literals can evaluate expressions:

```javascript
const a = 5;
const b = 10;
console.log(`The sum of ${a} and ${b} is ${a + b}`);
// Output: "The sum of 5 and 10 is 15"
```

![template string url](https://res.cloudinary.com/dkiurfsjm/image/upload/v1687259385/
ts_url_jydkn8.png)

## The Problem with Template Literals in URLs

While template literals are convenient, they can cause significant issues when used for URL construction, especially with query parameters. Let's examine why:

### Common Anti-pattern

```javascript
// ❌ Bad Practice
const country = "Trinidad & Tobago";
const url = `/api/cities?country=${country}`;

// Results in: /api/cities?country=Trinidad & Tobago
```

This approach has several problems:

1. **Special Characters**: Characters like `&`, `?`, `=`, and spaces are not properly encoded
2. **Array Handling**: Arrays are converted to comma-separated strings
3. **Type Coercion**: Objects are converted to `[object Object]`

### Real-world Example of Issues

```javascript
// ❌ Problematic Implementation
const params = {
  country: "Trinidad & Tobago",
  cities: ["Port of Spain", "San Fernando"],
  page: 1,
  limit: 10
};

const badUrl = `/api/cities?country=${params.country}&cities=${params.cities}&page=${params.page}&limit=${params.limit}`;

// Results in: 
// /api/cities?country=Trinidad & Tobago&cities=Port of Spain,San Fernando&page=1&limit=10
```

## The Solution: URLSearchParams

`URLSearchParams` is the proper way to handle URL query parameters in JavaScript. It automatically handles:
- URL encoding
- Special characters
- Array parameters
- Multiple values for the same parameter

### Basic Usage

```javascript
// ✅ Good Practice
const params = new URLSearchParams({
  country: "Trinidad & Tobago",
  page: "1",
  limit: "10"
});

const url = `/api/cities?${params.toString()}`;

// Results in: 
// /api/cities?country=Trinidad+%26+Tobago&page=1&limit=10
```

### Advanced Usage

```javascript
// ✅ Handling Complex Parameters
const searchParams = new URLSearchParams();

// Adding single parameters
searchParams.append('country', 'Trinidad & Tobago');

// Adding multiple values for the same parameter
searchParams.append('cities', 'Port of Spain');
searchParams.append('cities', 'San Fernando');

// Adding numeric values
searchParams.append('page', '1');
searchParams.append('limit', '10');

const url = `/api/cities?${searchParams.toString()}`;

// Results in: 
// /api/cities?country=Trinidad+%26+Tobago&cities=Port+of+Spain&cities=San+Fernando&page=1&limit=10
```

### Working with Arrays

```javascript
// ✅ Handling Arrays
const cities = ['Port of Spain', 'San Fernando'];
const searchParams = new URLSearchParams();

cities.forEach(city => searchParams.append('cities', city));

const url = `/api/cities?${searchParams.toString()}`;

// Results in: 
// /api/cities?cities=Port+of+Spain&cities=San+Fernando
```

## Best Practices for URL Construction

1. **Always use URLSearchParams for query parameters**
2. **Handle special characters properly**
3. **Consider using a URL construction library for complex cases**
4. **Validate parameters before construction**
5. **Use TypeScript for type safety**

### TypeScript Example

```typescript
interface CityParams {
  country: string;
  cities: string[];
  page: number;
  limit: number;
}

function constructCityUrl(params: CityParams): string {
  const searchParams = new URLSearchParams();
  
  searchParams.append('country', params.country);
  params.cities.forEach(city => searchParams.append('cities', city));
  searchParams.append('page', params.page.toString());
  searchParams.append('limit', params.limit.toString());
  
  return `/api/cities?${searchParams.toString()}`;
}
```

## Common Pitfalls to Avoid

1. **Direct string concatenation**
   ```javascript
   // ❌ Don't do this
   const url = '/api/cities?country=' + country + '&page=' + page;
   ```

2. **Template literals for URLs**
   ```javascript
   // ❌ Don't do this
   const url = `/api/cities?country=${country}&page=${page}`;
   ```

3. **Manual encoding**
   ```javascript
   // ❌ Don't do this
   const url = '/api/cities?country=' + encodeURIComponent(country);
   ```

## Conclusion

While template literals are a powerful feature in JavaScript, they should not be used for URL construction. `URLSearchParams` provides a robust, built-in solution that handles all the edge cases and special characters correctly. By following these best practices, you can avoid common pitfalls and create more maintainable, reliable code.

Remember:
- Use `URLSearchParams` for query parameter handling
- Consider TypeScript for type safety
- Validate parameters before construction
- Test edge cases with special characters and arrays

✨ Thank you for reading! If you found this guide helpful, please share it with others. Your feedback and suggestions are always welcome in the comments section.

