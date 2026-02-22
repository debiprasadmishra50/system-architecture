# Event Driven Systems: CQRS Pattern

CQRS: Command Query Responsibility Segregation

## Table of Contents
1. [What is CQRS](#what-is-cqrs)
2. [What Problem Does It Solve](#what-problem-does-it-solve)
3. [How It Works and How to Use It](#how-it-works-and-how-to-use-it)
4. [Approaches to Achieve CQRS](#approaches-to-achieve-cqrs)
5. [Challenges and Event Sourcing](#challenges-and-event-sourcing)
6. [Healthcare Platform Example](#healthcare-platform-example)
7. [Summary](#summary)
8. [Code Implementation Reference](#code-implementation-reference)


---

## What is CQRS

- CQRS (Command Query Responsibility Segregation) is an architectural pattern that separates the operations that read data (queries) from the operations that write data (commands). 

- Instead of using a single unified model for both reading and writing, CQRS advocates for using separate models optimized for each responsibility.

![image](../../Resources/09-cqrs-pattern/Screenshot%202026-02-06%20at%2011.57.47 PM.png)

**Key Concepts:**
- **Commands**: Operations that modify state (Create, Update, Delete)
- **Queries**: Operations that retrieve data without modifying state (Read)
- **Separation**: Different models, services, or even databases for commands and queries
- **Asynchronous Communication**: Often uses event-driven architecture to keep read and write models in sync

The pattern is particularly useful in complex domains where read and write operations have different performance, scalability, and consistency requirements.

---

## What Problem Does It Solve

### 1. **Performance Optimization**
- Read operations often outnumber write operations (typically 10:1 or higher)
- A single model optimized for both reads and writes is a compromise
- CQRS allows optimizing each model independently for its specific use case

### 2. **Scalability Mismatch**
- Read and write workloads have different scaling characteristics
- You can scale the read model independently from the write model
- Example: A reporting system needs massive read capacity but minimal write capacity

### 3. **Complex Query Requirements**
- Business queries often require denormalized data across multiple aggregates
- Maintaining denormalized views in a single model is complex and error-prone
- CQRS allows maintaining optimized read models specifically for queries

### 4. **Conflicting Data Models**
- The model optimized for transactional writes (normalized, ACID) differs from the model optimized for reporting (denormalized, fast reads)
- CQRS eliminates the need to compromise between these two

### 5. **Audit and Compliance**
- Separating commands from queries makes it easier to audit all state changes
- Commands naturally become an audit trail
- Simplifies compliance requirements in regulated industries

### 6. **Team Productivity**
- Different teams can work on read and write logic independently
- Reduces merge conflicts and coordination overhead
- Allows specialized optimization for each concern

---

## How It Works and How to Use It

|  ![image](../../Resources/09-cqrs-pattern/Screenshot%202026-02-07%20at%2012.01.01 AM.png)   |   &  |   ![image](../../Resources/09-cqrs-pattern/Screenshot%202026-02-07%20at%2012.01.32 AM.png)  |
| :---: | :---: | :---: |


### Architecture Overview

```
┌─────────────────────────────────────────────────────────────┐
│                     Client Application                      │
└──────────────────┬──────────────────────────────────────────┘
                   │
        ┌──────────┴──────────┐
        │                     │
        ▼                     ▼
   ┌─────────┐           ┌─────────┐
   │ Commands│           │ Queries │
   └────┬────┘           └────┬────┘
        │                     │
        ▼                     ▼
   ┌──────────────┐      ┌──────────────┐
   │ Write Model  │      │ Read Model   │
   │ (Normalized) │      │ (Denormalized)
   └────┬─────────┘      └──────────────┘
        │                     ▲
        ▼                     │
   ┌──────────────┐      ┌────────────┐
   │ Write DB     │      │ Read DB    │
   │ (Source of   │      │ (Optimized │
   │  Truth)      │      │  for reads)│
   └──────────────┘      └────────────┘
        │
        ▼
   ┌──────────────┐
   │ Event Bus    │
   │ (Sync Models)│
   └──────────────┘
```

### Implementation Steps

**1. Identify Commands and Queries**
```
Commands:
- CreatePatient(name, email, phone)
- UpdatePatientRecord(patientId, data)
- ScheduleAppointment(patientId, doctorId, date)

Queries:
- GetPatientDetails(patientId)
- GetPatientAppointmentHistory(patientId)
- GetDoctorAvailability(doctorId)
```

**2. Create Separate Write Model**
- Normalize data for transactional consistency
- Implement business logic and validation
- Ensure ACID properties
- Use traditional relational database

**3. Create Separate Read Model**
- Denormalize data for query performance
- Optimize for specific query patterns
- Can use different database technology (NoSQL, search engines, etc.)
- Designed for fast retrieval

**4. Implement Synchronization**
- Use events to propagate changes from write model to read model
- Event handlers update read model asynchronously
- Maintain eventual consistency

**5. Handle Eventual Consistency**
- Accept that read model may be slightly behind write model
- Implement strategies for handling stale reads
- Provide mechanisms to refresh data when needed

### Usage Pattern

```
Write Operation Flow:
1. Client sends Command
2. Command Handler validates and executes
3. Changes persisted to Write DB
4. Event published to Event Bus
5. Event Handlers update Read DB asynchronously

Read Operation Flow:
1. Client sends Query
2. Query Handler retrieves from Read DB
3. Returns denormalized data immediately
```

---

## Approaches to Achieve CQRS

### Approach 1: Single Database with Separate Models
![image](../../Resources/09-cqrs-pattern/Screenshot%202026-02-07%20at%2012.01.01 AM.png)

**Architecture:**
- One physical database
- Separate logical models for reads and writes
- Synchronization through application layer

**Advantages:**
- Simpler deployment and operations
- Single source of truth
- Easier transaction management
- Lower infrastructure cost

**Disadvantages:**
- Limited scalability separation
- Database must support both normalized and denormalized views
- Less flexibility in technology choices

**When to Use:**
- Moderate read/write ratio
- Limited budget for infrastructure
- Simpler domain models
- Team prefers operational simplicity

**Example Implementation:**
```
Write Model: Normalized tables (Patients, Doctors, Appointments)
Read Model: Materialized views or denormalized tables
Sync: Database triggers or application-level event handlers
```

### Approach 2: Multiple Databases
![image](../../Resources/09-cqrs-pattern/Screenshot%202026-02-07%20at%2012.01.32 AM.png)

**Architecture:**
- Separate write database (source of truth)
- Separate read database(s) optimized for queries
- Event-driven synchronization between them

**Advantages:**
- Independent scalability of read and write
- Technology flexibility (SQL for writes, NoSQL for reads)
- Better performance optimization
- Easier to add multiple read models for different use cases

**Disadvantages:**
- Operational complexity
- Eventual consistency challenges
- Higher infrastructure cost
- Distributed system challenges

**When to Use:**
- High read/write ratio (100:1 or more)
- Complex query requirements
- Need for independent scaling
- Multiple teams working on different concerns

**Example Implementation:**
```
Write DB: PostgreSQL (normalized, ACID)
Read DB 1: Elasticsearch (full-text search)
Read DB 2: MongoDB (document-oriented queries)
Read DB 3: Redis (caching layer)
Sync: Event Bus (Kafka, RabbitMQ) with event handlers
```

### Comparison Table

| Aspect | Single Database | Multiple Databases |
|--------|-----------------|-------------------|
| **Complexity** | Low | High |
| **Scalability** | Limited | Excellent |
| **Cost** | Low | High |
| **Consistency** | Strong | Eventual |
| **Technology Flexibility** | Limited | High |
| **Operational Overhead** | Low | High |
| **Read Performance** | Good | Excellent |
| **Write Performance** | Good | Good |
| **Best For** | Small to medium systems | Large, complex systems |

---

## Challenges and Event Sourcing

### Challenges of CQRS

**1. Eventual Consistency**
- Read model may lag behind write model
- Clients may see stale data
- Requires careful handling of user expectations

**2. Increased Complexity**
- More moving parts to manage
- Distributed system challenges
- Harder to debug and trace issues

**3. Synchronization Issues**
- Events may be lost or duplicated
- Handling out-of-order events
- Ensuring idempotency of event handlers

**4. Operational Overhead**
- Multiple databases to maintain
- Event bus infrastructure
- Monitoring and alerting complexity

**5. Testing Difficulty**
- Asynchronous behavior harder to test
- Race conditions in eventual consistency
- Complex integration testing

**6. Data Consistency Guarantees**
- No ACID transactions across models
- Compensating transactions needed for failures
- Reconciliation mechanisms required

### Event Sourcing as a Solution

**Event Sourcing** is a complementary pattern that stores the state of an entity as a sequence of immutable events rather than storing the current state directly.

**How Event Sourcing Solves CQRS Challenges:**

| Challenge | Event Sourcing Solution |
|-----------|------------------------|
| **Eventual Consistency** | Complete audit trail ensures eventual consistency is achievable; events are the source of truth |
| **Synchronization Issues** | Events are immutable and ordered; replay ensures consistency; idempotency is built-in |
| **Data Loss** | Event log is append-only; no data is lost; complete history preserved |
| **Audit Trail** | Events naturally provide complete audit trail of all changes |
| **Debugging** | Can replay events to understand exact sequence of state changes |
| **Recovery** | Rebuild read models by replaying events from event store |

**Event Sourcing Benefits for CQRS:**
- **Single Source of Truth**: Event log is the authoritative source
- **Temporal Queries**: Can query state at any point in time
- **Audit Compliance**: Complete immutable record of all changes
- **Debugging**: Replay events to understand system behavior
- **Scalability**: Read models can be rebuilt independently
- **Resilience**: Recover from failures by replaying events

**Simple Event Sourcing Flow:**
```
1. Command received: "UpdatePatientRecord"
2. Validate command
3. Generate event: "PatientRecordUpdated"
4. Append event to Event Store (immutable log)
5. Event handlers process event
6. Update Read Model
7. Return success to client
```

---

## Healthcare Platform Example

### Scenario: Patient Management System

A healthcare platform needs to manage patient records, appointments, and medical history. The system has:
- High read volume (doctors, nurses, patients checking records)
- Moderate write volume (updates to records, new appointments)
- Complex queries (patient history, appointment availability, billing reports)
- Strict compliance requirements (HIPAA, audit trails)

### CQRS Implementation

**Write Model (Source of Truth):**
```
Database: PostgreSQL
Tables:
- patients (id, name, email, phone, created_at)
- medical_records (id, patient_id, record_type, data, created_at)
- appointments (id, patient_id, doctor_id, date, status)

Commands:
- RegisterPatient(name, email, phone)
- UpdateMedicalRecord(patientId, recordType, data)
- ScheduleAppointment(patientId, doctorId, dateTime)
- CancelAppointment(appointmentId)
```

**Read Models (Optimized for Queries):**

**Read Model 1: Patient Dashboard (MongoDB)**
```
{
  patientId: "P123",
  name: "John Doe",
  email: "john@example.com",
  upcomingAppointments: [
    {
      appointmentId: "A456",
      doctorName: "Dr. Smith",
      date: "2026-02-15",
      type: "Checkup"
    }
  ],
  recentRecords: [
    {
      date: "2026-02-01",
      type: "Lab Results",
      summary: "All normal"
    }
  ],
  lastUpdated: "2026-02-06T18:34:30Z"
}
```

**Read Model 2: Doctor Availability (Redis Cache)**
```
doctor:D789:availability:2026-02-15 = [
  "09:00", "09:30", "10:00", "14:00", "14:30"
]
```

**Read Model 3: Audit Log (Elasticsearch)**
```
{
  timestamp: "2026-02-06T18:34:30Z",
  eventType: "PatientRecordUpdated",
  patientId: "P123",
  userId: "U456",
  changes: {
    field: "medicalHistory",
    oldValue: "...",
    newValue: "..."
  },
  ipAddress: "192.168.1.1"
}
```

### Event Flow Example

**Scenario: Doctor updates patient's medical record**

```
1. Doctor submits command:
   UpdateMedicalRecord(patientId: "P123", type: "Diagnosis", data: {...})

2. Write Service validates and executes:
   - Verify doctor has access to patient
   - Validate medical data format
   - Check HIPAA compliance
   - Save to PostgreSQL

3. Event published to Event Bus:
   {
     eventId: "E789",
     eventType: "MedicalRecordUpdated",
     patientId: "P123",
     doctorId: "D456",
     recordType: "Diagnosis",
     timestamp: "2026-02-06T18:34:30Z",
     data: {...}
   }

4. Event Handlers process event:
   
   Handler 1 (Patient Dashboard Updater):
   - Fetch updated record from write DB
   - Update MongoDB patient document
   - Invalidate cache
   
   Handler 2 (Audit Logger):
   - Index event in Elasticsearch
   - Ensure compliance logging
   
   Handler 3 (Notification Service):
   - Send notification to patient
   - Update notification queue

5. Read Models eventually consistent:
   - Patient sees updated record in dashboard
   - Audit trail shows complete history
   - All queries return consistent data
```

### Benefits in Healthcare Context

| Benefit | Application |
|---------|-------------|
| **Audit Trail** | Complete immutable record of all medical record changes (HIPAA compliance) |
| **Performance** | Doctors get instant access to patient dashboards without waiting for complex joins |
| **Scalability** | Handle thousands of concurrent reads without impacting write performance |
| **Compliance** | Event sourcing provides proof of who changed what and when |
| **Resilience** | Rebuild read models if corrupted; replay events to recover |
| **Analytics** | Separate analytics database for reporting without impacting operational system |
| **Temporal Queries** | View patient's medical history at any point in time |

### Challenges Addressed

- **Eventual Consistency**: Acceptable in healthcare; read model updates within seconds
- **Synchronization**: Event sourcing ensures no data loss; complete audit trail
- **Compliance**: Events provide immutable proof of all changes
- **Recovery**: Rebuild read models by replaying events if needed
- **Debugging**: Replay events to understand exact sequence of medical record changes

---

## Summary

CQRS is a powerful pattern for systems with complex read/write requirements. By separating command and query responsibilities, it enables:
- Independent optimization of read and write paths
- Better scalability and performance
- Clearer separation of concerns
- Improved compliance and auditability

When combined with Event Sourcing, CQRS becomes even more powerful, providing:
- Complete audit trails
- Temporal queries
- Resilience and recovery capabilities
- Strong consistency guarantees through event replay

The healthcare platform example demonstrates how CQRS solves real-world problems in regulated industries where audit trails, performance, and compliance are critical requirements.

---

## Code Implementation Reference

### 1. Command Handler (Node.js/TypeScript)

```typescript
// Command Definition
interface UpdatePatientRecordCommand {
  patientId: string;
  recordType: string;
  data: Record<string, any>;
  userId: string;
}

// Command Handler
class UpdatePatientRecordHandler {
  constructor(
    private writeDb: Database,
    private eventBus: EventBus
  ) {}

  async handle(command: UpdatePatientRecordCommand): Promise<void> {
    // 1. Validate command
    if (!command.patientId || !command.recordType) {
      throw new Error('Invalid command');
    }

    // 2. Execute business logic
    const patient = await this.writeDb.query(
      'SELECT * FROM patients WHERE id = ?',
      [command.patientId]
    );

    if (!patient) {
      throw new Error('Patient not found');
    }

    // 3. Persist to write database
    const result = await this.writeDb.query(
      'INSERT INTO medical_records (patient_id, record_type, data, created_at) VALUES (?, ?, ?, NOW())',
      [command.patientId, command.recordType, JSON.stringify(command.data)]
    );

    // 4. Publish event
    const event = {
      eventId: generateId(),
      eventType: 'PatientRecordUpdated',
      patientId: command.patientId,
      recordType: command.recordType,
      userId: command.userId,
      timestamp: new Date().toISOString(),
      data: command.data
    };

    await this.eventBus.publish('patient-events', event);
  }
}
```

### 2. Query Handler (Node.js/TypeScript)

```typescript
// Query Definition
interface GetPatientDashboardQuery {
  patientId: string;
}

// Query Handler
class GetPatientDashboardHandler {
  constructor(private readDb: MongoClient) {}

  async handle(query: GetPatientDashboardQuery): Promise<PatientDashboard> {
    // Query read model directly (no joins, no complex logic)
    const dashboard = await this.readDb
      .collection('patient_dashboards')
      .findOne({ patientId: query.patientId });

    if (!dashboard) {
      throw new Error('Patient dashboard not found');
    }

    return dashboard;
  }
}
```

### 3. Event Handler (Node.js/TypeScript)

```typescript
// Event Handler - Updates Read Model
class PatientRecordUpdatedEventHandler {
  constructor(
    private readDb: MongoClient,
    private writeDb: Database
  ) {}

  async handle(event: PatientRecordUpdatedEvent): Promise<void> {
    // 1. Fetch latest data from write database
    const medicalRecords = await this.writeDb.query(
      'SELECT * FROM medical_records WHERE patient_id = ? ORDER BY created_at DESC LIMIT 10',
      [event.patientId]
    );

    const patient = await this.writeDb.query(
      'SELECT * FROM patients WHERE id = ?',
      [event.patientId]
    );

    // 2. Transform to read model format
    const dashboard = {
      patientId: event.patientId,
      name: patient.name,
      email: patient.email,
      recentRecords: medicalRecords.map(r => ({
        date: r.created_at,
        type: r.record_type,
        summary: r.data.summary
      })),
      lastUpdated: new Date().toISOString()
    };

    // 3. Update read database
    await this.readDb
      .collection('patient_dashboards')
      .updateOne(
        { patientId: event.patientId },
        { $set: dashboard },
        { upsert: true }
      );

    // 4. Invalidate cache
    await this.cache.del(`patient:${event.patientId}:dashboard`);
  }
}
```

### 4. Event Bus Setup (Kafka Example)

```typescript
// Event Bus Implementation
class KafkaEventBus implements EventBus {
  private producer: KafkaProducer;
  private consumer: KafkaConsumer;
  private handlers: Map<string, EventHandler[]> = new Map();

  async publish(topic: string, event: Event): Promise<void> {
    await this.producer.send({
      topic,
      messages: [
        {
          key: event.patientId,
          value: JSON.stringify(event),
          headers: {
            'event-type': event.eventType,
            'timestamp': event.timestamp
          }
        }
      ]
    });
  }

  async subscribe(eventType: string, handler: EventHandler): Promise<void> {
    if (!this.handlers.has(eventType)) {
      this.handlers.set(eventType, []);
    }
    this.handlers.get(eventType)!.push(handler);
  }

  async startConsuming(): Promise<void> {
    await this.consumer.subscribe({ topics: ['patient-events'] });
    
    await this.consumer.run({
      eachMessage: async ({ topic, partition, message }) => {
        const event = JSON.parse(message.value.toString());
        const handlers = this.handlers.get(event.eventType) || [];
        
        for (const handler of handlers) {
          try {
            await handler.handle(event);
          } catch (error) {
            console.error(`Error handling event: ${error}`);
            // Implement retry logic or dead letter queue
          }
        }
      }
    });
  }
}
```

### 5. API Endpoints

```typescript
// Express API Setup
app.post('/api/patients/:patientId/records', async (req, res) => {
  const command: UpdatePatientRecordCommand = {
    patientId: req.params.patientId,
    recordType: req.body.recordType,
    data: req.body.data,
    userId: req.user.id
  };

  try {
    const handler = new UpdatePatientRecordHandler(writeDb, eventBus);
    await handler.handle(command);
    res.status(201).json({ success: true });
  } catch (error) {
    res.status(400).json({ error: error.message });
  }
});

// Query endpoint - reads from read model
app.get('/api/patients/:patientId/dashboard', async (req, res) => {
  const query: GetPatientDashboardQuery = {
    patientId: req.params.patientId
  };

  try {
    const handler = new GetPatientDashboardHandler(readDb);
    const dashboard = await handler.handle(query);
    res.json(dashboard);
  } catch (error) {
    res.status(404).json({ error: error.message });
  }
});
```

### 6. Idempotent Event Handler (Handling Duplicates)

```typescript
// Ensure event is processed only once
class IdempotentEventHandler {
  constructor(
    private readDb: MongoClient,
    private eventStore: Database
  ) {}

  async handle(event: Event): Promise<void> {
    // 1. Check if event already processed
    const processed = await this.eventStore.query(
      'SELECT * FROM processed_events WHERE event_id = ?',
      [event.eventId]
    );

    if (processed) {
      console.log(`Event ${event.eventId} already processed, skipping`);
      return;
    }

    try {
      // 2. Process event
      await this.updateReadModel(event);

      // 3. Mark as processed
      await this.eventStore.query(
        'INSERT INTO processed_events (event_id, processed_at) VALUES (?, NOW())',
        [event.eventId]
      );
    } catch (error) {
      console.error(`Error processing event ${event.eventId}: ${error}`);
      throw error;
    }
  }

  private async updateReadModel(event: Event): Promise<void> {
    // Update read model logic here
  }
}
```

### 7. Handling Eventual Consistency

```typescript
// Client-side handling of eventual consistency
async function getPatientDashboardWithRetry(
  patientId: string,
  maxRetries: number = 3
): Promise<PatientDashboard> {
  for (let attempt = 1; attempt <= maxRetries; attempt++) {
    try {
      const dashboard = await queryHandler.handle({
        patientId
      });

      // Check if data is fresh enough
      const timeSinceUpdate = Date.now() - new Date(dashboard.lastUpdated).getTime();
      if (timeSinceUpdate < 5000) { // Less than 5 seconds old
        return dashboard;
      }

      // If stale, wait and retry
      if (attempt < maxRetries) {
        await sleep(1000 * attempt); // Exponential backoff
        continue;
      }
    } catch (error) {
      if (attempt === maxRetries) throw error;
      await sleep(1000 * attempt);
    }
  }

  throw new Error('Failed to get fresh patient dashboard');
}
```

### 8. Event Sourcing - Event Store

```typescript
// Event Store Implementation
class EventStore {
  constructor(private db: Database) {}

  async append(event: Event): Promise<void> {
    await this.db.query(
      `INSERT INTO events
       (event_id, aggregate_id, event_type, data, timestamp, version)
       VALUES (?, ?, ?, ?, ?, ?)`,
      [
        event.eventId,
        event.patientId,
        event.eventType,
        JSON.stringify(event),
        event.timestamp,
        event.version
      ]
    );
  }

  async getEvents(patientId: string, fromVersion: number = 0): Promise<Event[]> {
    const events = await this.db.query(
      `SELECT * FROM events
       WHERE aggregate_id = ? AND version > ?
       ORDER BY version ASC`,
      [patientId, fromVersion]
    );

    return events.map(e => JSON.parse(e.data));
  }

  async rebuildReadModel(patientId: string): Promise<void> {
    // Get all events for patient
    const events = await this.getEvents(patientId, 0);

    // Replay events to rebuild state
    let state = this.getInitialState();
    for (const event of events) {
      state = this.applyEvent(state, event);
    }

    // Update read model with rebuilt state
    await this.updateReadModel(patientId, state);
  }

  private applyEvent(state: any, event: Event): any {
    switch (event.eventType) {
      case 'PatientCreated':
        return { ...state, id: event.patientId, name: event.data.name };
      case 'PatientRecordUpdated':
        return {
          ...state,
          records: [...(state.records || []), event.data]
        };
      default:
        return state;
    }
  }

  private getInitialState(): any {
    return { records: [] };
  }

  private async updateReadModel(patientId: string, state: any): Promise<void> {
    // Update MongoDB or other read database
  }
}
```

### Key Takeaways for Implementation

1. **Separate Command and Query Handlers**: Keep write and read logic completely separate
2. **Use Events as Communication**: Events are the bridge between write and read models
3. **Ensure Idempotency**: Handle duplicate events gracefully
4. **Implement Retry Logic**: Account for eventual consistency delays
5. **Monitor Event Processing**: Track event lag and processing failures
6. **Version Your Events**: Include version info for schema evolution
7. **Use Event Sourcing**: Store complete event history for audit and recovery
8. **Handle Failures Gracefully**: Implement dead letter queues and compensation logic

---

<button onclick="window.scrollTo({top: 0, behavior: 'instant'})" style="position: fixed; bottom: 30px; right: 30px; z-index: 99; padding: 12px 16px; background-color: #007bff; color: white; border: none; border-radius: 50%; cursor: pointer; font-size: 18px; width: 50px; height: 50px; text-align: center; line-height: 24px; box-shadow: 0 4px 8px rgba(0, 0, 0, 0.2); font-weight: bold; transition: background-color 0.3s;" onmouseover="this.style.backgroundColor='#0056b3'" onmouseout="this.style.backgroundColor='#007bff'" onmousedown="this.style.backgroundColor='#003d82'" onmouseup="this.style.backgroundColor='#0056b3'" title="Go to top">↑</button>
