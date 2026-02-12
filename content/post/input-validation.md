---
layout: post
title: "Ship Safer Nodejs APIs: Validate & Sanitize (Joi vs Zod)"
description: "Secure Node.js APIs: Master Input Validation & Sanitization with Zod, Joi & class-validator (Defense Against Injection & Flaws)."
thumbnail: 'https://res.cloudinary.com/dkiurfsjm/image/upload/v1676698473/nodejs_dark_cjoudy.png'
date: 2025-11-04 00:00:00 +0530
lastmod: 2026-02-07T00:00:30
images: ['https://res.cloudinary.com/dkiurfsjm/image/upload/v1762253900/input-validation_f0lire.jpg']
tags: ['nodejs', 'security', 'validation', 'zod', 'joi', 'express', 'typescript']
categories: ['Nodejs']
keywords: 'nodejs,security,input validation,zod,joi,express,typescript,sanitization'
url: 'nodejs/input-validation-and-sanitization-joi-vs-zod'
---

Input validation isn't just checking types; it's your first line of defense against injection attacks, data corruption, and logic flaws. Here's how to implement it properly.

## Why Validation Matters in Production
In production systems, invalid input causes more than just 400 errors. It leads to:

- NoSQL/SQL injection exposing your database
- XSS attacks compromising your users
- Prototype pollution hijacking your application logic
- Data corruption from malformed payloads
- Business logic bypasses that break your core functionality

I'll show you how to build a validation layer that stops these threats while keeping your code clean and TypeScript-friendly.

For a broader security posture across your stack, see the companion guide: [Nodejs Security Checklist](/nodejs/nodejs-security-checklist/).

## Schema Validators Compared

### Joi: The Battle-Tested Veteran

```javascript
const Joi = require('joi');

const userSchema = Joi.object({
  email: Joi.string().email().required(),
  password: Joi.string().min(8).pattern(/^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)/),
  age: Joi.number().integer().min(13).max(120),
  role: Joi.string().valid('user', 'admin').default('user')
});

// Custom sanitization
const sanitizeUser = Joi.object({
  username: Joi.string().trim().replace(/<script\b[^<]*(?:(?!<\\/script>)<[^<]*)*<\\/script>/gi, ''),
  bio: Joi.string().max(500) // implement custom extension like stripTags if needed
});
```

If you maintain Joi schemas, you can auto-generate API docs from them: [Generating API Documentation with Swagger using joi-to-swagger](/nodejs/generate-api-docs-swagger-joi/).

***Best for:** REST APIs, form data, complex validation rules. Mature ecosystem with MongoDB/ObjectId validation built-in.*

### Zod: TypeScript-Native Validation

```typescript
import { z } from 'zod';

const UserSchema = z.object({
  email: z.string().email(),
  password: z.string().min(8).regex(/^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)/),
  age: z.number().int().min(13).max(120),
  role: z.enum(['user', 'admin']).default('user')
});

// Infer TypeScript type automatically
type User = z.infer<typeof UserSchema>;

// Strict mode for prototype pollution protection
const StrictSchema = z.object({}).strict();
```

***Best for:** TypeScript projects, frontend-backend type sharing, when you want automatic type inference.*

### class-validator: Decorator-Based Approach

```typescript
import { IsEmail, IsString, MinLength, IsInt, Min, Max, IsIn } from 'class-validator';

class CreateUserDto {
  @IsEmail()
  email: string;

  @IsString()
  @MinLength(8)
  password: string;

  @IsInt()
  @Min(13)
  @Max(120)
  age: number;

  @IsIn(['user', 'admin'])
  role: string = 'user';
}
```

***Best for:** Enterprise applications, NestJS, teams comfortable with decorators and classes.*

### Quick Decision Guide

| Need | Choose |
| --- | --- |
| TypeScript integration | Zod |
| Complex validation rules | Joi |
| NestJS/decorator patterns | class-validator |
| MongoDB-focused projects | Joi (better ObjectId support) |
| Shared frontend/backend types | Zod |

Working with MongoDB? Pair request validation with schema rules: [Mongoose Schema Validation: Best Practices](/mongodb/mongoose-schema-validation/).

## Centralized Validation Middleware
Create a reusable validation layer that handles errors consistently:

