---
layout: post
title: "Nodejs Security Checklist To Prevent Common Vulnerabilities"
description: "Stop SQL injection, prototype pollution & supply-chain attacks. Copy-paste code, 12-step checklist, and CI/CD automation for bullet-proof Nodejs apps in 2025."
thumbnail: 'https://res.cloudinary.com/dkiurfsjm/image/upload/v1676698473/nodejs_dark_cjoudy.png'
date: 2025-09-05 00:00:00 +0530
lastmod: 2025-09-05T00:00:30
images: ['https://res.cloudinary.com/dkiurfsjm/image/upload/v1757079951/nodejs-security-checklist_ny71ku.jpg']
tags: ['nodejs', 'javascript', 'security']
keywords: 'nodejs,javascript,security,security checklist'
categories: ['Nodejs']
url: 'nodejs/nodejs-security-checklist'
---

“One forgotten `eval()` sank a fintech’s $2 M seed round. Let’s make sure it doesn’t happen to you.”

Nodejs makes shipping features fast; sometimes *too* fast. In 2025, public CVE data shows **SQL/NoSQL injection** is still the #1 flaw in Nodejs applications, followed closely by **prototype pollution** and **malicious dependencies**. Below you’ll find runnable code, real-world fixes, and a copy-paste checklist so you can push secure code without slowing CI/CD.

![nodejs security checklist](https://res.cloudinary.com/dkiurfsjm/image/upload/v1757079951/nodejs-security-checklist_ny71ku.jpg)


## Why Nodejs Apps Get Pwned in 2025

*Dependency sprawl*: the average Express project pulls 1 200+ packages.  
*Event-loop blocking*: a single ReDoS regex can spin CPU to 100 %.  
*Typosquatting*: packages like `electorn` (note the typo) still rack up 20 k weekly downloads before takedown.


## The 7 Deadly Vulnerabilities & Code-Level Fixes

### SQL/NoSQL Injection

**Bad**  
```js
const query = `SELECT * FROM users WHERE id = ${req.params.id}`;
```

**Good**  
```js
const text = 'SELECT * FROM users WHERE id = $1';
const values = [req.params.id];
await pool.query(text, values);
```
Use **parameterized queries** (`pg`, `mysql2`, `mongodb` all support them). Validate IDs with `Joi.number().integer().positive()`.


### Prototype Pollution

**Vulnerable**  
```js
const _ = require('lodash');
_.merge({}, JSON.parse(req.body.json)); // user input
```

**Fixed**  
```js
const cloned = structuredClone(JSON.parse(req.body.json)); // or
_.merge({}, JSON.parse(req.body.json), (objValue, srcValue) => {
  if (_.isObject(objValue) && objValue.constructor !== Object) return objValue;
});
```
Add `Object.freeze(Object.prototype)` during bootstrap to prevent prototype tampering.


### XSS & CSRF

**XSS fix**  
```js
const DOMPurify = require('isomorphic-dompurify');
const safeHTML = DOMPurify.sanitize(userInput);
```

**CSRF fix**  
```js
const csrf = require('csurf');
app.use(csrf());
```
Send token in `res.locals._csrf` and validate on every state-changing request.


### ReDoS & DoS

**ReDoS-safe regex**  
```js
const safeRegex = require('safe-regex');
if (!safeRegex(userRegex)) throw new Error('Dangerous pattern');
```

**Rate-limiting**  
```js
const rateLimit = require('express-rate-limit');
app.use('/login', rateLimit({ windowMs: 15 * 60 * 1000, max: 5 }));
```


### Malicious Dependencies

1. `npm config set ignore-scripts true`  
2. `npx lockfile-lint --path package-lock.json --allowed-hosts npm`  
3. `npm audit --audit-level moderate` in CI.


### Sensitive Data Exposure

- Store secrets in `.env`, **never** commit to Git.  
- Hash passwords with `bcrypt.hash(password, 12)`.  
- Add Helmet in one line:  
```js
const helmet = require('helmet');
app.use(helmet());
```

Headers added include `Strict-Transport-Security`, `X-Content-Type-Options`, `Content-Security-Policy`, etc.


### SSRF & DNS Rebinding

**Allow-list IPs**  
```js
const ip = require('ip');
if (!ip.cidrSubnet('10.0.0.0/8').contains(targetIP)) throw new Error('Forbidden');
```

Disable inspector in prod: `node --inspect=0.0.0.0:0` is **never** safe.


## Nodejs Security Checklist 2025 (Copy-Paste Ready)

| # | Task | CLI / Snippet |
|---|---|---|
| 1 | Use Nodejs 20+ | `nvm use 20` |
| 2 | Enable `--experimental-permission` | `node --experimental-permission --allow-fs-read=* index.js` |
| 3 | Parameterize all queries | `pool.query('SELECT * FROM users WHERE id = $1', [id])` |
| 4 | Freeze prototypes | `Object.freeze(Object.prototype)` |
| 5 | Add Helmet | `app.use(helmet())` |
| 6 | Rate-limit sensitive routes | `express-rate-limit` |
| 7 | Audit deps | `npm audit --audit-level moderate` |
| 8 | Lint lockfile | `npx lockfile-lint` |
| 9 | Sanitize HTML | `DOMPurify.sanitize()` |
|10 | Hash passwords | `bcrypt.hash(password, 12)` |
|11 | Validate env | `require('dotenv-safe').config()` |
|12 | Secure CI | `npm ci --ignore-scripts` |

Save as `SECURITY.md` in repo root.

## Tooling Deep-Dive

| Tool | Type | Example |
|---|---|---|
| **eslint-plugin-security** | Static linting | `npm i -D eslint-plugin-security` <br> `"extends": ["plugin:security/recommended"]` |
| **Snyk Code** | SAST | `npx snyk test` |
| **CodeQL** | GitHub-native | `.github/workflows/codeql.yml` |

## Helmet vs express-rate-limit

| Concern | Helmet | express-rate-limit |
|---|---|---|
| XSS | ✔ CSP header | ✖ |
| Brute-force login | ✖ | ✔ |
| ReDoS mitigation | ✖ | ✔ |
| Lines of code | 1 | 3-5 |

Use **both**; they’re complementary.

## Automating Security in CI/CD

`.github/workflows/security.yml`
```yml
name: Security
on: [push]
jobs:
  audit:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 20
      - run: npm ci --ignore-scripts
      - run: npm audit --audit-level moderate
      - run: npx lockfile-lint --path package-lock.json
```

## FAQ

**Q: How to prevent SQL injection in Nodejs?**  
A: Always use **parameterized queries** (`$1`, `$2` placeholders) and validate inputs with `Joi` or `zod`.

**Q: Nodejs prototype pollution fix with example?**  
A: Freeze `Object.prototype` on startup and sanitize deep merges as shown above.

**Q: Best Nodejs security linter?**  
A: **eslint-plugin-security** for quick local checks; **Snyk Code** or **CodeQL** for deeper SAST in CI.


## Conclusion & Next Step

Security doesn’t have to be a drag. Add the checklist to your repo, drop Helmet & rate-limiting into your app today, and let CI fail builds that introduce new vulns.

---

### Sources
- OWASP Top 10 2021  
- Nodejs Security WG CVE Reports 2023-24  
- Snyk State of Open Source Security 2025

✨ Thank you for reading and I hope you find it helpful. I sincerely request for your feedback in the comment's section.
