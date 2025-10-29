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

---

**During setup:**
- Enable “Use WSL 2 instead of Hyper-V”
- Install the Ubuntu distribution (if not already)



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

**1. Login to Docker Hub**

     docker login

**2. Create a small web app folder**

     mkdir myapp && cd myapp
     echo '<h1>Hello from Preranas Private App!</h1>' > index.html

---

**3. Create a Dockerfile**

     cat <<EOF > Dockerfile
     FROM nginx:alpine
     COPY index.html /usr/share/nginx/html/index.html
     EOF

**4. Build your Docker image**

     docker build -t private-app:v1 .

**5. Tag it with your Docker Hub username**

     docker tag private-app:v1 preranabl/private-app:v1
    
*(replace **preranabl** with your **Docker Hub username** everywhere)*

---

**6. Push image to Docker Hub**

     docker push preranabl/private-app:v1
    
**7. Logout to simulate another user**

     docker logout

**8. Try pulling again (should fail)**
  
     docker pull preranabl/private-app:v1

**9. Login again and pull successfully**

     docker login
     docker pull preranabl/private-app:v1

---
**10. Run your private container**

     docker run -d -p 8081:80 --name private-app preranabl/private-app:v1

## Explanation

**1. Dockerfile:** Blueprint that defines how your image is built.
**2. docker build:** Builds an image from your Dockerfile.
**3. docker tag:** Gives your image a name following the Docker Hub format.
**4. docker push:** Uploads the image to your repository.
**5. docker pull:** Downloads images from the repository.
6. Logging out and trying again shows that private images require authentication.

---

## PART 3: Using Docker Compose (Public Repo Example)
**Goal:** Use Docker Compose to manage multiple containers at once (e.g., a web server + a database).

**1. Create a new folder**

      mkdir compose-lab && cd compose-lab

---

**2. touch docker-compose.yaml**

        version: "3"
        services:
        web:
            image: nginx:latest
            ports:
            - "8082:80"
        db:
            image: mongo:4.4
        container_name: my-mongo

**3. Run the multi-container app**

      docker compose up -d
---

**4. Check running services**

      docker compose ps


**5. Stop and clean up**

      docker compose down

### Explanation

**1. docker-compose.yml:** Describes multiple containers (called services) in one file.

**2. docker compose up -d**: Starts all services together.

**3. docker compose down:** Stops and removes everything cleanly.

---
## PART 4: Docker Compose with a Private Image

**Goal:** Use the image you pushed earlier (private image) with Docker Compose.

**1. Create a new folder**

      mkdir private-lab && cd private-lab

**2. touch docker-compose.yaml**

        version: "3"
        services:
        private-web:
            image: preranabl/private-app:v1
            ports:
            - "8083:80"

---

**3. Login to Docker Hub**

      docker login

**2. Run with compose**

     docker compose up -d
 ## Explanation 

1. **Docker Compose** automatically pulls the image from your **Docker Hub repo.**
2. Since the image is private, **authentication** (docker login) is required first.
3. The container runs and serves your custom webpage at: http://localhost:8083