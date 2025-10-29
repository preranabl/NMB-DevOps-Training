---
marp: true

---
# Section 4
## Architecture of a 3-Tier Application
### 1. Overview: Frontend, Backend, and Database (The Three Tiers)
**Definition:**
A 3-tier application architecture is a software design pattern that divides an application into three logical and physical layers (or tiers) — each responsible for a specific function in the system.

It’s the most common architecture used in enterprise systems, web apps, and microservices-based solutions.

---
These tiers are:

**A. Presentation Tier (Frontend)**

**B. Application Tier (Backend / Business Logic)**

**C. Data Tier (Database / Storage)**

Each tier can be developed, deployed, and scaled independently providing flexibility, modularity, and performance benefits.

---
#### A. Presentation Tier (Frontend Layer)

**Purpose:**
This is the user interface — where users interact with the application.
It handles displaying information to the user and collecting input.

**Characteristics:**
- Typically built using HTML, CSS, JavaScript, or modern frameworks like React, Angular, or Vue.js.
- Communicates with the backend using APIs (usually REST or GraphQL).
- Runs in browsers or mobile apps.

**Example:**
The frontend displays account balances, forms, and dashboards that users see on their screen.It sends a request like “Get account balance” to the backend through an API.

---
#### B. Application Tier (Backend / Business Logic Layer)
**Purpose:**
The backend is the brain of the application.It processes user requests, applies business rules, performs calculations, and interacts with the database.

**Characteristics:**
- Built using programming languages such as Python (Flask, Django), Node.js (Express), Java (Spring Boot), or .NET.
- Implements application logic, authentication, data validation, and security checks.
- Communicates with both the frontend and the database.

--- 

**Example:**
When the frontend sends a request to fetch account details, the backend:
- Receives the request (e.g., via an API endpoint).
- Validates the user session.
- Queries the database for account data.
- Sends a formatted response (usually in JSON) back to the frontend.

#### C. Data Tier (Database Layer)

**Purpose:**
The database tier is responsible for data storage, retrieval, and management.
It ensures data integrity, security, and availability.

---
**Characteristics:**

- Common databases include MySQL, PostgreSQL, MongoDB, Oracle, or Redis (for caching).
- Handles transactions, relationships, and backup/recovery mechanisms.
- Only the backend should access this layer directly (not the frontend).

**Example:**
The database stores user credentials, account balances, transaction history, and session data for the banking application.

---
#### Architecture 
#### Diagram: 
![bg center 55%](3tier.png)

---
### 2. Benefits of Decoupled Architecture (Points)

- Clear separation of concerns – frontend, backend, and database have distinct roles.

- Independent development – teams can work on different layers simultaneously.

- Scalability – each layer can scale independently based on demand.

- Reusability and modularity – one backend can serve multiple frontends.

- Enhanced security – layered design with HTTPS, authentication, and isolation.

- Easier maintenance and updates – layers can be updated without affecting others.

- Better performance – workload distributed across frontend, backend, and database.

- Cloud & DevOps friendly – aligns with Docker, Kubernetes, and microservices.

---

### 3. Communication Between Tiers

**1. Frontend to Backend Communication**

- The frontend communicates with the backend using API calls, typically over HTTP or HTTPS.
- The most common pattern is RESTful APIs, though GraphQL and gRPC are also used.
- Data is exchanged in lightweight formats like JSON or XML.

**2. Backend to Database Communication**

The backend communicates with the database using SQL queries (for relational DBs) or query languages/APIs (for NoSQL DBs).

---
This layer handles:
1. Query execution
2. Data transformation
3. Connection pooling
4. Error handling

**3. Frontend to  Database (Direct Communication?)**
- The frontend should never talk directly to the database.
- Direct database access would expose sensitive information and credentials, bypassing authentication and validation layers.
- Instead, all communication must pass through the backend API.
---
### # Communication Technologies Used

**Frontend to Backend**: REST APIs, HTTPS, WebSockets, GraphQL
**Backend to Database**:	SQL, JDBC/ODBC, ORM frameworks (like Sequelize, Hibernate)
**Internal Caching**:	Redis, Memcached, In-memory caching
**Message Passing**: 	Kafka, RabbitMQ, SQS (for async communication)