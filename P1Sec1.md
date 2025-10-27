---
marp: true

---
# Phase One: Foundations of Containerization & Microservices
---
## Session 1:
## 1.  Understanding Containers vs Virtual Machines (VMs)
### Definition
**Virtual Machine (VM):**
A VM is a software-based emulation of a physical computer that includes its own Operating System (OS), libraries, and applications. It runs on top of a hypervisor such as VMware, VirtualBox,or KVM.  

**Container:**
A container is a lightweight, isolated environment that packages an application and all its dependencies but shares the host OS kernel. Containers start faster, use fewer resources, and are portable across environments.
 
 ---
 ### Difference Between VMs and Containers

| Feature | Virtual Machine | Container |
|-----------|-----------|-----------|
| OS | Includes full OS | 	Shares host OS |
| Startup Time | Minutes | Seconds |
| Size | Gigabytes | Megabytes |
| Isolation | Strong (hardware-level) | Moderate (process-level) |

**Example**:
**VM Example:** Running Ubuntu with Java application on VMware.
**Container Example:** Running a Node.js web app in Docker with only dependencies, not full OS.

---
#### Diagram
![bg center 55%](VmvsCon.png)

---
## 2. Benefits & Trade-Offs of Containers

### Definition
A **container** a lightweight, portable, and isolated environment that packages an application with everything it needs to run including code, runtime, and dependencies,but shares the host OS kernel instead of running its own operating system like a virtual machine (VM).
### Benefits of Containers
1. Containers run the same on any system that supports Docker/Kubernetes.
2. Share the host OS kernel → less RAM & CPU use.
4. Identical runtime environment everywhere.
5. Easily scale up or down using orchestration tools.
6. Great for CI/CD automation.
 ---

### Trade-Offs of Containers

1. Shared kernel may expose all containers to attacks.
2. Containers are temporary; local data is lost when stopped.
3. Multi-container networking can be tricky
4. Harder with many short-lived containers.
5. Docker and Kubernetes require new skills.

## 3. Container Registries
A **container registry** is a centralized repository used to store, manage, and distribute Docker images (container blueprints).

You can think of it like **GitHub** for **Docker images**.

---

## 4. Authentication into Private Container Registries

**Definition:**
Private container registries require **authentication** to ensure that only authorized users or systems can push or pull images. This protects proprietary code and prevents unauthorized access to sensitive application images.

**Purpose:**
Authentication helps:
1. Secure the registry from unauthorized users

2. Maintain accountability for who uploads or downloads images

3. Ensure compliance with enterprise security policies

---
### Common Authentication Methods:

1. **Username and Password / Tokens:**
Users authenticate with credentials or personal access tokens. These credentials are verified by the registry before allowing access.

2. **Cloud-based Authentication:**
Cloud registries like AWS ECR, GCP GCR, and Azure ACR integrate with their respective cloud identity systems (IAM, service accounts, etc.) to provide secure access without manual login.

3. **Kubernetes Secrets:**
In Kubernetes environments, credentials for private registries are stored as Secrets, which the cluster uses automatically when pulling images for pods.

---
4. **Single Sign-On (SSO) and Role-Based Access Control (RBAC):**
Enterprises often integrate registries with corporate identity systems (like Active Directory or SSO) and define specific roles and permissions for developers, DevOps teams, and automation tools.

### Key Considerations:
1. Use access tokens or IAM roles instead of plain passwords for better security.

2. Rotate credentials regularly to minimize risk.

3. Implement least-privilege access, giving users only the permissions they need.

4. Monitor access logs for suspicious activity.

5. Enable encryption in transit (HTTPS) for all registry communications.