```typescript
// middleware/validation.js
const { ZodError } = require('zod');

const validate = (schema) => (req, res, next) => {
  try {
    const result = schema.parse({
      body: req.body,
      query: req.query,
      params: req.params
    });
    
    // Replace input with validated data
    req.body = result.body || {};
    req.query = result.query || {};
    req.params = result.params || {};
    
    next();
  } catch (error) {
    if (error instanceof ZodError) {
      const errors = error.errors.map(err => ({
        path: err.path.join('.'),
        message: err.message
      }));
      
      return res.status(400).json({
        error: 'Validation failed',
        details: errors
      });
    }
    
    next(error);
  }
};

module.exports = validate;
```

For more reusable patterns, see [Express.js Middleware Patterns](/nodejs/nodejs-expressjs-middlewares/) and production-grade [Node.js Error Handling](/nodejs/error-handling-nodejs/).

## Complete Authentication Route Example
Here's a production-ready auth route with comprehensive validation:

```typescript
// routes/auth.js
const express = require('express');
const { z } = require('zod');
const bcrypt = require('bcryptjs');
const validate = require('../middleware/validation');

const router = express.Router();

// Auth schemas
const LoginSchema = z.object({
  body: z.object({
    email: z.string().email('Valid email required'),
    password: z.string().min(1, 'Password required'),
    // Prevent massive payloads
    _: z.undefined().optional() 
  }).strict()
});

const RegisterSchema = z.object({
  body: z.object({
    email: z.string().email().max(254),
    password: z.string()
      .min(8, 'Password must be 8+ characters')
      .regex(/[a-z]/, 'Password needs lowercase letter')
      .regex(/[A-Z]/, 'Password needs uppercase letter')  
      .regex(/\d/, 'Password needs number')
      .max(100), // Prevent DoS via massive inputs
    name: z.string().min(1).max(100).trim(),
    age: z.number().int().min(13).max(120).optional()
  }).strict()
});

// Routes
router.post('/login', validate(LoginSchema), async (req, res) => {
  const { email, password } = req.body;
  
  try {
    // Your auth logic here
    const user = await findUserByEmail(email);
    if (!user || !await bcrypt.compare(password, user.passwordHash)) {
      return res.status(401).json({ error: 'Invalid credentials' });
    }
    
    // Return safe user data (no password hash)
    const safeUser = {
      id: user.id,
      email: user.email,
      name: user.name
    };
    
    res.json({ user: safeUser, token: generateToken(user) });
  } catch (error) {
    res.status(500).json({ error: 'Authentication failed' });
  }
});

router.post('/register', validate(RegisterSchema), async (req, res) => {
  const { email, password, name, age } = req.body;
  
  try {
    // Check for existing user
    if (await findUserByEmail(email)) {
      return res.status(409).json({ error: 'User already exists' });
    }
    
    // Hash password
    const passwordHash = await bcrypt.hash(password, 12);
    
    // Create user
    const user = await createUser({
      email: email.toLowerCase().trim(), // Sanitization
      passwordHash,
      name: name.trim(),
      age,
      role: 'user'
    });
    
    res.status(201).json({ 
      user: { id: user.id, email: user.email, name: user.name },
      token: generateToken(user)
    });
  } catch (error) {
    res.status(500).json({ error: 'Registration failed' });
  }
});

module.exports = router;
```

This route composes middleware; explore additional patterns in [Express.js Middleware Patterns](/nodejs/nodejs-expressjs-middlewares/).

## Preventing Common Vulnerabilities

### 1. NoSQL Injection Protection

Deep dive into database-side defenses: [MongoDB Security Best Practices](/mongodb/mongodb-security-best-practices/).

```javascript
// middleware/mongo-sanitize.js
const sanitizeMongo = (req, res, next) => {
  const sanitize = (obj) => {
    if (obj && typeof obj === 'object') {
      // Remove MongoDB operators
      Object.keys(obj).forEach(key => {
        if (key.startsWith('$')) {
          delete obj[key];
        } else {
          sanitize(obj[key]);
        }
      });
    }
    return obj;
  };
  
  req.body = sanitize(req.body);
  req.query = sanitize(req.query);
  next();
};

// In your routes - use explicit queries
const { z } = require('zod');
const safeUserQuery = z.object({
  query: z.object({
    email: z.string().email().optional(),
    role: z.enum(['user', 'admin']).optional(),
    // Never accept arbitrary where clauses
    limit: z.number().int().min(1).max(100).default(10),
    offset: z.number().int().min(0).default(0)
  })
});

router.get('/users', validate(safeUserQuery), async (req, res) => {
  const { email, role, limit, offset } = req.query;
  
  // Safe query construction
  const filter = {};
  if (email) filter.email = email;
  if (role) filter.role = role;
  
  const users = await User.find(filter)
    .limit(limit)
    .skip(offset)
    .select('-passwordHash'); // Exclude sensitive fields
    
  res.json(users);
});
```

