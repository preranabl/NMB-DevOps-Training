---
marp: true

---
# Section 6
## Service Decomposition & DDD
### 1. What is monolith vs microservices
**Definition**:
**Monolith**:
A monolithic application is built as a single deployable unit. All functionality (UI, business logic, data access, background jobs) runs in one process or tightly-coupled set of processes packaged and deployed together.


---


**Microservices**
Microservices break an application into many small, independently-deployable services. Each service owns a single business capability, runs in its own process/container, has its own data (or at least its own data model), and communicates with other services over lightweight APIs (HTTP/gRPC, message queue)
#### Diagram:
<div align="center">
  <img src="mono-micro.png" width="50%">
</div>

---


| **Aspect** | **Monolithic Architecture** | **Microservices Architecture** |
|-------------|-----------------------------|--------------------------------|
| **Definition** | Single unified application where all components are tightly integrated and run as one unit. | Application is divided into multiple small, independent services that communicate through APIs. |
| **Codebase** | One large codebase for the entire application. | Separate codebases for each service or module. |
| **Deployment** | Deployed as a single package (WAR, JAR, or container). | Each service can be deployed independently. |
| **Scalability** | Scaled as a whole — even if only one part needs more resources. | Each microservice can be scaled independently based on demand. |

---
| **Aspect** | **Monolithic Architecture** | **Microservices Architecture** |
|-------------|-----------------------------|--------------------------------|
| **Development Speed** | Fast for small teams and simple projects. | Faster for large teams since services can be developed in parallel. |
| **Technology Stack** | Usually one common tech stack for the entire system. | Polyglot approach — each service can use different technologies or databases. |
| **Database** | Single shared database for all modules. | Each service manages its own database or schema. |
| **Communication** | Internal method or function calls (in-process). | Inter-service communication through APIs (HTTP/REST, gRPC) or messaging queues. |
---
| **Aspect** | **Monolithic Architecture** | **Microservices Architecture** |
|-------------|-----------------------------|--------------------------------|
| **Fault Isolation** | A single failure can bring down the entire application. | Failures are isolated — one service failing doesn’t crash the whole system. |
| **Maintenance** | Harder to maintain and modify as the codebase grows. | Easier to maintain; changes affect only individual services. |
| **Testing** | Easier to test as a single application. | More complex — requires integration and contract testing between services. |
| **Deployment Frequency** | Infrequent and heavy deployments. | Continuous and independent deployments possible. |

---
| **Aspect** | **Monolithic Architecture** | **Microservices Architecture** |
|-------------|-----------------------------|--------------------------------|
| **Performance** | Faster internal calls; no network overhead. | Slightly slower due to network communication between services. |
| **Team Organization** | Centralized team works on the same codebase. | Decentralized teams work independently on separate services. |
| **Security** | Easier to secure as it runs in one boundary. | More challenging — requires securing inter-service communications. |
| **Scaling Strategy** | Vertical scaling (increase server capacity). | Horizontal scaling (add more service instances). |
| **Example** | Traditional ERP, CRM, or early-stage web app. | Netflix, Amazon, Uber, Spotify. |

---
