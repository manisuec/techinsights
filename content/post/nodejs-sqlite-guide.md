---
layout: post
title: "SQLite Nodejs Guide: Native Module Tutorial & Examples"
description: "Learn to use Nodejs v22+ native SQLite module: setup, prepared statements, custom functions, real-world examples, performance tips, and zero-dependency code snippets."
thumbnail: 'https://res.cloudinary.com/dkiurfsjm/image/upload/v1676698473/nodejs_dark_cjoudy.png'
date: 2025-09-09 00:00:00 +0530
lastmod: 2025-09-09T00:00:30
images: ['https://res.cloudinary.com/dkiurfsjm/image/upload/v1757420416/sqlite-database_l7pn6f.png']
tags: ['nodejs', 'sqlite', 'database']
keywords: 'nodejs,sqlite,database,native module'
categories: ['Nodejs']
url: 'nodejs/nodejs-sqlite-guide'
---

The landscape of Nodejs database integration has evolved significantly with the introduction of the built-in `node:sqlite` module in Nodejs v22.5.0. This native SQLite integration eliminates the need for external dependencies, offering a streamlined, efficient way to work with databases directly within your Nodejs applications.

In this comprehensive guide, we'll explore the capabilities of the native SQLite module, from basic operations to advanced features, complete with practical code examples and real-world use cases.