### 2. XSS Prevention with Output Encoding

```javascript
// middleware/xss-sanitize.js
const createDOMPurify = require('dompurify');
const { JSDOM } = require('jsdom');

const window = new JSDOM('').window;
const DOMPurify = createDOMPurify(window);

const deepMap = (value, fn) => {
  if (Array.isArray(value)) return value.map(v => deepMap(v, fn));
  if (value && typeof value === 'object') {
    return Object.fromEntries(Object.entries(value).map(([k, v]) => [k, deepMap(v, fn)]));
  }
  return fn(value);
};

const xssSanitize = (req, res, next) => {
  const sanitize = (data) => {
    if (typeof data === 'string') {
      return DOMPurify.sanitize(data, {
        ALLOWED_TAGS: [], // Strip all HTML
        ALLOWED_ATTR: []  // Strip all attributes
      });
    }
    return data;
  };
  
  // Apply to body and query params
  if (req.body) req.body = deepMap(req.body, sanitize);
  if (req.query) req.query = deepMap(req.query, sanitize);
  
  next();
};

// In your user profile update
const { z } = require('zod');
const ProfileSchema = z.object({
  body: z.object({
    bio: z.string().max(500).transform(bio => 
      DOMPurify.sanitize(bio, { 
        ALLOWED_TAGS: ['br', 'em', 'strong'], 
        ALLOWED_ATTR: [] 
      })
    ),
    website: z.string().url().optional()
      .transform(url => url ? encodeURI(url) : url)
  }).strict()
});
```

### 3. Prototype Pollution Protection

Prototype pollution is a critical class of vulnerabilities; see the prevention items in the [Nodejs Security Checklist](/nodejs/nodejs-security-checklist/).

```javascript
// middleware/prototype-pollution.js
const blockPrototypePollution = (req, res, next) => {
  const isDangerous = (key) => 
    key === '__proto__' || key === 'constructor' || key === 'prototype';
  
  const sanitize = (obj) => {
    if (obj && typeof obj === 'object') {
      Object.keys(obj).forEach(key => {
        if (isDangerous(key)) {
          delete obj[key];
          return;
        }
        sanitize(obj[key]);
      });
    }
    return obj;
  };
  
  req.body = sanitize(req.body);
  req.query = sanitize(req.query);
  req.params = sanitize(req.params);
  next();
};

// In Zod schemas, use .strict() to prevent extra properties
const { z } = require('zod');
const SafeSchema = z.object({
  body: z.object({}).strict() // No extra properties allowed
});
```

## Allowlists vs Denylists: Always Choose Allowlists

❌ Denylist approach (fragile):

```javascript
// This will always have gaps
const badWords = ['script', 'javascript', 'onload'];
const sanitized = input.replace(new RegExp(badWords.join('|'), 'gi'), '');
```

✅ Allowlist approach (secure):

```javascript
// Only allow what you need
const usernameSchema = z.string()
  .min(3).max(30)
  .regex(/^[a-zA-Z0-9_-]+$/) // Only these characters allowed
  .transform(s => s.toLowerCase().trim());

const htmlSchema = z.string().transform(html => 
  DOMPurify.sanitize(html, {
    ALLOWED_TAGS: ['p', 'br', 'strong', 'em', 'ul', 'li', 'a'],
    ALLOWED_ATTR: ['href'],
    ALLOWED_URI_REGEXP: /^(https?|mailto):/
  })
);
```

## TypeScript Request Typing
Get full type safety from request to response:

