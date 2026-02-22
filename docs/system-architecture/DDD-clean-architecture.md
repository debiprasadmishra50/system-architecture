# Domain Driven Design & Clean Architecture

## Table of Contents
1. [What is Domain Driven Design and Clean Architecture](#what-is-domain-driven-design-and-clean-architecture)
2. [What Problem Does It Solve](#what-problem-does-it-solve)
3. [Where to Use It and How](#where-to-use-it-and-how)
4. [Node.js Example](#nodejs-example)

---

## What is Domain Driven Design and Clean Architecture

### Domain Driven Design (DDD)

Domain Driven Design is a software development approach that emphasizes understanding and modeling the business domain at the core of your application. 

It was introduced by Eric Evans and focuses on:

- **Ubiquitous Language**: A shared vocabulary between developers and domain experts that is reflected in the code
- **Bounded Contexts**: Clear boundaries around different parts of the domain, each with its own model
- **Entities and Value Objects**: Core building blocks that represent domain concepts
- **Aggregates**: Clusters of entities and value objects that are treated as a single unit
- **Domain Events**: Events that represent something important that happened in the domain
- **Repositories**: Abstractions for accessing aggregates
- **Services**: Domain logic that doesn't naturally fit into entities or value objects


### Clean Architecture

![image](../../Resources/clean-architecture/Screenshot%202026-02-08%20at%209.55.58 PM.png)

Clean Architecture is an architectural pattern that emphasizes separation of concerns and independence from frameworks, databases, and UI. Key principles include:

- **Independence from Frameworks**: The business logic is not tied to any specific framework
- **Testability**: Business rules can be tested without UI, database, or web server
- **Independence from UI**: The UI can change without affecting the business logic
- **Independence from Database**: Business logic doesn't depend on database implementation
- **Independence from External Agencies**: Business rules are isolated from external systems

The architecture typically consists of concentric layers:
1. **Entities Layer**: Core business rules and entities
2. **Use Cases Layer**: Application-specific business rules
3. **Interface Adapters Layer**: Controllers, gateways, presenters
4. **Frameworks & Drivers Layer**: Web, database, UI frameworks

---

## What Problem Does It Solve

### Problems Addressed

1. **Complexity Management**: Large applications become difficult to understand and maintain. DDD + Clean Architecture provides structure and clear organization.

2. **Tight Coupling**: Code tightly coupled to frameworks and databases is hard to test and modify. Clean Architecture enforces loose coupling through dependency inversion.

3. **Business Logic Scattered**: Business rules spread across controllers, models, and services become hard to locate and maintain. DDD centralizes domain logic.

4. **Poor Communication**: Developers and domain experts speak different languages. Ubiquitous Language bridges this gap.

5. **Difficult Testing**: When business logic is mixed with infrastructure code, unit testing becomes challenging. Clean Architecture enables easy testing.

6. **Unclear Domain Boundaries**: Without clear boundaries, features bleed into each other. Bounded Contexts in DDD define clear separation.

7. **Maintenance Nightmares**: Changes in one area unexpectedly break other areas. Proper architecture prevents this through isolation.

---

## Where to Use It and How

### When to Use

- **Complex Business Domains**: Applications with intricate business rules and logic
- **Long-lived Projects**: Systems that need to evolve over years
- **Team Collaboration**: Projects where multiple teams work on different features
- **Microservices**: Each microservice can have its own bounded context
- **Enterprise Applications**: Large-scale systems with complex requirements

### When NOT to Use

- **Simple CRUD Applications**: Overkill for basic data management applications
- **Rapid Prototypes**: Too much overhead for quick proof-of-concepts
- **Small Projects**: Unnecessary complexity for small codebases

### How to Implement

1. **Identify Bounded Contexts**: Map out different areas of your domain
2. **Define Ubiquitous Language**: Establish shared terminology with stakeholders
3. **Model Entities and Value Objects**: Create domain models that represent business concepts
4. **Create Aggregates**: Group related entities and value objects
5. **Implement Repositories**: Abstract data access
6. **Separate Layers**: Keep domain logic separate from infrastructure
7. **Use Dependency Injection**: Inject dependencies to maintain loose coupling
8. **Write Tests**: Test business logic independently from infrastructure

---

## Node.js Example

### Project Structure

```
src/
├── domain/
│   ├── entities/
│   │   └── User.js
│   ├── value-objects/
│   │   └── Email.js
│   └── repositories/
│       └── UserRepository.js
├── application/
│   └── use-cases/
│       └── CreateUserUseCase.js
├── infrastructure/
│   ├── persistence/
│   │   └── InMemoryUserRepository.js
│   └── http/
│       └── UserController.js
└── index.js
```

### Implementation

#### Domain Layer - Value Object (Email.js)

```javascript
class Email {
  constructor(value) {
    if (!this.isValidEmail(value)) {
      throw new Error('Invalid email format');
    }
    this.value = value;
  }

  isValidEmail(email) {
    const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
    return emailRegex.test(email);
  }

  equals(other) {
    return this.value === other.value;
  }

  toString() {
    return this.value;
  }
}

module.exports = Email;
```

#### Domain Layer - Entity (User.js)

```javascript
const Email = require('../value-objects/Email');

class User {
  constructor(id, name, email) {
    this.id = id;
    this.name = name;
    this.email = new Email(email);
  }

  changeName(newName) {
    this.name = newName;
  }

  changeEmail(newEmail) {
    this.email = new Email(newEmail);
  }

  static create(id, name, email) {
    return new User(id, name, email);
  }
}

module.exports = User;
```

#### Domain Layer - Repository Interface (UserRepository.js)

```javascript
class UserRepository {
  async save(user) {
    throw new Error('save() must be implemented');
  }

  async findById(id) {
    throw new Error('findById() must be implemented');
  }

  async findByEmail(email) {
    throw new Error('findByEmail() must be implemented');
  }
}

module.exports = UserRepository;
```

#### Application Layer - Use Case (CreateUserUseCase.js)

```javascript
const User = require('../../domain/entities/User');

class CreateUserUseCase {
  constructor(userRepository) {
    this.userRepository = userRepository;
  }

  async execute(request) {
    const { id, name, email } = request;

    // Check if user already exists
    const existingUser = await this.userRepository.findByEmail(email);
    if (existingUser) {
      throw new Error('User with this email already exists');
    }

    // Create new user
    const user = User.create(id, name, email);

    // Save user
    await this.userRepository.save(user);

    return {
      id: user.id,
      name: user.name,
      email: user.email.toString()
    };
  }
}

module.exports = CreateUserUseCase;
```

#### Infrastructure Layer - Repository Implementation (InMemoryUserRepository.js)

```javascript
const UserRepository = require('../../domain/repositories/UserRepository');

class InMemoryUserRepository extends UserRepository {
  constructor() {
    super();
    this.users = new Map();
  }

  async save(user) {
    this.users.set(user.id, user);
  }

  async findById(id) {
    return this.users.get(id) || null;
  }

  async findByEmail(email) {
    for (const user of this.users.values()) {
      if (user.email.equals(email)) {
        return user;
      }
    }
    return null;
  }
}

module.exports = InMemoryUserRepository;
```

#### Infrastructure Layer - HTTP Controller (UserController.js)

```javascript
class UserController {
  constructor(createUserUseCase) {
    this.createUserUseCase = createUserUseCase;
  }

  async create(req, res) {
    try {
      const { id, name, email } = req.body;

      const result = await this.createUserUseCase.execute({
        id,
        name,
        email
      });

      res.status(201).json(result);
    } catch (error) {
      res.status(400).json({ error: error.message });
    }
  }
}

module.exports = UserController;
```

#### Main Application (index.js)

```javascript
const express = require('express');
const InMemoryUserRepository = require('./infrastructure/persistence/InMemoryUserRepository');
const CreateUserUseCase = require('./application/use-cases/CreateUserUseCase');
const UserController = require('./infrastructure/http/UserController');

const app = express();
app.use(express.json());

// Setup dependencies
const userRepository = new InMemoryUserRepository();
const createUserUseCase = new CreateUserUseCase(userRepository);
const userController = new UserController(createUserUseCase);

// Routes
app.post('/users', (req, res) => userController.create(req, res));

// Start server
const PORT = 3000;
app.listen(PORT, () => {
  console.log(`Server running on port ${PORT}`);
});

module.exports = app;
```

#### Testing (example)

```javascript
const CreateUserUseCase = require('../src/application/use-cases/CreateUserUseCase');
const InMemoryUserRepository = require('../src/infrastructure/persistence/InMemoryUserRepository');

describe('CreateUserUseCase', () => {
  let useCase;
  let repository;

  beforeEach(() => {
    repository = new InMemoryUserRepository();
    useCase = new CreateUserUseCase(repository);
  });

  test('should create a user successfully', async () => {
    const result = await useCase.execute({
      id: '1',
      name: 'John Doe',
      email: 'john@example.com'
    });

    expect(result.id).toBe('1');
    expect(result.name).toBe('John Doe');
    expect(result.email).toBe('john@example.com');
  });

  test('should throw error for invalid email', async () => {
    await expect(
      useCase.execute({
        id: '1',
        name: 'John Doe',
        email: 'invalid-email'
      })
    ).rejects.toThrow('Invalid email format');
  });

  test('should throw error if user already exists', async () => {
    await useCase.execute({
      id: '1',
      name: 'John Doe',
      email: 'john@example.com'
    });

    await expect(
      useCase.execute({
        id: '2',
        name: 'Jane Doe',
        email: 'john@example.com'
      })
    ).rejects.toThrow('User with this email already exists');
  });
});
```

### Key Benefits in This Example

1. **Testability**: Business logic (CreateUserUseCase) is tested without HTTP or database
2. **Flexibility**: Can swap InMemoryUserRepository with a database implementation
3. **Clear Separation**: Domain logic is isolated from infrastructure
4. **Maintainability**: Each layer has a single responsibility
5. **Scalability**: Easy to add new use cases and features

--- 

<button onclick="window.scrollTo({top: 0, behavior: 'instant'})" style="position: fixed; bottom: 30px; right: 30px; z-index: 99; padding: 12px 16px; background-color: #007bff; color: white; border: none; border-radius: 50%; cursor: pointer; font-size: 18px; width: 50px; height: 50px; text-align: center; line-height: 24px; box-shadow: 0 4px 8px rgba(0, 0, 0, 0.2); font-weight: bold; transition: background-color 0.3s;" onmouseover="this.style.backgroundColor='#0056b3'" onmouseout="this.style.backgroundColor='#007bff'" onmousedown="this.style.backgroundColor='#003d82'" onmouseup="this.style.backgroundColor='#0056b3'" title="Go to top">↑</button>
