# Docker Image Optimization - Assessment Quiz

**Name:** ___________________________  
**Date:** ___________________________

---


### Question 1
What does the `-d` flag do in the command `docker run -d getting-started`?

A) Deletes the container after it stops  
B) Runs the container in detached mode (background)  
C) Downloads the image before running  
D) Displays detailed logs

---

### Question 2
What is the primary purpose of a `.dockerignore` file?

A) To ignore Docker commands during build  
B) To exclude files from being copied into the Docker image  
C) To hide Docker containers from `docker ps`  
D) To prevent Docker from starting automatically

---

### Question 3
Which command displays all Docker images on your system?

A) `docker ps`  
B) `docker list`  
C) `docker images`  
D) `docker show images`

---

### Question 4
What does `EXPOSE 3000` do in a Dockerfile?

A) Automatically publishes port 3000 to the host  
B) Documents that the container listens on port 3000  
C) Opens port 3000 in the firewall  
D) Forces the application to run on port 3000

---

### Question 5
In the command `docker run -p 127.0.0.1:3000:3000 my-app`, what does the `-p` flag do?

A) Pauses the container after starting  
B) Sets the priority of the container  
C) Maps port 3000 from the container to port 3000 on the host  
D) Makes the container persistent

---

### Question 6
Which command correctly builds a Docker image with a custom build argument for Node version?

A) `docker build -t my-app --arg NODE_VERSION=22 .`  
B) `docker build -t my-app --build-arg NODE_VERSION=22 .`  
C) `docker build -t my-app -e NODE_VERSION=22 .`  
D) `docker build -t my-app --env NODE_VERSION=22 .`

---