```typescript
// types/express.d.ts
import { Request } from 'express';
import { z } from 'zod';
import { UserSchema } from '../schemas/user';

declare global {
  namespace Express {
    interface Request {
      validatedData?: z.infer<typeof UserSchema>['body'];
    }
  }
}

// middleware/validation.ts
import { Response, NextFunction } from 'express';
import { AnyZodObject, ZodError } from 'zod';

export const validate = (schema: AnyZodObject) => 
  async (req: Request, res: Response, next: NextFunction) => {
    try {
      const result = await schema.parseAsync({
        body: req.body,
        query: req.query,
        params: req.params,
      });
      
      req.body = result.body;
      req.query = result.query;
      req.params = result.params;
      
      // For TypeScript users who want typed access
      req.validatedData = result.body;
      
      next();
    } catch (error) {
      if (error instanceof ZodError) {
        return res.status(400).json({
          error: 'Validation failed',
          details: error.errors
        });
      }
      next(error);
    }
  };

// In your controller - full type safety!
router.post('/users', validate(UserSchema), (req: Request, res: Response) => {
  // req.body is fully typed based on UserSchema
  const { email, name, age } = req.body;
  
  // Or use the validatedData for extra certainty
  const userData = req.validatedData;
});
```

## Performance Considerations
Validation doesn't have to be slow:

```javascript
// middleware/performance-optimized.js
const { memoize } = require('lodash');
const Joi = require('joi');

// Cache schema compilation (useful for Joi)
const compileJoiSchema = memoize(Joi.compile);

// Use fast-json-parse for initial JSON validation
const fastJsonParse = require('fast-json-parse');

const validateJsonBody = (req, res, next) => {
  if (req.is('application/json') && typeof req.body === 'string') {
    const result = fastJsonParse(req.body);
    if (result.err) {
      return res.status(400).json({ error: 'Invalid JSON' });
    }
    req.body = result.value;
  }
  next();
};

// Batch validation for arrays
const { z } = require('zod');
const BatchUserSchema = z.array(UserSchema).max(100); // Prevent massive arrays
```

## Validation Checklist for CI/CD
Add this to your project's `VALIDATION_CHECKLIST.md`:

✅ Input Validation

- All user input validated with schema validators (Joi/Zod/class-validator)
- Request body, query params, and route params validated
- TypeScript types generated from validation schemas
- Custom validation for business rules (email uniqueness, etc.)
- Array length limits to prevent resource exhaustion
- File upload validation (type, size, name)

✅ Security Protections

- NoSQL injection prevention (strip $ operators)
- XSS prevention (HTML sanitization on storage/display)
- Prototype pollution protection
- SQL injection prevention (use parameterized queries)
- Command injection prevention (validate before shell execution)
- Directory traversal prevention (sanitize file paths)

✅ Data Sanitization

- Trim whitespace from text inputs
- Convert emails to lowercase
- Sanitize HTML content (allowlist approach)
- Normalize phone numbers/addresses
- Encode URLs before storage/use

✅ Error Handling

- Generic error messages (don't reveal validation rules)
- Consistent error response format
- Input validation errors logged for monitoring
- No stack traces in production errors

Strengthen this layer with [Node.js Error Handling: Strategies for Production Applications](/nodejs/error-handling-nodejs/) and wire it into your telemetry per [Monitoring Tips for Nodejs Applications](/nodejs/monitoring-nodejs-applications/).

✅ Performance & Maintenance

- Validation schemas centralized and reusable
- Schema compilation cached where possible
- Validation doesn't block event loop
- Regular review of validation rules
- Automated tests for validation logic

## Key Takeaways

- Validate early, validate often ;  check input at the API boundary
- Use allowlists, not denylists ;  only allow what you need
- Schema validation > manual checks ;  consistency prevents gaps
- TypeScript + validation = safety ;  get both runtime and compile-time checking
- Sanitize based on context ;  HTML sanitization differs from SQL escaping
- Centralize your validation ;  consistent approach across all routes

## Final Thoughts

Your validation layer should be like a bouncer at a club: it checks IDs at the door, doesn't let suspicious characters in, and makes sure everyone inside behaves properly. Implement this approach and you'll stop entire classes of vulnerabilities before they reach your business logic.

Validation is not just about checking types; it's about protecting your application from injection attacks, data corruption, and logic flaws. By implementing a centralized validation layer, you can stop entire classes of vulnerabilities before they reach your business logic.

## References

- [Joi](https://joi.dev/)
- [Zod](https://zod.dev/)
- [class-validator](https://github.com/typestack/class-validator)