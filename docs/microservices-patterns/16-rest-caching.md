# Event Driven Systems: REST Caching Strategies

## Table of Contents
1. [What is Caching](#what-is-caching)
2. [Caching Strategies in REST APIs](#caching-strategies-in-rest-apis)
   - [Application Layer Caching](#application-layer-caching)
   - [Request Level Caching](#request-level-caching)
   - [Conditional Caching](#conditional-caching)
3. [Cache Invalidation](#cache-invalidation)
   - [Write-Through](#write-through)
   - [Write-Behind](#write-behind)
   - [TTL-Based](#ttl-based)

---

## What is Caching

- Caching is a technique of storing frequently accessed data in a fast-access storage layer to reduce latency, decrease database load, and improve overall system performance. 

- In REST APIs, caching acts as an intermediary between clients and servers, serving previously computed or fetched data instead of recalculating or re-fetching it from the source.

### Key Benefits of Caching:
- **Reduced Latency**: Serve data from memory instead of disk or network
- **Lower Database Load**: Fewer queries reach the database
- **Improved Throughput**: Handle more concurrent requests
- **Cost Efficiency**: Reduce infrastructure costs by serving cached data
- **Better User Experience**: Faster response times

### Caching Layers in REST APIs:
1. **Client-side Caching**: Browser cache, local storage
2. **CDN Caching**: Geographic distribution of cached content
3. **Application Layer Caching**: In-memory caches like Redis
4. **Database Query Caching**: Query result caching

---

## Caching Strategies in REST APIs

### Application Layer Caching

Application layer caching stores data in memory within your application or a dedicated cache store like Redis. This is the most common caching strategy in microservices architectures.

![image](../../Resources/16-rest-caching/Screenshot%202026-02-07%20at%2012.37.03 PM.png)

#### Where and How to Use:
- Cache frequently accessed database queries
- Store computed results that are expensive to calculate
- Cache API responses from external services
- Store user session data
- Cache configuration data that changes infrequently

#### Pros:
- ✅ Fast access to cached data (microseconds)
- ✅ Full control over cache logic and invalidation
- ✅ Can cache complex objects and computed results
- ✅ Works across multiple requests and users
- ✅ Reduces database load significantly

#### Cons:
- ❌ Requires additional infrastructure (Redis, Memcached)
- ❌ Memory overhead for storing data
- ❌ Cache coherency issues in distributed systems
- ❌ Stale data risks if invalidation is not handled properly
- ❌ Complexity in managing cache lifecycle

#### Code Examples:

**NestJS Example:**
```typescript
import { Injectable } from '@nestjs/common';
import { RedisService } from '@nestjs-modules/ioredis';

@Injectable()
export class UserService {
  constructor(private redis: RedisService) {}

  async getUserById(userId: string) {
    // Try to get from cache first
    const cacheKey = `user:${userId}`;
    const cachedUser = await this.redis.get(cacheKey);
    
    if (cachedUser) {
      return JSON.parse(cachedUser);
    }

    // If not in cache, fetch from database
    const user = await this.database.users.findById(userId);
    
    // Store in cache for 1 hour (3600 seconds)
    await this.redis.setex(cacheKey, 3600, JSON.stringify(user));
    
    return user;
  }

  async updateUser(userId: string, data: any) {
    const user = await this.database.users.update(userId, data);
    
    // Invalidate cache after update
    const cacheKey = `user:${userId}`;
    await this.redis.del(cacheKey);
    
    return user;
  }
}
```

**Python Example:**
```python
import redis
import json
from typing import Optional

class UserService:
    def __init__(self):
        self.redis_client = redis.Redis(host='localhost', port=6379, db=0)
        self.db = DatabaseConnection()
    
    def get_user(self, user_id: str) -> Optional[dict]:
        cache_key = f"user:{user_id}"
        
        # Try cache first
        cached_user = self.redis_client.get(cache_key)
        if cached_user:
            return json.loads(cached_user)
        
        # Fetch from database
        user = self.db.query(f"SELECT * FROM users WHERE id = {user_id}")
        
        # Store in cache for 1 hour
        self.redis_client.setex(cache_key, 3600, json.dumps(user))
        
        return user
    
    def update_user(self, user_id: str, data: dict) -> dict:
        user = self.db.update(f"users", user_id, data)
        
        # Invalidate cache
        cache_key = f"user:{user_id}"
        self.redis_client.delete(cache_key)
        
        return user
```

#### Tiny Example:
```
Request 1: GET /api/users/123
  → Cache miss → Query database → Return user + cache result
  → Response time: 150ms

Request 2: GET /api/users/123 (within 1 hour)
  → Cache hit → Return cached user
  → Response time: 2ms
```

---

### Request Level Caching

Request level caching caches the entire HTTP response based on the request URL and parameters. This is useful for idempotent GET requests that return the same data for the same input.

![image](../../Resources/16-rest-caching/Screenshot%202026-02-07%20at%2012.39.25 PM.png)

#### Where and How to Use:
- Cache GET requests with query parameters
- Cache API responses that don't change frequently
- Implement in API gateways or middleware
- Cache responses for read-only operations
- Use for public APIs with high traffic

#### Pros:
- ✅ Simple to implement at middleware level
- ✅ Transparent to application logic
- ✅ Works well with HTTP caching headers
- ✅ Reduces server load for repeated requests
- ✅ Easy to understand and debug

#### Cons:
- ❌ Cannot cache POST, PUT, DELETE requests
- ❌ Difficult to handle partial updates
- ❌ Cache key generation can be complex with multiple parameters
- ❌ May cache sensitive data unintentionally
- ❌ Requires careful handling of cache invalidation

#### Code Examples:

**NestJS Example:**
```typescript
import { Injectable, NestMiddleware } from '@nestjs/common';
import { Request, Response, NextFunction } from 'express';
import { RedisService } from '@nestjs-modules/ioredis';

@Injectable()
export class CacheMiddleware implements NestMiddleware {
  constructor(private redis: RedisService) {}

  async use(req: Request, res: Response, next: NextFunction) {
    // Only cache GET requests
    if (req.method !== 'GET') {
      return next();
    }

    const cacheKey = `request:${req.originalUrl}`;
    
    // Try to get cached response
    const cachedResponse = await this.redis.get(cacheKey);
    if (cachedResponse) {
      res.set('X-Cache', 'HIT');
      return res.json(JSON.parse(cachedResponse));
    }

    // Intercept response
    const originalJson = res.json.bind(res);
    res.json = function(data: any) {
      // Cache the response for 5 minutes
      this.redis.setex(cacheKey, 300, JSON.stringify(data));
      res.set('X-Cache', 'MISS');
      return originalJson(data);
    };

    next();
  }
}
```

**Python Example:**
```python
from flask import Flask, request, jsonify
import redis
import json
import hashlib

app = Flask(__name__)
redis_client = redis.Redis(host='localhost', port=6379, db=0)

def cache_request(f):
    def decorated_function(*args, **kwargs):
        # Generate cache key from URL and query params
        cache_key = f"request:{request.path}:{request.query_string.decode()}"
        
        # Try cache
        cached = redis_client.get(cache_key)
        if cached:
            return json.loads(cached), 200, {'X-Cache': 'HIT'}
        
        # Execute request
        result = f(*args, **kwargs)
        
        # Cache response for 5 minutes
        redis_client.setex(cache_key, 300, json.dumps(result))
        
        return result, 200, {'X-Cache': 'MISS'}
    
    return decorated_function

@app.route('/api/products')
@cache_request
def get_products():
    products = db.query("SELECT * FROM products")
    return products
```

#### Tiny Example:
```
Request 1: GET /api/products?category=electronics
  → Cache miss → Query database → Return products
  → Response time: 200ms
  → Header: X-Cache: MISS

Request 2: GET /api/products?category=electronics
  → Cache hit → Return cached response
  → Response time: 5ms
  → Header: X-Cache: HIT
```

---

### Conditional Caching

Conditional caching uses HTTP headers like `ETag` and `Last-Modified` to validate whether cached data is still fresh. The server only sends the full response if the data has changed.

![image](../../Resources/16-rest-caching/Screenshot%202026-02-07%20at%2012.42.56 PM.png)

#### Where and How to Use:
- Implement HTTP caching headers (ETag, Last-Modified)
- Use 304 Not Modified responses
- Cache resources that change infrequently
- Reduce bandwidth for large responses
- Implement in REST APIs with versioned resources

#### Pros:
- ✅ Reduces bandwidth by sending only changed data
- ✅ Follows HTTP standards and best practices
- ✅ Works with browser caching automatically
- ✅ Allows clients to validate cache freshness
- ✅ Reduces server load for unchanged resources

#### Cons:
- ❌ Still requires server processing to generate ETag
- ❌ Complexity in managing ETags for dynamic content
- ❌ Requires client support for conditional requests
- ❌ May not work well with streaming responses
- ❌ ETag generation can be expensive for large files

#### Code Examples:

**NestJS Example:**
```typescript
import { Controller, Get, Param, Res } from '@nestjs/common';
import { Response } from 'express';
import * as crypto from 'crypto';

@Controller('api/products')
export class ProductController {
  @Get(':id')
  async getProduct(@Param('id') id: string, @Res() res: Response) {
    const product = await this.database.products.findById(id);
    
    // Generate ETag from product data
    const eTag = crypto
      .createHash('md5')
      .update(JSON.stringify(product))
      .digest('hex');
    
    // Set cache headers
    res.set('ETag', `"${eTag}"`);
    res.set('Cache-Control', 'public, max-age=3600');
    res.set('Last-Modified', new Date(product.updatedAt).toUTCString());
    
    // Check if client has matching ETag
    if (req.headers['if-none-match'] === `"${eTag}"`) {
      return res.status(304).send(); // Not Modified
    }
    
    return res.json(product);
  }
}
```

**Python Example:**
```python
from flask import Flask, request, jsonify, make_response
import hashlib
from datetime import datetime

app = Flask(__name__)

@app.route('/api/products/<product_id>')
def get_product(product_id):
    product = db.query(f"SELECT * FROM products WHERE id = {product_id}")
    
    # Generate ETag
    product_json = json.dumps(product, sort_keys=True)
    etag = hashlib.md5(product_json.encode()).hexdigest()
    
    # Check If-None-Match header
    if request.headers.get('If-None-Match') == f'"{etag}"':
        return '', 304  # Not Modified
    
    response = make_response(jsonify(product))
    response.headers['ETag'] = f'"{etag}"'
    response.headers['Cache-Control'] = 'public, max-age=3600'
    response.headers['Last-Modified'] = datetime.utcnow().strftime('%a, %d %b %Y %H:%M:%S GMT')
    
    return response
```

#### Tiny Example:
```
Request 1: GET /api/products/123
  → Response: 200 OK
  → Headers: ETag: "abc123", Cache-Control: max-age=3600

Request 2: GET /api/products/123 (with If-None-Match: "abc123")
  → Product unchanged
  → Response: 304 Not Modified
  → Bandwidth saved: 95% (no body sent)
```

---

## Cache Invalidation

Cache invalidation is the process of removing or updating stale data from the cache. It's one of the most challenging aspects of caching.

### Write-Through

![image](../../Resources/16-rest-caching/Screenshot%202026-02-07%20at%2012.44.14 PM.png)

Write-Through is a cache invalidation strategy where data is written to both the cache and the database simultaneously. The write operation completes only after both writes succeed.

#### Where and How to Use:
- Critical data that must always be consistent
- Financial transactions and sensitive operations
- When data consistency is more important than write speed
- Small datasets where write latency is acceptable

#### Pros:
- ✅ Guarantees cache and database consistency
- ✅ No stale data in cache
- ✅ Simple to understand and implement
- ✅ Good for critical data

#### Cons:
- ❌ Slower write operations (must wait for both writes)
- ❌ Increased latency for write requests
- ❌ Higher load on both cache and database
- ❌ Not suitable for high-throughput systems

#### Code Examples:

**NestJS Example:**
```typescript
import { Injectable } from '@nestjs/common';
import { RedisService } from '@nestjs-modules/ioredis';

@Injectable()
export class AccountService {
  constructor(
    private redis: RedisService,
    private database: DatabaseService
  ) {}

  async transferMoney(fromId: string, toId: string, amount: number) {
    try {
      // Write to database first
      const transaction = await this.database.transaction(async (trx) => {
        await trx.accounts.update(fromId, { balance: -amount });
        await trx.accounts.update(toId, { balance: +amount });
        return { success: true };
      });

      // Then update cache
      const fromAccount = await this.database.accounts.findById(fromId);
      const toAccount = await this.database.accounts.findById(toId);
      
      await this.redis.set(`account:${fromId}`, JSON.stringify(fromAccount));
      await this.redis.set(`account:${toId}`, JSON.stringify(toAccount));

      return transaction;
    } catch (error) {
      // If either write fails, transaction is rolled back
      throw error;
    }
  }
}
```

**Python Example:**
```python
class AccountService:
    def __init__(self, db, redis_client):
        self.db = db
        self.redis = redis_client
    
    def transfer_money(self, from_id: str, to_id: str, amount: float):
        try:
            # Start database transaction
            with self.db.transaction():
                # Write to database
                self.db.execute(
                    f"UPDATE accounts SET balance = balance - {amount} WHERE id = '{from_id}'"
                )
                self.db.execute(
                    f"UPDATE accounts SET balance = balance + {amount} WHERE id = '{to_id}'"
                )
                
                # Write to cache
                from_account = self.db.query(f"SELECT * FROM accounts WHERE id = '{from_id}'")
                to_account = self.db.query(f"SELECT * FROM accounts WHERE id = '{to_id}'")
                
                self.redis.set(f"account:{from_id}", json.dumps(from_account))
                self.redis.set(f"account:{to_id}", json.dumps(to_account))
                
                return {"success": True}
        except Exception as e:
            # Transaction rolled back automatically
            raise e
```

#### Tiny Example:
```
Write-Through Flow:
1. Client: POST /api/transfer (from: 123, to: 456, amount: 100)
2. Server: Write to database ✓
3. Server: Write to cache ✓
4. Server: Return 200 OK
5. Cache and database are always in sync
```

---

### Write-Behind

Write-Behind (also called Write-Back) is a cache invalidation strategy where data is written to the cache first, and then asynchronously written to the database. This improves write performance but risks data loss if the cache fails before the database write.

![image](../../Resources/16-rest-caching/Screenshot%202026-02-07%20at%2012.44.47 PM.png)

#### Where and How to Use:
- High-throughput systems where write speed is critical
- Non-critical data like user preferences or analytics
- When eventual consistency is acceptable
- Social media likes, views, and counters
- Logging and metrics collection

#### Pros:
- ✅ Fast write operations (only write to cache)
- ✅ Reduced database load
- ✅ Better throughput for write-heavy workloads
- ✅ Improved user experience with faster responses

#### Cons:
- ❌ Risk of data loss if cache fails
- ❌ Temporary inconsistency between cache and database
- ❌ Complex error handling and recovery
- ❌ Requires monitoring and alerting

#### Code Examples:

**NestJS Example:**
```typescript
import { Injectable } from '@nestjs/common';
import { RedisService } from '@nestjs-modules/ioredis';
import { BullModule, InjectQueue } from '@nestjs/bull';
import { Queue } from 'bull';

@Injectable()
export class AnalyticsService {
  constructor(
    private redis: RedisService,
    @InjectQueue('analytics') private analyticsQueue: Queue
  ) {}

  async recordPageView(userId: string, pageId: string) {
    // Write to cache immediately
    const viewKey = `views:${pageId}`;
    await this.redis.incr(viewKey);
    
    // Queue database write for later
    await this.analyticsQueue.add(
      'save-view',
      { userId, pageId },
      { delay: 5000 } // Write to DB after 5 seconds
    );

    return { success: true };
  }

  // Background job to flush cache to database
  async flushViewsToDB() {
    const keys = await this.redis.keys('views:*');
    
    for (const key of keys) {
      const count = await this.redis.get(key);
      const pageId = key.replace('views:', '');
      
      // Write to database
      await this.database.pageViews.increment(pageId, count);
      
      // Clear cache
      await this.redis.del(key);
    }
  }
}
```

**Python Example:**
```python
from celery import Celery
import redis
import json

app = Celery('analytics')
redis_client = redis.Redis(host='localhost', port=6379, db=0)

class AnalyticsService:
    def __init__(self, db):
        self.db = db
    
    def record_page_view(self, user_id: str, page_id: str):
        # Write to cache immediately
        view_key = f"views:{page_id}"
        redis_client.incr(view_key)
        
        # Queue database write
        save_view_to_db.delay(user_id, page_id)
        
        return {"success": True}

@app.task
def save_view_to_db(user_id: str, page_id: str):
    # This runs asynchronously
    db.execute(
        f"INSERT INTO page_views (user_id, page_id) VALUES ('{user_id}', '{page_id}')"
    )

@app.task
def flush_views_to_db():
    # Periodic task to flush all cached views
    keys = redis_client.keys('views:*')
    
    for key in keys:
        count = redis_client.get(key)
        page_id = key.decode().replace('views:', '')
        
        db.execute(
            f"UPDATE pages SET view_count = view_count + {count} WHERE id = '{page_id}'"
        )
        redis_client.delete(key)
```

#### Tiny Example:
```
Write-Behind Flow:
1. Client: POST /api/page-view (page: 123)
2. Server: Write to cache ✓ (instant)
3. Server: Return 200 OK (fast response)
4. Background job: Write to database (after 5 seconds)
5. Cache and database eventually consistent
```

---

### TTL-Based

TTL (Time-To-Live) based cache invalidation automatically removes cached data after a specified duration. This is the simplest and most common invalidation strategy.

![image](../../Resources/16-rest-caching/Screenshot%202026-02-07%20at%2012.47.33 PM.png)

#### Where and How to Use:
- Data that changes at predictable intervals
- User sessions and temporary data
- API responses that don't need real-time accuracy
- Configuration data that changes infrequently
- Most general-purpose caching scenarios

#### Pros:
- ✅ Simple to implement and understand
- ✅ Automatic cleanup of stale data
- ✅ No manual invalidation required
- ✅ Predictable memory usage
- ✅ Works well for most use cases

#### Cons:
- ❌ Stale data served until TTL expires
- ❌ Wasted cache space if data changes before TTL
- ❌ Requires tuning TTL values
- ❌ May not work for rapidly changing data
- ❌ No guarantee of data freshness

#### Code Examples:

**NestJS Example:**
```typescript
import { Injectable } from '@nestjs/common';
import { RedisService } from '@nestjs-modules/ioredis';

@Injectable()
export class ProductService {
  constructor(
    private redis: RedisService,
    private database: DatabaseService
  ) {}

  async getProduct(productId: string) {
    const cacheKey = `product:${productId}`;
    
    // Try cache
    const cached = await this.redis.get(cacheKey);
    if (cached) {
      return JSON.parse(cached);
    }

    // Fetch from database
    const product = await this.database.products.findById(productId);
    
    // Cache with different TTLs based on data type
    const ttl = product.category === 'featured' ? 300 : 3600; // 5 min vs 1 hour
    await this.redis.setex(cacheKey, ttl, JSON.stringify(product));
    
    return product;
  }

  async getProductsByCategory(category: string) {
    const cacheKey = `products:category:${category}`;
    
    const cached = await this.redis.get(cacheKey);
    if (cached) {
      return JSON.parse(cached);
    }

    const products = await this.database.products.findByCategory(category);
    
    // Cache for 30 minutes
    await this.redis.setex(cacheKey, 1800, JSON.stringify(products));
    
    return products;
  }

  // Optional: Manual refresh before TTL expires
  async refreshProductCache(productId: string) {
    const product = await this.database.products.findById(productId);
    const cacheKey = `product:${productId}`;
    
    // Reset TTL to full duration
    await this.redis.setex(cacheKey, 3600, JSON.stringify(product));
    
    return product;
  }
}
```

**Python Example:**
```python
import redis
import json
from datetime import datetime, timedelta

class ProductService:
    def __init__(self, db):
        self.db = db
        self.redis = redis.Redis(host='localhost', port=6379, db=0)
    
    def get_product(self, product_id: str):
        cache_key = f"product:{product_id}"
        
        # Try cache
        cached = self.redis.get(cache_key)
        if cached:
            return json.loads(cached)
        
        # Fetch from database
        product = self.db.query(f"SELECT * FROM products WHERE id = '{product_id}'")
        
        # Cache with TTL (1 hour = 3600 seconds)
        self.redis.setex(cache_key, 3600, json.dumps(product))
        
        return product
    
    def get_products_by_category(self, category: str):
        cache_key = f"products:category:{category}"
        
        cached = self.redis.get(cache_key)
        if cached:
            return json.loads(cached)
        
        products = self.db.query(
            f"SELECT * FROM products WHERE category = '{category}'"
        )
        
        # Cache for 30 minutes
        self.redis.setex(cache_key, 1800, json.dumps(products))
        
        return products
    
    def get_cache_ttl(self, key: str) -> int:
        """Get remaining TTL for a cache key"""
        ttl = self.redis.ttl(key)
        return ttl if ttl > 0 else None
    
    def extend_cache_ttl(self, key: str, additional_seconds: int):
        """Extend TTL for a cache key"""
        current_ttl = self.redis.ttl(key)
        if current_ttl > 0:
            self.redis.expire(key, current_ttl + additional_seconds)
```

#### Tiny Example:
```
TTL-Based Caching:

Cache Key: product:123
TTL: 3600 seconds (1 hour)

Timeline:
00:00 - Cache miss → Query DB → Cache result
00:05 - Cache hit → Return cached data
01:00 - TTL expires → Cache miss → Query DB again
01:05 - Cache hit → Return new cached data
```

![image](../../Resources/16-rest-caching/Screenshot%202026-02-07%20at%2012.48.15 PM.png)

#### TTL Strategy Table:

| Data Type | Recommended TTL | Reason |
|-----------|-----------------|--------|
| User Profile | 1 hour | Changes infrequently |
| Product Catalog | 30 minutes | May change during day |
| Featured Products | 5 minutes | Changes frequently |
| User Session | 24 hours | Long-lived session |
| API Rate Limit | 1 minute | Needs frequent updates |
| Configuration | 1 day | Rarely changes |
| Analytics Data | 5 minutes | Near real-time needed |
| Search Results | 10 minutes | Moderate change rate |

---

## Summary and Best Practices

### Choosing the Right Strategy:

1. **Application Layer Caching**: Use for frequently accessed data that's expensive to compute
2. **Request Level Caching**: Use for idempotent GET requests with stable parameters
3. **Conditional Caching**: Use for resources that change infrequently but need bandwidth optimization
4. **Write-Through**: Use for critical data requiring strong consistency
5. **Write-Behind**: Use for high-throughput systems with non-critical data
6. **TTL-Based**: Use as default for most caching scenarios

### General Best Practices:

- **Monitor Cache Hit Rates**: Aim for 80%+ hit rate
- **Set Appropriate TTLs**: Balance between freshness and performance
- **Implement Cache Warming**: Pre-load frequently accessed data
- **Use Cache Versioning**: Include version in cache keys for easy invalidation
- **Handle Cache Failures Gracefully**: Fall back to database if cache is unavailable
- **Log Cache Operations**: Track hits, misses, and invalidations
- **Test Cache Behavior**: Verify consistency and performance
- **Document Cache Strategy**: Make it clear which data is cached and why
- **Use Distributed Caching**: For multi-instance deployments
- **Implement Cache Metrics**: Monitor memory usage, eviction rates, and performance

### Common Pitfalls to Avoid:

- ❌ Caching sensitive data without encryption
- ❌ Using same TTL for all data types
- ❌ Not handling cache failures
- ❌ Caching POST/PUT/DELETE responses
- ❌ Ignoring cache coherency in distributed systems
- ❌ Not monitoring cache performance
- ❌ Over-caching small datasets
- ❌ Forgetting to invalidate cache on updates

---

<button onclick="window.scrollTo({top: 0, behavior: 'instant'})" style="position: fixed; bottom: 30px; right: 30px; z-index: 99; padding: 12px 16px; background-color: #007bff; color: white; border: none; border-radius: 50%; cursor: pointer; font-size: 18px; width: 50px; height: 50px; text-align: center; line-height: 24px; box-shadow: 0 4px 8px rgba(0, 0, 0, 0.2); font-weight: bold; transition: background-color 0.3s;" onmouseover="this.style.backgroundColor='#0056b3'" onmouseout="this.style.backgroundColor='#007bff'" onmousedown="this.style.backgroundColor='#003d82'" onmouseup="this.style.backgroundColor='#0056b3'" title="Go to top">↑</button>
