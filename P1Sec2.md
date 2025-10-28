---
marp: true

---
# Section 2
## Writing Dockerfile with Best Practices and Multi-Stage Builds
## 1. Dockerfile Structure: FROM, RUN, COPY, CMD, etc.

**Definition:**

A **Dockerfile** is a text file that contains a set of instructions that tell Docker how to build a container image.Each instruction creates a layer in the final image, forming the blueprint for your application environment.

---
## Common Dockerfile Instructions:

**1. FROM** : 
- Specifies the base image from which you build your image.
- Every Dockerfile must start with a FROM instruction unless it’s a scratch image.
- Best Practice: Use lightweight base images (Alpine, Slim) for smaller final images.

- **Example:**

        FROM ubuntu:22.04
        FROM node:18-alpine
        
---

**2. LABEL** 
- Adds metadata to the image (e.g., version, maintainer).
- Best Practice: Use LABELs for traceability and documentation
- Example:
    ```yaml
    LABEL maintainer="prerana@example.com" \
      version="1.0" \
      description="My sample app"
**3. WORKDIR**:
 - Sets the working directory inside the container.
 - Effect: All subsequent instructions (RUN, COPY, CMD) are executed in /app.
 - Example:

       WORKDIR /app
---
**4. COPY**: 
- Copies files from the host system into the container.
- Best Practice: Copy only necessary files to reduce image size and rebuild time.
- Example:

        COPY package.json package-lock.json /app/
        COPY src/ /app/src/
**5. RUN**: 
- Executes commands during image build (e.g., installing packages).
- Best Practice: Combine commands with && to reduce number of layers.
---
- Example:

        RUN apt-get update && apt-get install -y curl git
        RUN npm install

**6. CMD**: 
- Specifies the default command to run when a container starts.
- CMD can be overridden by docker run <image> <command>
- Example:

      CMD ["node", "index.js"]
---

**7. EXPOSE**: 
- Documents which ports the container listens on.
- EXPOSE does not publish the port — use docker run -p to map ports.
- Example:

        EXPOSE 3000

**8. ENV**: 
- Sets environment variables.
- Keep sensitive data out of Dockerfiles — use secrets or environment variables at runtime.
---
- Example:

        ENV NODE_ENV=production
        ENV PORT=3000

**9. ENTRYPOINT**: 
- Defines a command that always runs when the container starts.
- ENTRYPOINT cannot be overridden as easily, whereas CMD can.
- Example: 

        ENTRYPOINT ["python3", "app.py"]


---
## 2. Image Layering & Caching
### 2.1 Docker Image LayeringConcept of Layers

**What are Layers?**

- Every Docker image is built in layers.
- Each instruction in a Dockerfile (RUN, COPY, ENV, etc.) creates a read-only layer representing the changes made to the filesystem.
- These layers are stacked on top of each other when the container starts, plus a read-write layer on top for temporary changes
---

**Example Dockerfile:**

        FROM ubuntu:22.04
        COPY app/ /app
        RUN apt-get update && apt-get install -y curl
        ENV NODE_ENV=production

**Layer breakdown:**

- FROM ubuntu:22.04 → Base layer
- COPY app/ /app → Adds application files
- RUN apt-get ... → Installs packages
- ENV NODE_ENV=production → Sets environment variable

When Docker builds this image, it creates a separate layer for each instruction.

---
### 2.2 Layer Caching

Docker uses build caching if a layer hasn’t changed since the last build, it reuses the previous layer instead of rebuilding it.This significantly speeds up the image-building process.


### 3. Multi-Stage Builds

**Definition**

A multi-stage build allows you to use multiple FROM statements in a single Dockerfile to separate the build environment from the runtime environment.

This method is used to produce lean, production-ready images that contain only what is absolutely necessary for execution no compilers, temporary files, or build dependencies.

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

**Stage 1: Build Stage**

- Uses a heavy base image (with all build tools).
- Installs dependencies and compiles or builds the app

        FROM node:18 AS builder
        WORKDIR /app
        COPY package.json package-lock.json ./
        RUN npm install
        COPY . .
        RUN npm run build

---

**Stage 2: Runtime Stage**

- Uses a lightweight base image (like Alpine).
- Copies only the compiled output or binary from Stage 1.
- Excludes build tools and temporary files.

        FROM node:18-alpine
        WORKDIR /app
        COPY --from=builder /app/dist ./dist
        COPY --from=builder /app/package.json ./
        CMD ["node", "dist/index.js"]

3. **Result:**
- A clean, small, secure image optimized for running in production.

---
## Multistage Build Diagram:
 <div >
  <img src="multi.png" align="center"width="80%">
</div>


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