![sqlite database](https://res.cloudinary.com/dkiurfsjm/image/upload/v1757420416/sqlite-database_l7pn6f.png)

## Why Choose the Native SQLite Module?

Before diving into implementation details, let's understand why the built-in SQLite module is a game-changer:

- **Zero Dependencies**: No external packages required, reducing security vulnerabilities and maintenance overhead
- **Synchronous API**: All operations execute synchronously, simplifying code flow
- **Performance**: Direct integration with SQLite's C API for optimal performance
- **Type Safety**: Built-in type conversion between JavaScript and SQLite data types
- **Advanced Features**: Support for custom functions, aggregates, and database replication

## Getting Started

### Basic Setup

The SQLite module is available under the `node:` scheme, ensuring you're using the official Nodejs implementation:

```javascript
// ES6 Modules
import { DatabaseSync } from 'node:sqlite';

// CommonJS
const { DatabaseSync } = require('node:sqlite');
```

### Creating Your First Database

Let's start with a simple example that demonstrates creating an in-memory database, inserting data, and querying it back:

```javascript
import { DatabaseSync } from 'node:sqlite';

// Create an in-memory database
const database = new DatabaseSync(':memory:');

// Create a table
database.exec(`
  CREATE TABLE users(
    id INTEGER PRIMARY KEY,
    username TEXT NOT NULL,
    email TEXT UNIQUE,
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP
  ) STRICT
`);

// Insert data using prepared statements
const insert = database.prepare('INSERT INTO users (username, email) VALUES (?, ?)');
insert.run('johndoe', 'john@example.com');
insert.run('janedoe', 'jane@example.com');

// Query the data
const query = database.prepare('SELECT * FROM users ORDER BY username');
const users = query.all();
console.log(users);
// Output: [{ id: 1, username: 'johndoe', email: 'john@example.com', created_at: '2024-01-15 10:30:00' }, ...]

// Close the database when done
database.close();
```

## Core Concepts and Features

### Database Connection Options

The `DatabaseSync` constructor offers extensive configuration options:

```javascript
const database = new DatabaseSync('app.db', {
  open: true,                           // Open immediately
  readOnly: false,                      // Read-write mode
  enableForeignKeyConstraints: true,    // Enforce foreign keys
  enableDoubleQuotedStringLiterals: false, // Security best practice
  allowExtension: false,                // Disable extensions for security
  timeout: 5000,                        // 5-second timeout for locks
  readBigInts: false,                   // Use numbers for integers
  returnArrays: false,                  // Return objects, not arrays
  allowBareNamedParameters: true,       // Allow parameter binding without prefix
  allowUnknownNamedParameters: false    // Strict parameter validation
});
```

### Prepared Statements: Your First Line of Defense

Prepared statements are not just about performance; they're crucial for security:

```javascript
// Secure user registration
const registerUser = database.prepare(`
  INSERT INTO users (username, email, password_hash) 
  VALUES (@username, @email, @password_hash)
`);

// Using named parameters
registerUser.run({
  username: 'newuser',
  email: 'user@example.com',
  password_hash: 'hashed_password_here'
});

// Using anonymous parameters
const updateEmail = database.prepare('UPDATE users SET email = ? WHERE id = ?');
updateEmail.run('newemail@example.com', 1);
```

### Advanced Querying Techniques

```javascript
// Complex queries with joins and filtering
const getUserWithPosts = database.prepare(`
  SELECT 
    u.username,
    u.email,
    p.title as post_title,
    p.content as post_content,
    p.created_at as post_date
  FROM users u
  LEFT JOIN posts p ON u.id = p.user_id
  WHERE u.id = ?
  ORDER BY p.created_at DESC
`);

const userData = getUserWithPosts.all(1);
console.log(userData);

// Using iterators for large result sets
const largeQuery = database.prepare('SELECT * FROM large_table');
for (const row of largeQuery.iterate()) {
  // Process each row without loading all into memory
  console.log(row);
}
```

## Real-World Use Cases

### 1. Configuration Management System

```javascript
import { DatabaseSync } from 'node:sqlite';

class ConfigManager {
  constructor(dbPath = 'config.db') {
    this.db = new DatabaseSync(dbPath);
    this.init();
  }

  init() {
    this.db.exec(`
      CREATE TABLE IF NOT EXISTS config (
        key TEXT PRIMARY KEY,
        value TEXT NOT NULL,
        type TEXT CHECK(type IN ('string', 'number', 'boolean', 'json')) DEFAULT 'string',
        updated_at DATETIME DEFAULT CURRENT_TIMESTAMP
      ) STRICT
    `);
  }

  set(key, value, type = 'string') {
    const stmt = this.db.prepare(`
      INSERT OR REPLACE INTO config (key, value, type) 
      VALUES (?, ?, ?)
    `);
    stmt.run(key, JSON.stringify(value), type);
  }

  get(key) {
    const stmt = this.db.prepare('SELECT value, type FROM config WHERE key = ?');
    const result = stmt.get(key);
    
    if (!result) return null;
    
    try {
      return JSON.parse(result.value);
    } catch {
      return result.value;
    }
  }

  getAll() {
    const stmt = this.db.prepare('SELECT key, value, type FROM config');
    const configs = {};
    
    for (const row of stmt.iterate()) {
      try {
        configs[row.key] = JSON.parse(row.value);
      } catch {
        configs[row.key] = row.value;
      }
    }
    
    return configs;
  }

  close() {
    this.db.close();
  }
}

// Usage
const config = new ConfigManager();
config.set('api_key', 'secret123');
config.set('max_users', 1000, 'number');
config.set('features', { premium: true, analytics: false }, 'json');

console.log(config.get('features')); // { premium: true, analytics: false }
```

### 2. Session Store for Express Applications

```javascript
import { DatabaseSync } from 'node:sqlite';
import session from 'express-session';

class SQLiteSessionStore extends session.Store {
  constructor(options = {}) {
    super();
    this.db = new DatabaseSync(options.dbPath || 'sessions.db');
    this.init();
  }

  init() {
    this.db.exec(`
      CREATE TABLE IF NOT EXISTS sessions (
        sid TEXT PRIMARY KEY,
        sess TEXT NOT NULL,
        expire DATETIME NOT NULL
      ) STRICT
    `);
    
    // Clean up expired sessions periodically
    setInterval(() => this.cleanup(), 60 * 60 * 1000); // Every hour
  }

  get(sid, callback) {
    try {
      const stmt = this.db.prepare(`
        SELECT sess FROM sessions 
        WHERE sid = ? AND expire > datetime('now')
      `);
      const result = stmt.get(sid);
      
      if (!result) {
        return callback(null, null);
      }
      
      callback(null, JSON.parse(result.sess));
    } catch (err) {
      callback(err);
    }
  }

  set(sid, sess, callback) {
    try {
      const stmt = this.db.prepare(`
        INSERT OR REPLACE INTO sessions (sid, sess, expire)
        VALUES (?, ?, datetime('now', '+' || ? || ' seconds'))
      `);
      stmt.run(sid, JSON.stringify(sess), sess.cookie?.maxAge / 1000 || 86400);
      callback(null);
    } catch (err) {
      callback(err);
    }
  }

  destroy(sid, callback) {
    try {
      const stmt = this.db.prepare('DELETE FROM sessions WHERE sid = ?');
      stmt.run(sid);
      callback(null);
    } catch (err) {
      callback(err);
    }
  }

  cleanup() {
    const stmt = this.db.prepare("DELETE FROM sessions WHERE expire <= datetime('now')");
    stmt.run();
  }
}

// Express usage
const app = express();
app.use(session({
  store: new SQLiteSessionStore({ dbPath: 'app_sessions.db' }),
  secret: 'your-secret-key',
  resave: false,
  saveUninitialized: false,
  cookie: { maxAge: 24 * 60 * 60 * 1000 } // 24 hours
}));
```

### 3. Analytics and Metrics Collection

```javascript
import { DatabaseSync } from 'node:sqlite';

class AnalyticsCollector {
  constructor(dbPath = 'analytics.db') {
    this.db = new DatabaseSync(dbPath);
    this.init();
  }

  init() {
    this.db.exec(`
      CREATE TABLE IF NOT EXISTS events (
        id INTEGER PRIMARY KEY AUTOINCREMENT,
        event_type TEXT NOT NULL,
        user_id TEXT,
        session_id TEXT,
        properties TEXT,
        timestamp DATETIME DEFAULT CURRENT_TIMESTAMP,
        INDEX idx_event_type (event_type),
        INDEX idx_timestamp (timestamp),
        INDEX idx_user_id (user_id)
      ) STRICT
    `);
  }

  track(eventType, userId = null, sessionId = null, properties = {}) {
    const stmt = this.db.prepare(`
      INSERT INTO events (event_type, user_id, session_id, properties)
      VALUES (?, ?, ?, ?)
    `);
    stmt.run(eventType, userId, sessionId, JSON.stringify(properties));
  }

  getEventCounts(startDate, endDate) {
    const stmt = this.db.prepare(`
      SELECT 
        event_type,
        COUNT(*) as count,
        COUNT(DISTINCT user_id) as unique_users
      FROM events
      WHERE timestamp BETWEEN ? AND ?
      GROUP BY event_type
      ORDER BY count DESC
    `);
    return stmt.all(startDate, endDate);
  }

  getUserActivity(userId, days = 30) {
    const stmt = this.db.prepare(`
      SELECT 
        DATE(timestamp) as date,
        COUNT(*) as event_count,
        COUNT(DISTINCT event_type) as unique_events
      FROM events
      WHERE user_id = ? 
        AND timestamp >= datetime('now', '-' || ? || ' days')
      GROUP BY DATE(timestamp)
      ORDER BY date DESC
    `);
    return stmt.all(userId, days);
  }

  getRetentionCohort(startDate, days = 7) {
    const stmt = this.db.prepare(`
      WITH user_first_seen AS (
        SELECT user_id, MIN(DATE(timestamp)) as first_seen
        FROM events
        WHERE user_id IS NOT NULL
        GROUP BY user_id
      ),
      user_activity AS (
        SELECT 
          user_id,
          DATE(timestamp) as activity_date,
          JULIANDAY(DATE(timestamp)) - JULIANDAY(first_seen) as day_number
        FROM events e
        JOIN user_first_seen ufs ON e.user_id = ufs.user_id
        WHERE user_id IS NOT NULL
      )
      SELECT 
        first_seen as cohort_date,
        day_number,
        COUNT(DISTINCT user_id) as active_users
      FROM user_activity ua
      JOIN user_first_seen ufs ON ua.user_id = ufs.user_id
      WHERE first_seen >= ?
      GROUP BY first_seen, day_number
      ORDER BY first_seen, day_number
    `);
    return stmt.all(startDate);
  }
}

// Usage
const analytics = new AnalyticsCollector();

// Track various events
analytics.track('page_view', 'user123', 'session456', { page: '/home', referrer: 'google' });
analytics.track('button_click', 'user123', 'session456', { button_id: 'subscribe' });
analytics.track('purchase', 'user123', 'session456', { amount: 29.99, product: 'premium' });

// Generate reports
console.log(analytics.getEventCounts('2024-01-01', '2024-01-31'));
console.log(analytics.getUserActivity('user123'));
```

## Advanced Features

### Custom Functions

Extend SQLite with JavaScript functions:

```javascript
const db = new DatabaseSync(':memory:');

// Register a custom function
db.function('calculate_tax', {
  deterministic: true,
  varargs: false
}, (amount, taxRate = 0.08) => {
  return amount * (1 + taxRate);
});

// Use in SQL queries
const stmt = db.prepare('SELECT calculate_tax(100, 0.10) as total');
console.log(stmt.get()); // { total: 110 }

// String manipulation function
db.function('slugify', (text) => {
  return text
    .toLowerCase()
    .replace(/[^\w\s-]/g, '')
    .replace(/[\s_-]+/g, '-')
    .replace(/^-+|-+$/g, '');
});

const stmt2 = db.prepare("SELECT slugify('Hello World!') as slug");
console.log(stmt2.get()); // { slug: 'hello-world' }
```

### Custom Aggregate Functions

```javascript
// Create a custom aggregate for calculating median
db.aggregate('median', {
  start: [],
  step: (acc, value) => {
    acc.push(value);
    return acc;
  },
  result: (acc) => {
    if (acc.length === 0) return null;
    const sorted = acc.sort((a, b) => a - b);
    const mid = Math.floor(sorted.length / 2);
    return sorted.length % 2 === 0 
      ? (sorted[mid - 1] + sorted[mid]) / 2 
      : sorted[mid];
  }
});

// Use the median function
const stmt = db.prepare('SELECT median(salary) FROM employees');
console.log(stmt.get());
```

### Database Backup and Replication

```javascript
import { backup, DatabaseSync } from 'node:sqlite';

async function performBackup() {
  const sourceDb = new DatabaseSync('production.db');
  
  try {
    const totalPages = await backup(sourceDb, 'backup.db', {
      rate: 10, // Copy 10 pages at a time
      progress: ({ totalPages, remainingPages }) => {
        const percentage = ((totalPages - remainingPages) / totalPages * 100).toFixed(2);
        console.log(`Backup progress: ${percentage}%`);
      }
    });
    
    console.log(`Backup completed: ${totalPages} pages transferred`);
  } catch (error) {
    console.error('Backup failed:', error);
  } finally {
    sourceDb.close();
  }
}

// Database replication with changesets
function replicateDatabase() {
  const sourceDb = new DatabaseSync('source.db');
  const targetDb = new DatabaseSync('target.db');
  
  // Create identical schema
  targetDb.exec(sourceDb.exec('SELECT sql FROM sqlite_master WHERE type="table"'));
  
  // Create session for tracking changes
  const session = sourceDb.createSession();
  
  // Make changes to source
  const insert = sourceDb.prepare('INSERT INTO data (key, value) VALUES (?, ?)');
  insert.run(1, 'hello');
  insert.run(2, 'world');
  
  // Get changeset and apply to target
  const changeset = session.changeset();
  targetDb.applyChangeset(changeset);
  
  session.close();
  sourceDb.close();
  targetDb.close();
}
```

## Best Practices and Performance Tips

### 1. Connection Management

```javascript
class DatabasePool {
  constructor(dbPath, maxConnections = 5) {
    this.dbPath = dbPath;
    this.connections = [];
    this.maxConnections = maxConnections;
  }

  getConnection() {
    if (this.connections.length < this.maxConnections) {
      const db = new DatabaseSync(this.dbPath);
      this.connections.push(db);
      return db;
    }
    return this.connections[Math.floor(Math.random() * this.connections.length)];
  }

  closeAll() {
    this.connections.forEach(db => db.close());
    this.connections = [];
  }
}
```

### 2. Transaction Management

```javascript
function withTransaction(db, callback) {
  const isAlreadyInTransaction = db.isTransaction;
  
  if (!isAlreadyInTransaction) {
    db.exec('BEGIN TRANSACTION');
  }
  
  try {
    const result = callback(db);
    if (!isAlreadyInTransaction) {
      db.exec('COMMIT');
    }
    return result;
  } catch (error) {
    if (!isAlreadyInTransaction) {
      db.exec('ROLLBACK');
    }
    throw error;
  }
}

// Usage
withTransaction(database, (db) => {
  const insertUser = db.prepare('INSERT INTO users (username) VALUES (?)');
  const userId = insertUser.run('newuser').lastInsertRowid;
  
  const insertProfile = db.prepare('INSERT INTO profiles (user_id, bio) VALUES (?, ?)');
  insertProfile.run(userId, 'Hello, world!');
  
  return userId;
});
```

### 3. Indexing Strategy

```javascript
// Create indexes based on query patterns
database.exec(`
  -- For frequent lookups by email
  CREATE INDEX IF NOT EXISTS idx_users_email ON users(email);
  
  -- For composite queries
  CREATE INDEX IF NOT EXISTS idx_posts_user_date ON posts(user_id, created_at);
  
  -- For text searches
  CREATE INDEX IF NOT EXISTS idx_posts_title_content ON posts(title, content);
  
  -- For range queries
  CREATE INDEX IF NOT EXISTS idx_orders_date ON orders(order_date);
`);
```

### 4. Error Handling and Logging

```javascript
class RobustDatabase {
  constructor(dbPath) {
    this.db = new DatabaseSync(dbPath);
    this.setupErrorHandling();
  }

  setupErrorHandling() {
    // Log all queries for debugging
    const originalPrepare = this.db.prepare.bind(this.db);
    this.db.prepare = function(sql) {
      console.log(`[SQL] ${sql}`);
      return originalPrepare(sql);
    };
  }

  safeExecute(query, params = []) {
    try {
      const stmt = this.db.prepare(query);
      return stmt.run(...params);
    } catch (error) {
      console.error(`Database error: ${error.message}`);
      console.error(`Query: ${query}`);
      console.error(`Parameters:`, params);
      throw error;
    }
  }
}
```

## Migration and Schema Management

```javascript
class DatabaseMigration {
  constructor(db) {
    this.db = db;
    this.initMigrationTable();
  }

  initMigrationTable() {
    this.db.exec(`
      CREATE TABLE IF NOT EXISTS schema_migrations (
        version TEXT PRIMARY KEY,
        applied_at DATETIME DEFAULT CURRENT_TIMESTAMP
      ) STRICT
    `);
  }

  migrate(migrations) {
    const applied = this.db.prepare('SELECT version FROM schema_migrations').all()
      .map(row => row.version);
    
    for (const [version, sql] of Object.entries(migrations)) {
      if (!applied.includes(version)) {
        console.log(`Applying migration: ${version}`);
        this.db.exec('BEGIN TRANSACTION');
        try {
          this.db.exec(sql);
          this.db.prepare('INSERT INTO schema_migrations (version) VALUES (?)').run(version);
          this.db.exec('COMMIT');
          console.log(`Migration ${version} applied successfully`);
        } catch (error) {
          this.db.exec('ROLLBACK');
          throw new Error(`Migration ${version} failed: ${error.message}`);
        }
      }
    }
  }
}

// Usage
const migrations = {
  '001_create_users': `
    CREATE TABLE users (
      id INTEGER PRIMARY KEY AUTOINCREMENT,
      username TEXT UNIQUE NOT NULL,
      email TEXT UNIQUE NOT NULL,
      created_at DATETIME DEFAULT CURRENT_TIMESTAMP
    );
    CREATE INDEX idx_users_email ON users(email);
  `,
  '002_add_user_profile': `
    ALTER TABLE users ADD COLUMN profile_data TEXT;
  `,
  '003_create_sessions': `
    CREATE TABLE sessions (
      id TEXT PRIMARY KEY,
      user_id INTEGER REFERENCES users(id),
      expires_at DATETIME NOT NULL,
      created_at DATETIME DEFAULT CURRENT_TIMESTAMP
    );
  `
};

const migrator = new DatabaseMigration(database);
migrator.migrate(migrations);
```

## Final Thoughts: SQLite in Nodejs

The native SQLite module in Nodejs represents a significant advancement in database integration, offering:

- **Simplicity**: Direct, synchronous API that integrates seamlessly with Nodejs applications
- **Performance**: Native implementation with minimal overhead
- **Security**: Built-in protection against SQL injection through prepared statements
- **Flexibility**: Support for custom functions, aggregates, and advanced SQLite features
- **Reliability**: No external dependencies mean fewer points of failure

Whether you're building a small CLI tool, a web application, or a complex analytics system, the `node:sqlite` module provides the tools you need to handle data persistence efficiently and securely. The combination of SQLite's reliability and Nodejs's flexibility creates endless possibilities for innovative applications.