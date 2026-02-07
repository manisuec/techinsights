---
layout: post
title: "HTTP Headers: Complete Guide to Secure & Optimize Your APIs"
description: "A definitive guide to HTTP headers; from security headers that block attacks to performance headers that speed up apis;  with production-ready code examples."
thumbnail: 'https://res.cloudinary.com/dkiurfsjm/image/upload/v1676698473/nodejs_dark_cjoudy.png'
date: 2026-01-20 00:00:00 +0000
lastmod: 2026-02-07T00:00:30
images: ['https://res.cloudinary.com/dkiurfsjm/image/upload/v1768909662/http-headers_ctxfaj.png']
tags: ['nodejs', 'security', 'http', 'performance', 'express', 'cors', 'headers']
categories: ['Nodejs']
keywords: 'nodejs,http,headers,security,csp,hsts,cors,cache-control,performance'
url: 'nodejs/http-headers-complete-guide'
---

Running a production API without properly configured HTTP headers is leaving your front door unlocked. Browsers block modern features, CDNs refuse to cache your responses and attackers can exploit XSS, clickjacking and CSRF vulnerabilities in seconds.

This is the definitive guide to HTTP headers; from security headers that stop attacks to performance headers that make your app faster, with production-ready Node.js code examples.

![http headers security](https://res.cloudinary.com/dkiurfsjm/image/upload/v1768909662/http-headers_ctxfaj.png)

## Why HTTP Headers Are Critical?

| Risk                        | Without Proper Headers                     | With Proper Headers                      |
|-----------------------------|-------------------------------------------|------------------------------------------|
| Security                    | XSS, clickjacking, CSRF attacks possible  | CSP, HSTS, COOP block exploitation       |
| Performance                 | Every request hits origin                 | CDN caches, 304 responses save bandwidth |
| Browser Features            | WebAuthn, Service Workers blocked         | Permissions-Policy enables features      |
| CORS Issues                 | Cross-origin API calls fail              | Proper headers enable secure CORS        |
| Cache Efficiency            | Assets re-downloaded on every page load  | Long-lived cache with versioning         |
| Compliance                   | PCI-DSS, GDPR violations                  | Security headers demonstrate compliance   |

**Bottom line:** HTTP headers are not optional; they're the foundation of web security and performance. If your API is missing these headers, you're inviting attacks and wasting resources.

## What Are HTTP Headers?

HTTP headers are key-value pairs sent between clients and servers that pass additional context and metadata about requests and responses. They control security, caching, content negotiation and more.

| Header Type | Location | Purpose | Examples |
|-------------|----------|---------|----------|
| **Request Headers** | Client → Server | Describe the request | User-Agent, Accept, Authorization |
| **Response Headers** | Server → Client | Describe the response | Content-Type, Set-Cookie, Server |
| **Representation Headers** | Both | Describe the body | Content-Type, Content-Encoding |
| **Payload Headers** | Both | Transport info | Content-Length, Transfer-Encoding |

### In Simple Words

**HTTP headers are the metadata packets that travel with your actual data.**

Think of it like sending a package:

HTTP body = the package contents
HTTP headers = the shipping label (address, handling instructions, weight, priority)

Headers tell browsers and intermediaries how to handle, cache, secure and interpret the content they receive.

## Security Headers: Stopping Attacks Before They Happen

### Content-Security-Policy (CSP)

| Header Value | What It Does | Protection Level |
|--------------|--------------|------------------|
| `default-src 'self'` | Only load from same origin | Base protection |
| `script-src 'nonce-random'` | Inline scripts must have nonce | XSS protection |
| `object-src 'none'` | Block plugins (Flash, PDF) | Plugin-based attacks |
| `upgrade-insecure-requests` | Force HTTPS on all resources | MiTM prevention |

```javascript
app.use(helmet.contentSecurityPolicy({
  directives: {
    defaultSrc: ["'self'"],
    scriptSrc: ["'self'", (req, res) => `'nonce-${res.locals.cspNonce}'`],
    styleSrc: ["'self'", "'unsafe-inline'", 'https://fonts.googleapis.com'],
    imgSrc: ["'self'", 'data:', 'https:'],
    connectSrc: ["'self'", 'https://api.example.com'],
    frameAncestors: ["'none'"],
    upgradeInsecureRequests: []
  }
}));
```

**One-liner:**  
> **CSP is your primary defense against XSS attacks.**  
> *Start with strict `'self'` policy and only add exceptions as needed. Test with `Content-Security-Policy-Report-Only` before enforcing.*

### Strict-Transport-Security (HSTS)

| Parameter | Value | Purpose |
|-----------|-------|---------|
| `max-age` | `63072000` (2 years) | Force HTTPS duration |
| `includeSubDomains` | `true` | Apply to all subdomains |
| `preload` | `true` | Add to browser preload list |

```javascript
app.use(helmet.hsts({
  maxAge: 63072000,  // 2 years in seconds
  includeSubDomains: true,
  preload: true
}));
```

**Critical warning:** Never set long `max-age` or `preload` in development; you'll lock yourself out of localhost.

**One-liner:**  
> **HSTS forces HTTPS-only connections for your domain.**  
> *Once your site is fully HTTPS, add this header. Once on preload list, removal is nearly impossible.*

### Cross-Origin Isolation Headers (COOP + COEP)

| Header | Value | Enables |
|--------|-------|---------|
| `Cross-Origin-Opener-Policy` | `same-origin` | Process isolation |
| `Cross-Origin-Embedder-Policy` | `require-corp` | Cross-origin resource control |
| `Cross-Origin-Resource-Policy` | `same-origin` | Resource access control |

```javascript
app.use((req, res, next) => {
  res.setHeader('Cross-Origin-Opener-Policy', 'same-origin');
  res.setHeader('Cross-Origin-Embedder-Policy', 'require-corp');
  next();
});
```

**Required for:** `SharedArrayBuffer`, precise `performance.now()`, high-resolution timers and advanced Web APIs.

### X-Frame-Options

| Value | Behavior |
|-------|----------|
| `DENY` | Block all framing |
| `SAMEORIGIN` | Allow same-site only |
| `ALLOW-FROM uri` | Deprecated, ignored |

```javascript
app.use(helmet.frameguard({ action: 'deny' }));
```

**Note:** This is a legacy header. Prefer CSP's `frame-ancestors` directive, but keep both for browser compatibility.

### Essential Security Header Summary

| Header | Purpose | Recommended Value |
|--------|---------|-------------------|
| `Content-Security-Policy` | XSS protection | `default-src 'self'; object-src 'none'` |
| `Strict-Transport-Security` | HTTPS enforcement | `max-age=63072000; includeSubDomains; preload` |
| `X-Content-Type-Options` | MIME sniffing protection | `nosniff` |
| `X-Frame-Options` | Clickjacking protection | `DENY` |
| `Referrer-Policy` | Privacy | `strict-origin-when-cross-origin` |
| `Permissions-Policy` | Feature control | `geolocation=(), camera=(), microphone=()` |
| `Cross-Origin-Resource-Policy` | Resource protection | `same-origin` |

## CORS Headers: Enabling Secure Cross-Origin Requests

| Header | Purpose | Best Practice |
|--------|---------|---------------|
| `Access-Control-Allow-Origin` | Allowed origins | Exact URL, not `*` |
| `Access-Control-Allow-Methods` | Allowed HTTP methods | Only needed methods |
| `Access-Control-Allow-Headers` | Allowed request headers | Only needed headers |
| `Access-Control-Allow-Credentials` | Enable cookies | `true` only with specific origin |
| `Access-Control-Max-Age` | Preflight cache duration | `86400` (1 day) |

### Production-Grade CORS Configuration

```javascript
import express from 'express';
import cors from 'cors';

const app = express();

const allowedOrigins = [
  'https://app.example.com',
  'https://admin.example.com'
];

app.use(cors({
  origin: (origin, callback) => {
    if (!origin || allowedOrigins.includes(origin)) {
      callback(null, true);
    } else {
      callback(new Error('Not allowed by CORS'));
    }
  },
  credentials: true,
  methods: ['GET', 'POST', 'PUT', 'DELETE', 'PATCH'],
  allowedHeaders: ['Content-Type', 'Authorization', 'X-Request-ID'],
  exposedHeaders: ['X-Total-Count', 'X-RateLimit-Remaining'],
  maxAge: 86400
}));
```

**One-liner:**  
> **Never use `Access-Control-Allow-Origin: *` with `credentials: true`.**  
> *This causes browser errors. Always specify exact origins when using cookies or auth headers.*

### Manual CORS Setup (Without Middleware)

```javascript
app.use((req, res, next) => {
  const origin = req.headers.origin;

  if (allowedOrigins.includes(origin)) {
    res.setHeader('Access-Control-Allow-Origin', origin);
    res.setHeader('Access-Control-Allow-Credentials', 'true');
    res.setHeader('Access-Control-Allow-Methods', 'GET, POST, PUT, DELETE, PATCH, OPTIONS');
    res.setHeader('Access-Control-Allow-Headers', 'Content-Type, Authorization, X-Request-ID');
    res.setHeader('Access-Control-Expose-Headers', 'X-Total-Count, X-RateLimit-Remaining');
    res.setHeader('Access-Control-Max-Age', '86400');
    res.setHeader('Vary', 'Origin');
  }

  if (req.method === 'OPTIONS') {
    res.sendStatus(204);
  } else {
    next();
  }
});
```

## Performance Headers: Caching and Content Delivery

| Header | Use Case | Recommended Value |
|--------|----------|-------------------|
| `Cache-Control` | Asset caching | `public, max-age=31536000, immutable` |
| `ETag` | Cache validation | Content hash (strong) |
| `Last-Modified` | Fallback validator | Timestamp of last change |
| `Vary` | Cache key variation | `Vary: Accept-Encoding` |
| `Content-Encoding` | Compression | `br` (Brotli) or `gzip` |

### Cache-Control Strategies by Content Type

| Content Type | Cache Strategy | Value |
|--------------|----------------|-------|
| Static assets (JS, CSS) | Long-term cache | `public, max-age=31536000, immutable` |
| HTML | Short validation | `public, max-age=0, must-revalidate` |
| API responses | Conditional caching | `public, max-age=60, s-maxage=300` |
| Sensitive data | No cache | `no-store, no-cache, must-revalidate` |

### Production Caching Setup

```javascript
import express from 'express';
import crypto from 'crypto';

const app = express();

// Static assets with long-term caching
app.use(express.static('public', {
  maxAge: '1y',
  etag: true,
  lastModified: true,
  immutable: true
}));

// API responses with conditional caching
app.get('/api/data', (req, res) => {
  res.set('Cache-Control', 'public, max-age=60, s-maxage=300');
  res.set('Vary', 'Accept-Encoding, Origin');
  res.json({ data: 'Cached for 60s (browser), 300s (CDN)' });
});

// Sensitive data - never cache
app.get('/api/user/profile', (req, res) => {
  res.set('Cache-Control', 'no-store, no-cache, must-revalidate');
  res.set('Pragma', 'no-cache');
  res.set('Expires', '0');
  res.json({ user: 'profile data' });
});

// ETag with conditional request handling
app.get('/api/resource/:id', (req, res) => {
  const resource = fetchResource(req.params.id);
  const etag = generateETag(resource);

  res.set('ETag', etag);

  if (req.headers['if-none-match'] === etag) {
    return res.status(304).end();
  }

  res.set('Cache-Control', 'public, max-age=300, must-revalidate');
  res.json(resource);
});

function generateETag(data) {
  const hash = crypto.createHash('sha256').update(JSON.stringify(data)).digest('hex');
  return `"${hash}"`;
}
```

**One-liner:**  
> **Version your static assets (e.g., main.abc123.js) and cache them for a year.**  
> *When content changes, update the hash. This forces browsers to fetch the new version while keeping long cache times.*

### Compression Headers

```javascript
import compression from 'compression';

app.use(compression({
  filter: (req, res) => {
    if (req.headers['x-no-compression']) {
      return false;
    }
    return compression.filter(req, res);
  },
  level: 6,  // Compression level (1-9, higher = more CPU)
  threshold: 1024  // Only compress responses larger than 1KB
}));

app.use((req, res, next) => {
  res.set('Vary', 'Accept-Encoding');
  next();
});
```

## Cookie Security Headers

| Attribute | Purpose | Recommended Value |
|-----------|---------|-------------------|
| `HttpOnly` | Prevent JavaScript access | `true` for session cookies |
| `Secure` | HTTPS only | `true` in production |
| `SameSite` | CSRF protection | `Strict` or `Lax` |
| `Path` | Cookie scope | `/` for entire site |
| `Domain` | Subdomain access | Only if needed |

### Production Cookie Setup

```javascript
import express from 'express';
import cookieParser from 'cookie-parser';

const app = express();
app.use(cookieParser(process.env.COOKIE_SECRET));

// Session cookie - HttpOnly, Secure, SameSite=Strict
app.post('/login', (req, res) => {
  const token = generateSessionToken();

  res.cookie('sessionId', token, {
    httpOnly: true,    // Not accessible via JavaScript (XSS protection)
    secure: process.env.NODE_ENV === 'production',  // HTTPS only
    sameSite: 'strict',  // CSRF protection
    maxAge: 3600000,  // 1 hour
    path: '/'
  });

  res.json({ success: true });
});

// Analytics cookie - non-HttpOnly, SameSite=Lax
app.get('/track', (req, res) => {
  res.cookie('analytics', 'user_123', {
    httpOnly: false,  // Accessible by analytics scripts
    secure: process.env.NODE_ENV === 'production',
    sameSite: 'lax',  // Allows navigation-based cookies
    maxAge: 31536000000  // 1 year
  });

  res.send('OK');
});

// Cross-site API cookie - SameSite=None + Secure
app.post('/api/external', (req, res) => {
  res.cookie('externalToken', 'abc123', {
    httpOnly: true,
    secure: true,  // Required when SameSite=None
    sameSite: 'none',  // Cross-site requests (requires Secure)
    domain: '.example.com',  // Accessible across subdomains
    maxAge: 86400000  // 24 hours
  });

  res.json({ success: true });
});

// Signed cookie - prevents tampering
app.post('/settings', (req, res) => {
  res.cookie('preferences', JSON.stringify(req.body), {
    signed: true,  // Sign to prevent tampering
    httpOnly: true,
    secure: process.env.NODE_ENV === 'production',
    sameSite: 'strict',
    maxAge: 31536000000  // 1 year
  });

  res.json({ success: true });
});

app.get('/settings', (req, res) => {
  const preferences = req.signedCookies.preferences;
  res.json(preferences);
});
```

**One-liner:**  
> **Always set `HttpOnly` on session cookies to block XSS attacks.**  
> *Never send sensitive cookies over non-HTTPS connections. In production, `Secure` is non-negotiable.*

## Complete Production-Ready Setup

```javascript
// src/server.ts
import express, { Request, Response, NextFunction } from 'express';
import helmet from 'helmet';
import cors from 'cors';
import compression from 'compression';
import cookieParser from 'cookie-parser';

const app = express();

// Trust proxy (behind nginx/cloudflare)
app.set('trust proxy', true);

// Security headers
app.use(helmet({
  contentSecurityPolicy: {
    directives: {
      defaultSrc: ["'self'"],
      scriptSrc: ["'self'", (req, res) => `'nonce-${res.locals.cspNonce}'`],
      styleSrc: ["'self'", "'unsafe-inline'", 'https://fonts.googleapis.com'],
      imgSrc: ["'self'", 'data:', 'https:'],
      connectSrc: ["'self'", 'https://api.example.com'],
      frameAncestors: ["'none'"],
      objectSrc: ["'none'"],
      upgradeInsecureRequests: []
    }
  },
  hsts: {
    maxAge: 63072000,
    includeSubDomains: true,
    preload: true
  },
  frameguard: { action: 'deny' },
  noSniff: true,
  referrerPolicy: { policy: 'strict-origin-when-cross-origin' }
}));

// Cross-origin isolation
app.use((req: Request, res: Response, next: NextFunction) => {
  res.setHeader('Cross-Origin-Opener-Policy', 'same-origin');
  res.setHeader('Cross-Origin-Embedder-Policy', 'require-corp');
  res.setHeader('Cross-Origin-Resource-Policy', 'same-origin');
  next();
});

// CORS
app.use(cors({
  origin: ['https://app.example.com', 'https://admin.example.com'],
  credentials: true,
  methods: ['GET', 'POST', 'PUT', 'DELETE', 'PATCH', 'OPTIONS'],
  allowedHeaders: ['Content-Type', 'Authorization', 'X-Request-ID'],
  exposedHeaders: ['X-Total-Count', 'X-RateLimit-Remaining'],
  maxAge: 86400
}));

// Compression
app.use(compression({ level: 6, threshold: 1024 }));

// Cookies
app.use(cookieParser(process.env.COOKIE_SECRET));

// Static assets with long cache
app.use(express.static('public', {
  maxAge: '1y',
  etag: true,
  lastModified: true,
  immutable: true
}));

// Security middleware for all responses
app.use((req: Request, res: Response, next: NextFunction) => {
  res.removeHeader('X-Powered-By');
  res.set('X-Content-Type-Options', 'nosniff');
  res.set('X-Frame-Options', 'DENY');
  res.set('Vary', 'Accept-Encoding, Origin');
  next();
});

// Rate limiting headers
app.use((req: Request, res: Response, next: NextFunction) => {
  res.set('X-RateLimit-Limit', '1000');
  res.set('X-RateLimit-Remaining', '999');
  next();
});

app.get('/', (_req: Request, res: Response) => {
  res.json({ message: 'Headers configured correctly' });
});

app.listen(4000, () => {
  console.log('Server on :4000 with all security headers');
});
```

## Common Pitfalls and How to Avoid Them

| Pitfall | Why It's Bad | Fix |
|---------|--------------|-----|
| `Access-Control-Allow-Origin: *` with credentials | Browser blocks requests | Use specific origin URL |
| Missing `Vary: Origin` with dynamic CORS | CDN serves wrong response | Add `Vary: Origin` header |
| `unsafe-inline` in CSP | Defeats XSS protection | Use nonce or hash-based CSP |
| Long HSTS max-age in dev | Locks you out of localhost | Only set in production |
| Cache-Control on sensitive data | Data leaks via shared cache | Use `no-store, no-cache` |
| SameSite=None without Secure | Browser rejects cookie | Always use with Secure flag |
| Missing ETag on dynamic content | Inefficient revalidation | Generate content-based ETags |
| `X-Powered-By: Express` | Reveals tech stack | Remove with helmet.hidePoweredBy() |

## Deployment Checklist (Before You Go Live)

- [ ] **Security Headers**: CSP, HSTS, X-Frame-Options, X-Content-Type-Options, Referrer-Policy
- [ ] **Cross-Origin Isolation**: COOP + COEP if using SharedArrayBuffer
- [ ] **CORS**: Specific origins (not `*`), proper methods/headers, `Vary: Origin`
- [ ] **Caching**: Cache-Control by content type, ETags for validation
- [ ] **Compression**: Brotli or gzip enabled, `Vary: Accept-Encoding`
- [ ] **Cookies**: HttpOnly + Secure + SameSite on all auth cookies
- [ ] **HTTPS**: SSL/TLS 1.2+, HSTS enabled
- [ ] **Testing**: Test with https://securityheaders.com and https://observatory.mozilla.org
- [ ] **Monitoring**: Log CSP violations and CORS errors

## Final Thoughts

In today's world, "We'll configure headers later" is no longer acceptable.

- Modern browsers require security headers for advanced features
- Proper caching dramatically reduces server load and latency
- CORS misconfigurations silently break cross-origin functionality
- Header-based attacks are trivial to exploit without proper defenses

Configure your headers today. Your security team, your users and your performance metrics will thank you.
