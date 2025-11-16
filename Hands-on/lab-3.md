---
marp: true

---
# LAB 3
# Connect Containers Using Docker Networks and mount a volume and inspect data persistence

---
## PART 1 — Connect 2 containers using a user-defined Bridge network

**Step 1 — Create a network**

     docker network create mynet

Check the networks:

     docker network ls

**Step 2 — Run 2 containers in that network**

     docker run -dit --name c1 --network mynet ubuntu
     docker run -dit --name c2 --network mynet ubuntu

---
**Explanation:**

| Flag | Meaning |
|-----------|-----------|
| -d | run in background| 
| -i | interactive | 
| -t | allocate terminal |
| --name | gives easy container name | 
| --network mynet | attaches to network we created | 

**Step 3 — ping containers using container name**
Go inside container c1 shell:

      docker exec -it c1 bash

---
Inside container run:

     ping c2 -c 3

If it replies, you have container communication.
This proves containers on same user-defined bridge can talk using DNS

## PART 2 — Mount a Named Volume & Inspect Persistence

**Step 1 — Create a volume**

     docker volume create datavol

**Step 2 — Run container with mounted volume**

     docker run -dit --name voltest -v datavol:/data alpine

---
**Step 3 — Write some data inside container**
Enter container:

     docker exec -it voltest sh

Inside container:

     cd /data
     echo "Hello from volume" > file1.txt
     ls
     cat file1.txt
Exit container:

     exit

---

**Step 4 — Remove container**

     docker rm -f voltest
**Step 5 — Run a new container with same volume**

     docker run -it --rm -v datavol:/data alpine sh

**Check files:**

     ls /data
     cat /data/file1.txt
