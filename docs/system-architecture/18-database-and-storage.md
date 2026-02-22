# Databases and Storage

## Table of Contents
1. [What is Database](#what-is-database)
2. [Relational Databases](#relational-databases)
3. [Relational DB vs Non-relational DB](#relational-db-vs-non-relational-db)
4. [Schema](#schema)
5. [SQL vs NoSQL](#sql-vs-nosql)
6. [Transactions and ACID](#transactions-and-acid)
7. [Database Indexes](#database-indexes)
8. [Non-relational / Key-Value Databases](#non-relational--key-value-databases)
9. [Blob Store](#blob-store)
10. [Time Series Database](#time-series-database)
11. [Graph Database](#graph-database)
12. [Wide Column Database](#wide-column-database)
13. [Multi-Model Database](#multi-model-database)
14. [Partitioning](#partitioning)
15. [Sharding](#sharding)

---

## What is Database

- **Definition**: Organized collection of structured data stored persistently and accessed efficiently
- **Purpose**: Store, retrieve, update, and delete data reliably
- **Types**:
  - **Relational (SQL)**: Structured data in tables with relationships (MySQL, PostgreSQL)
  - **NoSQL**: Flexible schema for unstructured data (MongoDB, Cassandra, DynamoDB)
  - **Key-Value**: Fast lookups with simple key-value pairs (Redis, Memcached)
  - **Vector Database**: Stores and indexes high-dimensional vectors for similarity search (Pinecone, Faiss, Milvus)
- **Key Characteristics**:
  - **Persistence**: Data survives system failures
  - **Concurrency**: Multiple clients can access simultaneously
  - **Consistency**: Data integrity maintained
  - **Query Efficiency**: Indexes and optimization for fast retrieval
- **Architect's Perspective**: Choose database based on data structure, query patterns, scale requirements, and consistency needs

---

## Relational Databases

- **Definition**: Stores data in structured tables with rows and columns, connected through relationships
- **Core Concept**: Data organized into normalized tables with primary and foreign keys
- **Examples**: PostgreSQL, MySQL, Oracle, SQL Server, MariaDB
- **Structure**:
  - **Tables**: Collections of related data
  - **Rows**: Individual records
  - **Columns**: Attributes/fields with defined data types
  - **Primary Key**: Unique identifier for each row
  - **Foreign Key**: Links to primary key in another table
- **Advantages**:
  - Strong data consistency and integrity
  - ACID compliance
  - Complex queries with JOINs
  - Normalized data reduces redundancy
  - Well-established and mature
- **Disadvantages**:
  - Scaling horizontally is difficult
  - Schema changes require migrations
  - Not ideal for unstructured data
  - Performance degrades with complex joins at scale

---

## Relational DB vs Non-relational DB

| Aspect | Relational (SQL) | Non-relational (NoSQL) |
|--------|------------------|------------------------|
| **Data Model** | Tables with rows/columns | Documents, key-value, graphs, columns |
| **Schema** | Fixed, predefined | Flexible, dynamic |
| **Scalability** | Vertical (scale-up) | Horizontal (scale-out) |
| **Consistency** | Strong (ACID) | Eventual (BASE) |
| **Query Language** | SQL (standardized) | Varies (MongoDB, Cassandra, etc.) |
| **Relationships** | Explicit (foreign keys) | Embedded or denormalized |
| **Performance** | Good for complex queries | Fast for simple lookups |
| **Data Integrity** | Enforced by constraints | Application-level responsibility |
| **Use Case** | Structured, relational data | Unstructured, high-volume data |
| **Examples** | PostgreSQL, MySQL | MongoDB, Cassandra, DynamoDB |

---

## Schema

- **Definition**: Blueprint defining structure, data types, constraints, and relationships in a database
- **Components**:
  - **Tables/Collections**: Data containers
  - **Columns/Fields**: Named attributes with data types
  - **Constraints**: Rules enforcing data validity (NOT NULL, UNIQUE, CHECK)
  - **Relationships**: Foreign keys linking tables
  - **Indexes**: Structures for fast data retrieval
- **Schema Types**:
  - **Fixed Schema (SQL)**: Defined upfront, changes require migration
  - **Flexible Schema (NoSQL)**: Documents can have different structures
  - **Schema-on-Read**: Schema applied during query (NoSQL)
  - **Schema-on-Write**: Schema enforced during insertion (SQL)
- **Schema Design Principles**:
  - Normalize to reduce redundancy (SQL)
  - Denormalize for performance (NoSQL)
  - Plan for growth and evolution
  - Document relationships and constraints

---

## SQL vs NoSQL

### SQL (Structured Query Language)

- **Purpose**: Query language for relational databases
- **Characteristics**:
  - Standardized syntax across databases
  - Declarative (specify what, not how)
  - Supports complex queries with JOINs, aggregations, subqueries
  - ACID transactions
  - Strong schema enforcement
- **Common Operations**:
  ```sql
  SELECT * FROM users WHERE age > 25;
  INSERT INTO users (name, email) VALUES ('John', 'john@example.com');
  UPDATE users SET age = 30 WHERE id = 1;
  DELETE FROM users WHERE id = 1;
  ```

### NoSQL (Not Only SQL)

- **Purpose**: Query language for non-relational databases
- **Characteristics**:
  - Database-specific syntax (MongoDB, Cassandra, DynamoDB)
  - Flexible schema
  - Optimized for specific access patterns
  - Eventual consistency
  - Horizontal scalability
- **Common Operations** (MongoDB example):
  ```javascript
  db.users.find({ age: { $gt: 25 } });
  db.users.insertOne({ name: 'John', email: 'john@example.com' });
  db.users.updateOne({ _id: 1 }, { $set: { age: 30 } });
  db.users.deleteOne({ _id: 1 });
  ```

### When to Use

- **SQL**: Financial systems, e-commerce, CRM, structured data with complex relationships
- **NoSQL**: Real-time analytics, IoT data, content management, high-volume unstructured data

---

## Transactions and ACID

### What is a Transaction

- **Definition**: Sequence of database operations treated as a single atomic unit
- **All-or-Nothing**: Either all operations succeed or all rollback
- **Ensures**: Data consistency even with concurrent access or failures

### ACID Properties

#### **Atomicity**
- All operations in transaction complete or none do
- No partial updates
- **Example**: Bank transfer - debit from account A and credit to account B both succeed or both fail

#### **Consistency**
- Database moves from one valid state to another
- All constraints and rules are maintained
- **Example**: Account balance never goes negative (if constraint exists)

#### **Isolation**
- Concurrent transactions don't interfere with each other
- Each transaction executes independently
- **Isolation Levels**:
  - **Read Uncommitted**: Dirty reads possible
  - **Read Committed**: No dirty reads
  - **Repeatable Read**: No dirty or non-repeatable reads
  - **Serializable**: Complete isolation (slowest)
- **Example**: Two users withdrawing from same account simultaneously don't cause overdraft

#### **Durability**
- Once committed, data persists despite failures
- Survives crashes, power loss, hardware failures
- **Example**: After bank confirms transfer, data is safe even if server crashes

### ACID Transaction Example

```sql
BEGIN TRANSACTION;

-- Debit from Account A
UPDATE accounts SET balance = balance - 100 WHERE id = 'A';

-- Credit to Account B
UPDATE accounts SET balance = balance + 100 WHERE id = 'B';

-- If both succeed, commit
COMMIT;

-- If any fails, rollback
ROLLBACK;
```

**Scenario**: If first UPDATE succeeds but second fails, ROLLBACK ensures Account A's balance is restored.

---

## Database Indexes

- **Definition**: Data structure that improves query performance by enabling faster data retrieval
- **How It Works**: Creates sorted lookup table pointing to actual data
- **Trade-off**: Faster reads, slower writes, additional storage

### Common Index Types

- **Primary Index**: On primary key, ensures uniqueness
- **Secondary Index**: On non-primary columns for faster queries
- **Composite Index**: On multiple columns
- **Full-Text Index**: For text search
- **Hash Index**: For equality comparisons
- **B-Tree Index**: For range queries (most common)


### Index Types Comparison

| **Index Type** | **Definition** | **Benefits** | **Trade-offs** | **When to Use (Applications)** | **Time Complexity** |
|---|---|---|---|---|---|
| **Primary Index** | Index on primary key column(s) that uniquely identifies each row; automatically created and enforced | Ensures uniqueness; fast row lookups by primary key; foundation for relationships | Mandatory overhead; cannot be removed; impacts write performance | All tables require this; essential for data integrity and relationships | O(log n) |
| **Secondary Index** | Index on non-primary columns to speed up queries on those specific columns | Dramatically improves query performance on indexed columns; enables efficient filtering; supports sorting | Increases storage; slows INSERT/UPDATE/DELETE; requires maintenance; can fragment | Frequently queried columns (WHERE, ORDER BY, JOIN conditions); columns with good selectivity | O(log n) |
| **Composite Index** | Index on multiple columns together (e.g., `(user_id, created_at)`); order matters | Optimizes queries filtering/sorting on multiple columns; reduces need for multiple indexes; efficient for range queries | Higher storage overhead; less flexible than single-column indexes; order-dependent; complex to maintain | Multi-column WHERE clauses; queries like `WHERE user_id = 5 AND created_at > '2026-01-01'`; covering indexes | O(log n) |
| **Full-Text Index** | Specialized index for text search; tokenizes and indexes words in text fields | Enables fast full-text search; supports phrase search, boolean operators, relevance ranking | Significant storage overhead; slower writes; complex query syntax; language-dependent | Search engines; document search; content management systems; blog post/article search | O(1) to O(log n) depending on implementation |
| **Hash Index** | Index using hash function to map values to bucket locations; fast for exact matches | Extremely fast equality lookups; O(1) average case; minimal memory overhead for small datasets | Cannot support range queries; no sorting capability; poor for LIKE queries; hash collisions possible; not suitable for large datasets | Exact match lookups only; caching layers; session storage; small lookup tables; equality joins | O(1) average, O(n) worst case |
| **B-Tree Index** | Balanced tree structure with sorted keys; most common index type in relational databases | Supports range queries efficiently; enables sorting; works for equality and inequality; self-balancing; predictable performance | Moderate storage overhead; slower than hash for exact matches; requires rebalancing on updates; tree traversal overhead | General-purpose indexing; range queries; sorting; most relational database use cases; default choice | O(log n) |
| **Bitmap Index** | Index storing bitmap (array of bits) for each distinct value; efficient for low-cardinality columns | Excellent compression for low-cardinality data; fast for boolean/categorical columns; efficient for AND/OR operations | Poor for high-cardinality columns; slow for updates; not suitable for unique values; limited query flexibility | Data warehousing; analytical queries; gender/status/category columns; columns with few distinct values | O(log n) for lookup, O(1) for bit operations |
| **Spatial Index** | Index for geographic/geometric data (points, polygons, lines); uses R-tree or Quad-tree structures | Enables efficient geographic queries; fast nearest-neighbor searches; supports range queries on coordinates | Specialized use case; complex implementation; higher storage; slower writes; requires specialized databases | Geographic information systems (GIS); location-based services; mapping applications; spatial queries like "find restaurants near me" | O(log n) |
| **Inverted Index** | Maps content (words, terms) to document locations; reverse of normal indexing | Enables fast full-text search; supports complex search queries; foundation of search engines; efficient for text retrieval | High storage overhead; slower writes; complex maintenance; requires tokenization; language-dependent | Search engines; information retrieval; document databases; Elasticsearch; Lucene-based systems | O(1) to O(log n) |
| **Covering Index** | Index that includes all columns needed for a query (index-only scan possible) | Eliminates need to access main table; fastest possible query execution; reduces I/O operations | Larger index size; more storage overhead; slower writes; must be maintained for each query pattern | Frequently executed queries; read-heavy workloads; queries that can be satisfied entirely from index | O(log n) |
| **Partial Index** | Index on subset of rows matching a condition (e.g., `WHERE status = 'active'`) | Smaller index size; faster writes on excluded rows; reduced storage; faster index scans | Limited applicability; only useful for specific query patterns; requires careful planning; less flexible | Filtering on status columns; soft deletes; archival patterns; queries on active/recent data only | O(log n) |
| **Unique Index** | Index enforcing uniqueness constraint on column(s); similar to primary key but allows NULL | Prevents duplicate values; enforces data integrity; enables fast lookups; can have multiple per table | Write performance impact; storage overhead; NULL handling complexity; constraint violation checks | Email addresses; usernames; SKU codes; any column requiring uniqueness; alternative keys | O(log n) |


### Index Example

```sql
-- Without index: O(n) scan
SELECT * FROM users WHERE email = 'john@example.com';

-- With index: O(log n) lookup
CREATE INDEX idx_email ON users(email);
SELECT * FROM users WHERE email = 'john@example.com';
```

### Pros and Cons of Indexes

| Aspect | Pros | Cons |
|--------|------|------|
| **Query Performance** | Dramatically faster lookups and range queries | - |
| **Write Performance** | - | Slower INSERT, UPDATE, DELETE (index must be updated) |
| **Storage** | - | Requires additional disk space (can be significant) |
| **Memory** | - | Indexes loaded in memory, increases RAM usage |
| **Maintenance** | - | Index fragmentation over time, requires rebuilding |
| **Selectivity** | Excellent for selective columns (low cardinality) | Poor for low-selectivity columns (many duplicates) |
| **Sorting** | Enables efficient ORDER BY and GROUP BY | - |
| **Joins** | Speeds up JOIN operations | - |
| **Complexity** | - | More indexes = harder to optimize query planner |

### Index Strategy

- Index columns frequently used in WHERE, JOIN, ORDER BY clauses
- Avoid indexing low-cardinality columns (many duplicates)
- Monitor index usage and remove unused indexes
- Balance read performance with write performance

---

## Non-relational / Key-Value Databases

- **Definition**: Stores data as simple key-value pairs without schema
- **Data Model**: Key → Value mapping (value can be string, number, object, list)
- **Access Pattern**: Direct lookup by key (O(1) average)
- **Examples**: Redis, Memcached, DynamoDB, Dynamo

### Characteristics

- **Ultra-fast**: In-memory or optimized for speed
- **Simple**: No complex queries, just get/set/delete
- **Scalable**: Easy to distribute across nodes
- **Flexible**: Values can be any data type
- **No Relationships**: Data is independent

### Use Cases

- **Caching**: Session storage, query results, frequently accessed data
- **Real-time Analytics**: Counters, leaderboards, metrics
- **Rate Limiting**: Track API calls per user
- **Message Queues**: Task distribution
- **Pub/Sub**: Real-time messaging

### Example

```
Key: "user:123"
Value: { name: "John", email: "john@example.com", age: 30 }

Key: "session:abc123"
Value: "user_id=456&login_time=1234567890"

Key: "leaderboard:game1"
Value: [{ player: "Alice", score: 1000 }, { player: "Bob", score: 950 }]
```

### Limitations

- No complex queries (no WHERE, JOIN, aggregations)
- Limited data relationships
- Not suitable for transactional data
- Data must fit in memory (for in-memory stores)

---

## Blob Store

- **Definition**: Storage service for unstructured binary data (files, images, videos, documents)
- **Characteristics**:
  - Stores objects as-is without parsing or indexing
  - Accessed by unique identifier (key/path)
  - Highly scalable and durable
  - Cost-effective for large files
  - Supports versioning and lifecycle policies
- **Use Cases**:
  - User uploads (images, documents, videos)
  - Static website assets
  - Backups and archives
  - Log files and analytics data
  - Machine learning datasets

### Major Blob Store Services

#### **Amazon S3 (Simple Storage Service)**
- **Features**:
  - 99.99% availability, 11 nines durability
  - Unlimited scalability
  - Versioning, lifecycle policies, encryption
  - Access control (IAM, bucket policies, ACLs)
  - CloudFront CDN integration
- **Pricing**: Pay per GB stored + data transfer
- **Use Case**: Primary choice for AWS ecosystems

#### **Google Cloud Storage (GCS)**
- **Features**:
  - Multi-region redundancy
  - Strong consistency
  - Lifecycle management
  - Integration with BigQuery for analytics
  - Signed URLs for temporary access
- **Pricing**: Similar to S3, competitive rates
- **Use Case**: Google Cloud ecosystems, analytics workloads

#### **Azure Blob Storage**
- **Features**:
  - Hot, Cool, Archive tiers for cost optimization
  - Immutable storage for compliance
  - Soft delete and versioning
  - Integration with Azure Data Lake
  - Encryption at rest and in transit
- **Pricing**: Tiered pricing based on access frequency
- **Use Case**: Microsoft ecosystems, compliance-heavy workloads

### Comparison

| Feature | S3 | GCS | Azure Blob |
|---------|-----|-----|-----------|
| **Consistency** | Eventual | Strong | Strong |
| **Availability** | 99.99% | 99.95% | 99.9% |
| **Durability** | 11 nines | 11 nines | 11 nines |
| **Tiers** | Standard, Infrequent, Glacier | Standard, Nearline, Coldline | Hot, Cool, Archive |
| **Ecosystem** | AWS (largest) | Google Cloud | Azure |

---

## Time Series Database

- **Definition**: Optimized database for storing and querying time-stamped data points
- **Data Model**: (timestamp, metric_name, value, tags)
- **Examples**: InfluxDB, Prometheus, TimescaleDB, Graphite, Datadog

### Characteristics

- **Append-only**: Data rarely updated, mostly new inserts
- **Ordered by Time**: Natural ordering by timestamp
- **High Write Throughput**: Millions of data points per second
- **Compression**: Efficient storage of similar values
- **Downsampling**: Aggregate old data to save space

### What Kind of Data is Good for Time Series DB

- **Metrics**: CPU usage, memory, disk I/O, network traffic
- **Sensor Data**: Temperature, humidity, pressure from IoT devices
- **Stock Prices**: Historical price data with timestamps
- **Application Logs**: Events with timestamps
- **User Activity**: Page views, clicks, interactions over time
- **Health Monitoring**: Heart rate, blood pressure readings
- **Network Monitoring**: Packet loss, latency, bandwidth usage

### Example

```
Timestamp: 2026-02-09T22:00:00Z, Metric: cpu_usage, Value: 45.2%, Tags: {host: server1, region: us-east}
Timestamp: 2026-02-09T22:01:00Z, Metric: cpu_usage, Value: 48.5%, Tags: {host: server1, region: us-east}
Timestamp: 2026-02-09T22:02:00Z, Metric: cpu_usage, Value: 52.1%, Tags: {host: server1, region: us-east}
```

### Query Example (InfluxQL)

```sql
SELECT mean(cpu_usage) FROM metrics 
WHERE time > now() - 1h AND host = 'server1' 
GROUP BY time(5m)
```

### Advantages

- Optimized for time-range queries
- Efficient compression (80-90% space savings)
- Fast aggregations (sum, avg, max, min)
- Natural retention policies (delete old data)

---

## Graph Database

- **Definition**: Stores data as nodes (entities) and edges (relationships) optimized for traversal
- **Examples**: Neo4j, Amazon Neptune, ArangoDB, JanusGraph
- **Data Model**: Nodes with properties connected by typed relationships

### Use Cases

- **Social Networks**: Friends, followers, connections
- **Recommendation Engines**: Product recommendations based on user behavior
- **Knowledge Graphs**: Wikipedia, semantic relationships
- **Fraud Detection**: Detect suspicious patterns in transaction networks
- **Identity and Access**: User roles, permissions, hierarchies
- **Master Data Management**: Entity relationships across systems

### Example

```
Node: User {id: 1, name: "Alice"}
Node: User {id: 2, name: "Bob"}
Node: Product {id: 101, name: "Laptop"}

Edge: Alice --[FOLLOWS]--> Bob
Edge: Alice --[PURCHASED]--> Laptop
Edge: Bob --[PURCHASED]--> Laptop
```

### Query Example (Cypher - Neo4j)

```cypher
-- Find friends of friends
MATCH (user:User {name: "Alice"})-[:FOLLOWS]->(friend)-[:FOLLOWS]->(foaf)
RETURN foaf.name

-- Find products purchased by similar users
MATCH (user:User {name: "Alice"})-[:PURCHASED]->(product)<-[:PURCHASED]-(similar_user)
RETURN similar_user.name, product.name
```

### Advantages

- Extremely fast relationship traversal (constant time)
- Natural representation of connected data
- Powerful pattern matching queries
- Ideal for recommendation and fraud detection

---

## Wide Column Database

- **Definition**: Stores data in columns grouped by column families, optimized for analytical queries
- **Examples**: HBase, Cassandra, DynamoDB (partially)
- **Data Model**: Row key → Column families → Columns → Values

### Characteristics

- **Column-Oriented**: Data stored by column, not row
- **Sparse**: Columns can be empty without storage overhead
- **Scalable**: Distributed across nodes
- **High Write Throughput**: Optimized for bulk inserts
- **Compression**: Excellent compression for similar values in columns

### Structure

```
Row Key: user:123
  Column Family: profile
    name: "John"
    email: "john@example.com"
    age: 30
  Column Family: activity
    last_login: 2026-02-09
    login_count: 150
```

### Use Cases

- **Time Series Data**: Metrics, logs with timestamps as row keys
- **Analytics**: Columnar storage efficient for aggregations
- **Real-time Analytics**: Fast reads and writes at scale
- **Sensor Data**: IoT data from millions of devices

### Advantages

- Excellent compression (column values are similar)
- Fast analytical queries (only read needed columns)
- Horizontal scalability
- High write throughput

### Disadvantages

- Complex query language
- Not ideal for transactional workloads
- Steep learning curve

---

## Multi-Model Database

- **Definition**: Single database supporting multiple data models (relational, document, graph, key-value)
- **Examples**: FaunaDB, ArangoDB, Cosmos DB, Firebase, MongoDB (with extensions)
- **Purpose**: Flexibility to use best model for different use cases without multiple databases

### Supported Models

- **Document**: JSON-like flexible schema
- **Relational**: Tables with relationships
- **Graph**: Nodes and edges
- **Key-Value**: Simple lookups
- **Search**: Full-text search capabilities

### Example (FaunaDB)

```javascript
// Document model
db.users.insert({ name: "John", email: "john@example.com" });

// Graph model
db.follows.insert({ _from: "users/1", _to: "users/2" });

// Key-value model
db.cache.insert({ _key: "session:123", data: {...} });

// Query across models
FOR user IN users
  FOR follower IN INBOUND user follows
    RETURN { user: user.name, follower: follower.name }
```

### Advantages

- Single database for multiple use cases
- Reduced operational complexity
- Consistent query language across models
- Easier data migration and consolidation

### Disadvantages

- May not be optimal for any single use case
- Larger learning curve
- Potentially higher costs
- Less mature than specialized databases

---

## Database Selection Guide

```
┌─────────────────────────────────────────────────────────────┐
│                   Database Selection                        │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  Structured + Relationships?  ──→  Relational (PostgreSQL)  │
│                                                             │
│  Flexible Schema + Scale?     ──→  Document (MongoDB)       │
│                                                             │
│  Fast Lookups?                ──→  Key-Value (Redis)        │
│                                                             │
│  Time-stamped Metrics?        ──→  Time Series (InfluxDB)   │
│                                                             │
│  Connected Data?              ──→  Graph (Neo4j)            │
│                                                             │
│  Unstructured Files?          ──→  Blob Store (S3)          │
│                                                             │
│  Analytical Queries?          ──→  Wide Column (Cassandra)  │
│                                                             │
│  Multiple Models?             ──→  Multi-Model (FaunaDB)    │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## Partitioning

### What is Partitioning

- **Definition**: Dividing a single table into smaller, manageable pieces (partitions) while keeping them in the same database
- **Problem It Solves**:
  - Improves query performance by scanning only relevant partitions
  - Reduces memory and I/O overhead
  - Enables easier maintenance and archival of old data
  - Improves scalability within a single database instance
  - Simplifies backup and recovery operations

### Types of Partitioning

#### **Horizontal Partitioning**
- Divides rows based on a partition key
- Different rows go to different partitions
- **Example**: Partition users table by user_id ranges (1-1000, 1001-2000, etc.)
- **Use Case**: When you have millions of rows and want to split by data ranges

#### **Vertical Partitioning**
- Divides columns into separate tables
- Different columns go to different partitions
- **Example**: Separate user profile (name, email) from user activity (login_count, last_login)
- **Use Case**: When some columns are accessed frequently while others are rarely used

#### **Functional Partitioning**
- Divides data based on business function or domain
- **Example**: Partition orders table by region (US, EU, APAC) or by product category
- **Use Case**: When different business units need different data subsets

#### **Range-based Partitioning**
- Divides data into ranges based on a column value
- **Example**: Partition logs by date ranges (Jan 2026, Feb 2026, Mar 2026)
- **Use Case**: Time-series data, historical data, or data with natural ordering

### Examples of Each Partitioning Type

**Horizontal Partitioning Example:**
```sql
-- Partition users by user_id ranges
CREATE TABLE users_partition_1 AS SELECT * FROM users WHERE user_id BETWEEN 1 AND 1000;
CREATE TABLE users_partition_2 AS SELECT * FROM users WHERE user_id BETWEEN 1001 AND 2000;
CREATE TABLE users_partition_3 AS SELECT * FROM users WHERE user_id BETWEEN 2001 AND 3000;
```

**Vertical Partitioning Example:**
```sql
-- Separate frequently accessed columns
CREATE TABLE user_profile (user_id, name, email);
CREATE TABLE user_activity (user_id, login_count, last_login, created_at);
```

**Functional Partitioning Example:**
```sql
-- Partition by region
CREATE TABLE orders_us AS SELECT * FROM orders WHERE region = 'US';
CREATE TABLE orders_eu AS SELECT * FROM orders WHERE region = 'EU';
CREATE TABLE orders_apac AS SELECT * FROM orders WHERE region = 'APAC';
```

**Range-based Partitioning Example:**
```sql
-- Partition logs by month
CREATE TABLE logs_2026_01 AS SELECT * FROM logs WHERE created_at BETWEEN '2026-01-01' AND '2026-01-31';
CREATE TABLE logs_2026_02 AS SELECT * FROM logs WHERE created_at BETWEEN '2026-02-01' AND '2026-02-28';
CREATE TABLE logs_2026_03 AS SELECT * FROM logs WHERE created_at BETWEEN '2026-03-01' AND '2026-03-31';
```

### Partition Pruning

- **Definition**: Query optimizer automatically eliminates partitions that don't match query conditions
- **How It Works**: When you query with a WHERE clause on the partition key, the database skips scanning irrelevant partitions
- **Example**:
  ```sql
  -- Query only scans partition_2 (user_id 1001-2000)
  SELECT * FROM users WHERE user_id = 1500;
  ```
- **Benefits**:
  - Dramatically reduces I/O operations
  - Improves query performance
  - Reduces memory usage
  - Enables efficient handling of large datasets

### Best Way to Choose Partitioning

1. **Identify Access Patterns**: Understand how data is queried most frequently
2. **Choose Partition Key**: Select a column that:
   - Is frequently used in WHERE clauses
   - Has good cardinality (many distinct values)
   - Distributes data evenly
3. **Consider Data Growth**: Ensure partitions won't become too large
4. **Plan for Maintenance**: Choose partitioning strategy that simplifies archival and cleanup
5. **Test Performance**: Benchmark query performance before and after partitioning

### How to Partition

1. **Analyze Current Queries**: Identify most common query patterns
2. **Select Partition Strategy**: Choose horizontal, vertical, functional, or range-based
3. **Define Partition Key**: Pick column(s) that will divide data
4. **Create Partitions**: Use database-specific syntax (PostgreSQL, MySQL, Oracle all differ)
5. **Migrate Data**: Move existing data into appropriate partitions
6. **Update Application Logic**: Ensure queries include partition key for pruning
7. **Monitor Performance**: Track query performance and adjust if needed

### Benefits and Tradeoffs

| Aspect | Benefits | Tradeoffs |
|--------|----------|-----------|
| **Query Performance** | Partition pruning dramatically speeds up queries | Queries without partition key may be slower |
| **Maintenance** | Easier to archive/delete old partitions | Increased complexity in schema management |
| **Scalability** | Handles larger datasets within single database | Still limited by single database capacity |
| **Data Organization** | Logical separation of data | Requires careful planning of partition key |
| **Backup/Recovery** | Can backup/restore individual partitions | More complex backup strategies |
| **Write Performance** | Can be optimized per partition | May slow down writes if partitions unbalanced |
| **Storage** | Better space utilization with pruning | Potential for uneven partition sizes |
| **Complexity** | Simplifies some operations | Adds complexity to application logic |

---

## Sharding

### What is Sharding

- **Definition**: Distributing data across multiple independent database servers (shards), where each shard holds a subset of data
- **Problem It Solves**:
  - Enables horizontal scaling beyond single database limits
  - Distributes load across multiple servers
  - Improves query performance by reducing data per server
  - Increases availability (failure of one shard doesn't affect others)
  - Allows independent scaling of different data subsets

### Example of Sharding

**Scenario**: E-commerce platform with millions of users

```
User ID 1-1000000    → Shard 1 (Database Server 1)
User ID 1000001-2000000 → Shard 2 (Database Server 2)
User ID 2000001-3000000 → Shard 3 (Database Server 3)
User ID 3000001-4000000 → Shard 4 (Database Server 4)

Query for user_id = 1500000:
1. Hash function determines: 1500000 % 4 = 1 → Shard 2
2. Route query to Database Server 2
3. Retrieve data from Shard 2
```

### Hash Function and Shard Key

#### **How Hash Function Helps**
- **Purpose**: Consistently maps data to specific shards
- **Consistency**: Same key always maps to same shard
- **Distribution**: Spreads data evenly across shards
- **Example**:
  ```
  hash(user_id) % number_of_shards = shard_number
  hash(1500000) % 4 = 1 → Shard 2
  hash(2500000) % 4 = 2 → Shard 3
  ```

#### **How to Choose Shard Key**

The shard key must have these characteristics:

##### **High Cardinality**
- **Definition**: Column has many distinct values
- **Why Important**: Ensures even distribution across shards
- **Good Example**: user_id (millions of unique values)
- **Bad Example**: gender (only 2-3 values, creates hotspots)

##### **Even Distribution**
- **Definition**: Data distributed uniformly across shards
- **Why Important**: Prevents some shards from becoming overloaded
- **Good Example**: user_id with hash function
- **Bad Example**: timestamp (recent data clusters in one shard)

##### **Stability**
- **Definition**: Shard key value doesn't change frequently
- **Why Important**: Prevents expensive data migration
- **Good Example**: user_id (never changes)
- **Bad Example**: user_location (users move, requires resharding)

### Query Efficiency with Even Distribution

When data is evenly distributed:

```
Without Sharding (Single Database):
- Query scans 100 million rows
- Response time: 5 seconds

With Sharding (4 Shards, Even Distribution):
- Each shard has 25 million rows
- Query scans only 25 million rows
- Response time: 1.25 seconds (4x faster)
- Can handle 4x more concurrent queries
```

**Benefits of Even Distribution:**
- Each shard handles equal load
- Query performance is predictable
- No single shard becomes bottleneck
- Easier capacity planning

### Avoiding Multi-Shard Queries

**Problem**: Queries that don't include shard key must hit all shards

```sql
-- GOOD: Includes shard key (user_id)
SELECT * FROM users WHERE user_id = 1500000;
-- Routes to single shard, fast

-- BAD: Doesn't include shard key
SELECT * FROM users WHERE email = 'john@example.com';
-- Must query all 4 shards, slow (scatter-gather)
```

**How to Avoid Multi-Shard Queries:**
1. **Design Schema**: Ensure frequently queried columns include shard key
2. **Denormalize Data**: Store shard key in related tables
3. **Use Secondary Indexes**: Maintain lookup tables mapping non-shard-key columns to shard key
4. **Application Logic**: Always include shard key in queries when possible
5. **Query Planning**: Analyze queries and refactor to include shard key

### Consistent Hashing

- **Definition**: Hash function that minimizes data movement when shards are added/removed
- **Problem It Solves**: Traditional modulo hashing requires resharding entire dataset when shard count changes
- **How It Works**:
  ```
  Traditional Hashing:
  hash(key) % 4 = shard_number
  If add 5th shard: hash(key) % 5 = new_shard_number
  Result: 80% of data must move to new shards (expensive!)

  Consistent Hashing:
  - Arrange shards in a ring (0 to 2^32)
  - Hash key to position on ring
  - Find next shard clockwise
  - Adding new shard only affects ~1/n of data
  ```
- **Benefits**:
  - Minimal data movement when scaling
  - Better load balancing
  - Easier to add/remove shards
  - Reduces resharding overhead

### Best Way to Choose Sharding

1. **Assess Scale Requirements**: Determine if single database can handle growth
2. **Identify Shard Key**: Choose column with high cardinality, even distribution, stability
3. **Calculate Shard Count**: Plan for 2-3x expected growth
4. **Design Shard Strategy**: Decide between range-based, hash-based, or directory-based
5. **Plan for Resharding**: Have strategy for adding shards in future
6. **Test Thoroughly**: Benchmark performance and failover scenarios

### Reducing Data Hotspots

**Problem**: Some shards receive disproportionate traffic

**Solutions:**
1. **Better Shard Key**: Choose key with more even distribution
2. **Composite Shard Key**: Combine multiple columns for better distribution
3. **Shard Splitting**: Divide hot shard into multiple shards
4. **Caching Layer**: Cache frequently accessed data to reduce database load
5. **Read Replicas**: Add read replicas for hot shards
6. **Time-based Sharding**: For time-series data, shard by time + user_id

**Example of Hotspot:**
```
Shard by country:
- US shard: 80% of traffic (hotspot)
- EU shard: 15% of traffic
- APAC shard: 5% of traffic

Solution: Shard by country + region:
- US-East: 30% of traffic
- US-West: 30% of traffic
- EU-Central: 15% of traffic
- APAC: 25% of traffic
```

### Operational Complexity

#### **Operational Transactions**
- **Challenge**: Transactions spanning multiple shards are difficult
- **Solution Options**:
  - Avoid cross-shard transactions (design schema to prevent them)
  - Use distributed transactions (2-phase commit, slower)
  - Accept eventual consistency
  - Use saga pattern for distributed transactions
- **Example Problem**:
  ```sql
  -- User 1 on Shard 1, User 2 on Shard 2
  BEGIN TRANSACTION;
  UPDATE accounts SET balance = balance - 100 WHERE user_id = 1;
  UPDATE accounts SET balance = balance + 100 WHERE user_id = 2;
  COMMIT;
  -- Requires coordination across shards
  ```

#### **Global Secondary Indexes**
- **Challenge**: Indexes spanning all shards are expensive to maintain
- **Solution Options**:
  - Maintain local indexes per shard
  - Use separate index service (Elasticsearch)
  - Denormalize data to avoid complex queries
  - Accept slower queries without shard key
- **Example**:
  ```
  Query by email (not shard key):
  - Without index: Must scan all shards (slow)
  - With global index: Maintain separate index service
  - Index maps email → user_id → shard_number
  ```

#### **Monitoring**
- **Challenges**:
  - Monitoring multiple databases instead of one
  - Detecting hotspots across shards
  - Tracking data consistency
  - Identifying slow queries per shard
- **Solutions**:
  - Centralized monitoring dashboard
  - Per-shard metrics and alerts
  - Distributed tracing
  - Regular data consistency checks

### Benefits and Tradeoffs

| Aspect | Benefits | Tradeoffs |
|--------|----------|-----------|
| **Scalability** | Horizontal scaling beyond single database limits | Increased operational complexity |
| **Performance** | Reduced data per shard, faster queries | Multi-shard queries are slow |
| **Availability** | Failure of one shard doesn't affect others | Partial outage if shard fails |
| **Load Distribution** | Spreads load across multiple servers | Requires careful shard key selection |
| **Cost** | Can use commodity hardware | Multiple databases = higher infrastructure cost |
| **Transactions** | Can be optimized per shard | Cross-shard transactions are difficult |
| **Maintenance** | Easier to maintain smaller datasets | Resharding is complex and expensive |
| **Consistency** | Can achieve strong consistency per shard | Global consistency is harder |
| **Query Complexity** | Simple queries are very fast | Complex queries may need application logic |
| **Data Hotspots** | Can scale hot data independently | Requires monitoring and rebalancing |

---

## Partitioning vs Sharding Comparison

| Aspect | Partitioning | Sharding |
|--------|--------------|----------|
| **Definition** | Dividing table into partitions within same database | Distributing data across multiple databases |
| **Location** | Same database server | Different database servers |
| **Scalability** | Limited by single database capacity | Horizontal scaling across servers |
| **Query Performance** | Partition pruning speeds up queries | Reduced data per shard speeds up queries |
| **Multi-partition Queries** | Fast (same server) | Slow (scatter-gather across servers) |
| **Transactions** | ACID transactions across partitions | Difficult across shards |
| **Complexity** | Lower operational complexity | Higher operational complexity |
| **Resharding** | Relatively simple | Complex and expensive |
| **Use Case** | Large tables in single database | Data too large for single database |
| **Consistency** | Strong consistency | Eventual consistency (usually) |
| **Maintenance** | Easier backup/recovery | More complex maintenance |
| **Cost** | Single database infrastructure | Multiple database infrastructure |
| **When to Use** | Table has millions of rows | Data exceeds single database capacity |

---

## Key Takeaways

- **Relational**: Best for structured, transactional data with strong consistency
- **NoSQL**: Better for unstructured, high-volume data with flexible schemas
- **Indexes**: Critical for performance but require careful planning
- **ACID**: Ensures data reliability in critical systems
- **Specialized Databases**: Choose based on specific access patterns and data characteristics
- **Multi-Database Architecture**: Modern systems often use multiple databases for different purposes

--- 

<button onclick="window.scrollTo({top: 0, behavior: 'instant'})" style="position: fixed; bottom: 30px; right: 30px; z-index: 99; padding: 12px 16px; background-color: #007bff; color: white; border: none; border-radius: 50%; cursor: pointer; font-size: 18px; width: 50px; height: 50px; text-align: center; line-height: 24px; box-shadow: 0 4px 8px rgba(0, 0, 0, 0.2); font-weight: bold; transition: background-color 0.3s;" onmouseover="this.style.backgroundColor='#0056b3'" onmouseout="this.style.backgroundColor='#007bff'" onmousedown="this.style.backgroundColor='#003d82'" onmouseup="this.style.backgroundColor='#0056b3'" title="Go to top">↑</button>
