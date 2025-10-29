---
marp: true

---
# Section 3
## Docker networking and storage
## 1. Types of Docker Networks: Bridge, Host, Overlay, etc.

**Definition**
Docker networking provides a system that allows containers to communicate with one another, with the host system, and with external networks such as the internet.
Each container runs in its own isolated network namespace, but Docker uses network drivers to connect containers securely and efficiently.

---
**Purpose of Docker Networking**
When you run multiple containers such as a web app, API, and database they need to exchange data. Networking defines how these containers find each other, connect, and send information while maintaining isolation and security.

### Main Types of Docker Networks
1. Bridge Network
2. Host Network
3. Overlay Network
4. Macvlan Network
5. None Network

---
### Bridge Network (Default and Most Common)

1. Default network type created by Docker on a single host. Containers in the same bridge network can communicate using container names.
2. Best for standalone or local multi-container setups.
3. Containers on the same host can communicate internally using DNS-based container names.

### Host Network
1. Shares the host machine’s network stack. The container uses the host’s IP directly, removing isolation.
2. High-performance apps, or tools that require direct host access (e.g., monitoring agents).
3. Container and host share same ports and IP.

---

### Overlay Network
1. Virtual network that spans across multiple Docker hosts (nodes). Used in Swarm or Kubernetes clusters.
2. Designed for multi-host communication — it connects containers running on different physical servers.Multi-host communication, distributed microservices.
3. Containers on different hosts can communicate securely.

### Macvlan Network

1. Assigns a MAC address to each container, making it appear as a physical device on the network.
2. Integrating containers with existing physical networks (legacy systems).

3. Containers behave like individual devices on LAN

---

### None Network
1. Disables all networking. Container is isolated and cannot communicate externally.
2. Highly secure or testing environments.
3. The container has no external or internal networking.
4. Used for fully isolated workloads or for testing applications that don’t require network connectivity.

## 2. Port Binding & Container Communication

**Concept of Port Binding**

By default, containers are isolated and can only communicate internally with other containers on the same network.To allow external access (e.g., from your browser or API client), you must bind a container port to a host port.
This process is known as port binding or port mapping.

---
## How Port Binding Works

When you run a container, you can specify how host ports map to container ports using the syntax:

***HostPort : ContainerPort***


For example, if your application runs on port 80 inside the container but you want to access it on port 8080 of your machine:

You bind host port **8080*** to container port 80.

Conceptually:

***User (Browser) → localhost:8080 → Docker Host → Container (port 80)***

---
### Container Communication

There are two levels of communication to consider:

**1. Internal Communication (Container-to-Container)**

Containers on the same user-defined bridge or overlay network can communicate directly by name.
Docker’s built-in DNS service allows resolving containers by their service name (e.g., db, api).

**2. External Communication (Container-to-Outside World)**

When you expose and bind ports, outside clients can communicate with the container via the host’s IP address and port.

---
## 3 Volume Types: Bind Mounts vs Named Volumes

**Definition**
Docker volumes manage how data is stored, shared, and persisted by containers.Since containers are ephemeral (temporary), data stored inside them disappears when they are deleted.To persist data beyond a container’s lifecycle, Docker provides storage mechanisms mainly volumes and bind mounts.

### Why Docker Storage Is Needed
Applications often need to save:
1. Database files
2. Configuration settings
3. Uploaded files
4. Logs or cache data

---
### Volume Types: 
**A. Bind Mounts**

- Connect a directory or file from the host filesystem directly into the container. 
- Whatever changes happen inside the container reflect immediately on the host (and vice versa).

**Characteristics:**

- Path is defined by the host (e.g., /home/user/data:/data).

- Useful for development because it allows live editing of code or data.

- Not portable — depends on host machine’s file structure.

- Offers direct access to the host filesystem, so it has security risks.
 
 ---
**B. Named Volumes**

- Managed entirely by Docker.

- Stored in Docker’s own area (/var/lib/docker/volumes/ on Linux).

- Docker automatically handles the path, creation, and lifecycle.

**Characteristics:**

- Decoupled from host file paths.

- Easily shared between containers.

- Persist even after the container is removed.

- Easier to back up, move, or manage across environments.

---
### Bind Mount Vs Name Volumes

| Feature | Bind Mounts  | Named Volumes |
|-----------|-----------|-----------|
| Location  | Anywhere on host filesystem | 	Managed by Docker under /var/lib/docker/volumes/ |
|Created by  | User (manual path specified) | Docker (automatic management) |
| Persistence | Depends on host path | Persistent even after container deletion |
| Use Case | Local development, debugging | Production, persistent databases |
| Security |Less secure (direct host access) | More secure (isolated) |


