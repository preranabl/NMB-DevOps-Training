---
marp: true

---
# Docker Lab: Pulling and Running Containers (Public & Private Repos w/wo Docker Compose)

---
## PART 0: Installation & Setup
**1. Install Docker (includes Docker Compose)**

- Docker Desktop (for Windows/macOS) and Docker Engine (for Linux) both include Docker Compose.

### For macOS
**Steps:**

- Go to : https://www.docker.com/products/docker-desktop
- Download Docker Desktop for Mac
- Choose the version for your chip (Intel or Apple Silicon).
- Open the .dmg file and drag Docker to Applications.
- Open Docker Desktop and wait until it shows “Docker is running”.
---
- Verify the installation in your Terminal:

        docker --version
        docker compose version

### For Windows (with WSL 2)
**Steps:**

- Download Docker Desktop for Windows: https://www.docker.com/products/docker-desktop
- Run the installer.

**During setup:**
- Enable “Use WSL 2 instead of Hyper-V”
- Install the Ubuntu distribution (if not already)

---

- Restart your system after installation.
- Open Docker Desktop → Settings → ensure “Use WSL 2 based engine” is selected.
- Verify installation in PowerShell or WSL Terminal:

        docker --version
        docker compose version

---
## PART 1: Pull and Run Container from a Public Repository (Without Docker Compose)


**Goal :** Learn how to pull and run a container image that is publicly available on Docker Hub, no login or authentication needed.

**1. Check Docker installation:**

      docker --version

**2. Pull the official Nginx image from Docker Hub:**

      pull nginx:latest

---

**3. Run the container and expose port 8080 on your system:**

      docker run -d -p 8080:80 --name my-nginx nginx

**4. Verify running containers:**

      docker ps
**5. Open your browser and visit:**

      http://localhost:8080
**6. Stop and remove the container**

      docker stop my-nginx
      docker rm my-nginx

---

### Explanation:


**1. docker pull nginx:latest**: Downloads the Nginx image from the public Docker Hub repository.

**2. docker run -d -p 8080:80:**

- -d → detached mode (runs in the background)
- -p 8080:80 → maps your computer’s port 8080 to the container’s port 80 (web traffic)

**3. docker ps:** Lists all running containers
**4. docker stop & docker rm:** Used to stop and remove containers


---

## PART 2: Pull and Run Container from a Private Repository (Without Docker Compose)

**Goal**: Learn how authentication works in Docker, create a custom image, push it to your private Docker Hub repository, then pull and run it again.

