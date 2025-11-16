---
marp: true
_class: invert
---
# Lab 2 
# Write a Dockerfile for a simple Node.js or Python app
---
## Dockerfile
- A Dockerfile is a text file that tells Docker how to build an image.
- It contains step-by-step instructions like:
    1. Which base image to use
    2. what files to copy into the container
    3. what command to run when the container starts
    
**Workflow understanding:**

1. you write Dockerfile
2. Docker reads the file and builds an image
3. you can run that image as a container

---
## PART 1: Write a Dockerfile for a simple Node.js app
**1. Create folder and app**

     mkdir node-lab
     cd node-lab

**2. Create package.json**

    {
    "name": "node-lab",
    "version": "1.0.0",
    "scripts": {
        "start": "node index.js"
    }
    }

---
**3. Create index.js**

     console.log("Hello from a simple Dockerized Node.js App!")

**4. Create Dockerfile (SINGLE STAGE)**

- Create file named: Dockerfile
---
```

#Use official Node.js runtime as the base image
FROM node:18-alpine

#Create app directory
WORKDIR /usr/src/app

#Copy package.json first (takes advantage of Docker cache)
COPY package.json ./

#Install dependencies
RUN npm install --production

#Copy app source code
COPY . .

#Environment variable (optional)
ENV PORT=3000

#Expose port (documentation only)
EXPOSE 3000

#Command to run the app
CMD ["npm", "start"]
```

---
Line-by-line explanation 

- **FROM node:18-alpine —** base image with Node 18 on Alpine Linux (small). The base image provides node runtime.
- **WORKDIR /usr/src/app —** sets the working directory inside container; subsequent commands run there.
- **COPY package.json ./ —** copy only package.json first so npm install can reuse cached layers if code changes later.
- **RUN npm install --production** — install only dependencies needed to run (not devDeps).
- **COPY . .** — copy application files into the container.
- **ENV PORT=3000** — sets a default environment variable inside the container.

---
- **EXPOSE 3000 —** documents that container listens on port 3000 (doesn't publish by itself).

- **CMD ["npm", "start"]** — process to run when the container starts. Use JSON form to avoid shell parsing surprises.

**5. Multi-stage Dockerfile**
- Create Dockerfile.multistage:

        #Stage 1: build

        FROM node:18-alpine AS build
        WORKDIR /usr/src/app
        COPY package.json ./
        RUN npm install
        COPY . .

        #If you had a build step (e.g. frontend bundling), you'd run it here
        #RUN npm run build

---

        #Stage 2: production runtime

        FROM node:18-alpine AS runtime
        WORKDIR /usr/src/app

        #Create a non-root user for security (best practice)

        RUN addgroup -S appgroup && adduser -S appuser -G appgroup

        #Copy only necessary artifacts from build stage

        COPY --from=build /usr/src/app /usr/src/app

        #Install only production dependencies (optional if copied from build)

        RUN npm prune --production

        USER appuser

        ENV PORT=3000
        EXPOSE 3000
        CMD ["npm", "start"]

---
**6. 4. Build, run, test locally**
From node-app folder:

- Build single-stage:

      docker build -t node-hello:1.0 .

- Or build multi-stage (use the multistage Dockerfile):

      docker build -f Dockerfile.multistage -t node-hello:1.0 .
    
- Run:

      docker run --rm -p 3000:3000 --name node-hello node-hello:1.0

---
- *Visit http://localhost:3000 to see output.*
- *--rm removes container after it stops (useful for temporary runs).*

**7. Check images and containers:**

     docker images
     docker ps -a

---
## Part 2: Tag, Login, and Push to Registry 

**1. Create a remote repo (Docker Hub)**

Go to Docker Hub (hub.docker.com) and create a new repository named e.g. preranabl/node-hello (replace preranabl with your Docker Hub username).

**2. Login locally**

     docker login

---

**3. Tag your image in the correct format**

- Docker Hub image name format: <username>/<repo>:<tag>

**From Node example:**

     docker tag node-hello:1.0 preranabl/node-hello:1.0

**4. Push the image**

     docker push preranabl/node-hello:1.0

**5. Pull from another machine**

     docker logout
     docker pull preranabl/node-hello:1.0   # will fail if private and not logged in
     docker login
     docker pull preranabl/node-hello:1.0   # will succeed after login