---
marp: true

---
# Section 2
## Writing Dockerfile with Best Practices and Multi-Stage Builds
### 1. Dockerfile Structure: FROM, RUN, COPY, CMD, etc.

**Definition:**

A **Dockerfile** is a text file that contains a set of instructions that tell Docker how to build a container image.Each instruction creates a layer in the final image, forming the blueprint for your application environment.

---
### Common Dockerfile Instructions:

1. **FROM** : Specifies the base image from which you build your image
2. **LABEL** : Adds metadata to the image (e.g., version, maintainer).
3. **WORKDIR**: Sets the working directory inside the container.
4. **COPY**: Copies files from the host system into the container.
5. **RUN**: Executes commands during image build (e.g., installing packages).
6. **CMD**: Specifies the default command to run when a container starts.
7. **EXPOSE**: Documents which ports the container listens on.
8. **ENV**: Sets environment variables.
9. **ENTRYPOINT**: Defines a command that always runs when the container starts.

---
### 2. Image Layering & Caching

#### Concept of Layers
Every Docker image is built in layers. Each line in a Dockerfile (e.g., RUN, COPY, ENV) creates a new read-only layer that represents changes to the file system.

When a container is started, Docker stacks these layers together plus a read-write layer on top, allowing the container to modify files temporarily.

This layered approach makes Docker images:

**Modular** — only changed parts are rebuilt.

**Cache-friendly** — speeds up builds using existing layers.

---
#### Layer Caching

Docker uses build caching if a layer hasn’t changed since the last build, it reuses the previous layer instead of rebuilding it.

This significantly speeds up the image-building process.

### 3. Multi-Stage Builds

**Definition**

A multi-stage build allows you to use multiple FROM statements in a single Dockerfile to separate the build environment from the runtime environment.

This method is used to produce lean, production-ready images that contain only what is absolutely necessary for execution — no compilers, temporary files, or build dependencies.

---

### Why Multi-Stage Builds Are Needed
When building applications (like in Node.js, Go, or Java), the build process requires:

**1. Compilers**

**2. Build tools**

Dependencies only used during compilation

However, those tools are not needed in the final image that runs the app.
Keeping them would make the image unnecessarily large and expose potential security vulnerabilities.

---
### How Multi-Stage Builds Work Conceptually

1. **Stage 1: Build Stage**

- Uses a heavy base image (with all build tools).

- Installs dependencies and compiles or builds the app

2. **Stage 2: Runtime Stage**

- Uses a lightweight base image (like Alpine).

- Copies only the compiled output or binary from Stage 1.

- Excludes build tools and temporary files.

3. **Result:**
- A clean, small, secure image optimized for running in production.

---
### 4. Statelessness, Disposability & Logging Best Practices

####  A. Statelessness

A **stateless container** means it does not depend on internal stored data or session state.
All persistent data should live outside the container in databases, storage volumes, or external caches.

This is vital for scalability because stateless containers can be stopped, replaced, or scaled horizontally without losing user data.

--- 
#### B. Disposability
Containers should be **disposable**  they can start and stop quickly, leaving no permanent footprint.This aligns with microservices architecture and cloud-native principles.

**Best Practices:**

1. Handle shutdown signals gracefully to prevent data corruption.

2. Keep startup time short (avoid long initialization processes).

3. Design for fast replacement (e.g., stateless apps, health checks).

Disposability enables rolling updates, zero-downtime deployments, and auto-scaling essential in Kubernetes.

--- 
#### C. Logging Best Practices

Logging in containers should follow these principles:

Write logs to standard output (stdout) and standard error (stderr).

Avoid writing logs to files inside the container because they are temporary and can be lost when the container stops.

Use centralized log management systems to collect and analyze logs, such as:

1. ELK Stack (Elasticsearch, Logstash, Kibana)

2. Fluentd

3. CloudWatch

4. Prometheus

