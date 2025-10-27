---
marp: true

---
# Section 5
## 12-factor application
### 1. Overview of all 12 factors
**Definition**: 
The 12-Factor App methodology is a set of best practices for building modern, cloud-native applications. It was introduced by Heroku to enable scalable, maintainable, and deployable applications. Each factor addresses a specific aspect of application development and deployment.

---

**1. Codebase**

Every application should have a single codebase in a version control system. Multiple versions should use branches, not separate repositories. Each deployment environment, including local development, counts as a deploy. In microservices, each service should have its own codebase for easier CI/CD. Shared code should be managed as libraries via package repositories.

**2. Dependencies** 

All dependencies must be declared and managed externally, not included in the codebase. For Java, tools like Maven or Gradle manage dependencies. System-level dependencies should also be installed via configuration management tools or containerization (e.g., Docker). In microservices, package managers handle dependencies independently for each service.

---

**3. Config**

Configuration values, such as database credentials, API keys, and ports, must not be hardcoded. They should be stored in environment variables. This ensures the same code runs across all environments. Microservices can manage configs externally via tools like Spring Cloud Config.

**4. Backing Services**

External services like databases, message queues, or caches are treated as attached resources. They can be swapped without code changes. Microservices use interface-based or plugin-based implementations to switch resources dynamically.

---

**5. Build, Release, Run**

Build, release, and run stages must be separate. Build packages the code, release combines it with environment configs, and run executes it. CI/CD pipelines and Docker make this separation efficient in microservices.

**6. Processes**

Applications should run as stateless processes. Any data or state must be stored in external backing services. This avoids sticky sessions and allows horizontal scaling. Microservices use stateless REST APIs and caches like Redis or Memcached for state.

**7. Port Binding**

Applications should be self-contained and expose HTTP services through a port. They should not rely on external web servers. Microservices, like Spring Boot apps, often use embedded servers like Tomcat or Jetty.

---

**8. Concurrency**

Apps should scale horizontally by running multiple processes or instances, rather than relying on vertical scaling. Threads can optimize concurrency within processes. Containerized microservices can scale automatically according to demand.

**9. Disposability**

Processes should start and stop quickly without affecting application state. Graceful shutdowns ensure minimal disruption. Containers in microservices make disposability easier by handling state in backing services.

**10. Dev/Prod Parity**

Development, staging, and production should be as similar as possible. This reduces environment-specific bugs. Containerization ensures parity by running the same image in all environments.

---
**11. Logs**

Applications should write logs to standard output. The execution environment captures, stores, and processes logs. Microservices rely on centralized logging and observability tools like ELK, Splunk, or New Relic.

**12. Admin Processes**

One-off tasks, like migrations or scripts, should be part of the codebase and run using the same environment as the app. Automation tools and containerized tasks make admin processes easier in microservices.

---
### 2. Importance in cloud-native systems

The 12-Factor principles enable cloud-native systems to be:

**1. Portable:** The explicit declaration of dependencies and environment-based config allows apps to run on any infrastructure or cloud provider without modification.
**2. Scalable:** Stateless process models and concurrency principles enable horizontal scaling with automated orchestration platforms like Kubernetes.
**3. Resilient:** Disposability ensures apps can start, stop, and recover quickly, improving system robustness and uptime.
**4. Maintainable:** Strict separation of concerns (build, config, run) aids continuous integration/delivery (CI/CD) pipelines, speeding up development and deployment.
**5. Consistent:** Dev/Prod parity reduces the “works on my machine” problem by aligning all environments closely.
