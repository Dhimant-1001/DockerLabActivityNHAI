# Docker Image Optimization - Hands-On Activity

## Learning Objectives

By the end of this activity, you will be able to:
- Optimize Docker images using `.dockerignore` files
- Use build arguments (ARG) for flexible image builds
- Measure and compare Docker image sizes

---

## Introduction

In the previous lab, you learned how to build Docker images, run containers, persist data with volumes, and use bind mounts for development. However, the Dockerfile you created was simple and not optimized for production use.

In real-world scenarios, you need to:
- **Minimize image size** for faster deployments and reduced storage costs
- **Exclude unnecessary files** from the build context
- **Separate build and runtime dependencies** to reduce the attack surface
- **Create flexible builds** that can be configured without editing the Dockerfile

This activity will teach you how to optimize Docker images using **multi-stage builds** and **`.dockerignore`** files.

---

## Understanding the Problem

Your current Dockerfile looks like this:

```dockerfile
FROM node:24-alpine
WORKDIR /app
COPY . .
RUN npm install --omit=dev
CMD ["node", "src/index.js"]
EXPOSE 3000
```

**Issues with this approach:**
1. **Everything is copied** - Including test files, documentation, git history, etc.
2. **Build cache inefficiency** - Any file change invalidates `COPY . .` and subsequent layers
3. **No build optimization** - Dependencies are installed every time, even if unchanged
4. **Larger than necessary** - Unnecessary files increase image size

Multi-stage builds solve these problems!

---

## Task 1: Measure Baseline Image Size

Before optimizing, let's measure the current image size.

### Step 1.1: Navigate to the Project

```bash
cd getting-started-app
```

### Step 1.2: Build the Current Image

```bash
docker build -t getting-started:baseline .
```

### Step 1.3: Check the Image Size

```bash
docker images | grep getting-started
```

**📝 Note:** Record the image size. You should see something like this:

```
getting-started   baseline   abc123def456   1 minute ago   200MB
```

**Action Required:** Take a screenshot of this output. You'll compare it later!

### Step 1.4: Test the Baseline Image

Run the container to ensure it works:

```bash
docker run -dp 127.0.0.1:3000:3000 getting-started:baseline
```

Open http://localhost:3000 in your browser and verify the app works.

Stop and remove the container:

```bash
docker ps
docker rm -f <container_id>
```

---

## Task 2: Create a `.dockerignore` File

Just like `.gitignore` excludes files from Git, `.dockerignore` excludes files from the Docker build context.

### Step 2.1: Create `.dockerignore`

In the `getting-started-app` directory, create a file named `.dockerignore`:

```bash
touch .dockerignore
```

Or use your editor:
```bash
code .dockerignore
```

### Step 2.2: Add Exclusions

Add the following content to `.dockerignore`:

```
# Dependencies (will be installed in container)
node_modules/

# Test files (not needed in production)
spec/

# Git repository
.git/
.gitignore

# Documentation
*.md
README.md

# Development files
.vscode/
.idea/

# OS files
.DS_Store
Thumbs.db

# Logs
*.log
npm-debug.log*
```

**Explanation:**
- `node_modules/` - Will be installed fresh in the container
- `spec/` - Test files not needed in production
- `.git/` - Git history not needed in the image
- `*.md` - Documentation files not needed at runtime

### Step 2.3: Build with `.dockerignore`

```bash
docker build -t getting-started:with-dockerignore .
```

### Step 2.4: Compare Sizes

```bash
docker images | grep getting-started
```

**📝 Note:** The build should be faster, and the image might be slightly smaller (depending on how many files were excluded).

**Action Required:** Take a screenshot showing both images.

---

## Task 3: Implement Multi-Stage Build

Multi-stage builds allow you to use multiple `FROM` statements in your Dockerfile. Each `FROM` instruction starts a new stage. You can selectively copy artifacts from one stage to another.

### Step 3.1: Create Optimized Dockerfile

Open the existing `Dockerfile` and **replace its entire content** with this multi-stage version:

```dockerfile
# syntax=docker/dockerfile:1

# Stage 1: Build stage
FROM node:24-alpine AS builder
WORKDIR /app

# Copy package files first (for better caching)
COPY package*.json ./

# Install ALL dependencies (including dev dependencies for building)
RUN npm install

# Copy application source code
COPY . .

# Stage 2: Production stage
FROM node:24-alpine AS production
WORKDIR /app

# Copy package files
COPY package*.json ./

# Install ONLY production dependencies
RUN npm install --omit=dev

# Copy application source from builder (or current directory)
COPY --from=builder /app/src ./src

# Set the command
CMD ["node", "src/index.js"]

# Expose port
EXPOSE 3000
```

**Key Differences:**
- **Two stages**: `builder` and `production`
- **Layer caching**: Package files copied before source code
- **Selective copying**: Only necessary files copied to final stage
- **Optimized dependencies**: Production dependencies only in final image

### Step 3.2: Build the Multi-Stage Image

```bash
docker build -t getting-started:optimized .
```

**Notice:** The build shows multiple stages:

```
[builder 1/5] FROM node:24-alpine
[production 1/5] FROM node:24-alpine
...
```

### Step 3.3: Compare All Image Sizes

```bash
docker images | grep getting-started
```

You should now have three images:
- `getting-started:baseline`
- `getting-started:with-dockerignore`
- `getting-started:optimized`

**📝 Expected Results:**
- `baseline`: ~200MB
- `with-dockerignore`: ~195MB (slightly smaller)
- `optimized`: ~140-160MB (significantly smaller!)

**Action Required:** Take a screenshot of the comparison!

### Step 3.4: Test the Optimized Image

```bash
docker run -dp 127.0.0.1:3000:3000 --mount type=volume,src=todo-db,target=/etc/todos getting-started:optimized
```

Open http://localhost:3000 and verify:
- ✅ App loads correctly
- ✅ You can add todo items
- ✅ Items persist after container restart

**Action Required:** Take a screenshot of the working app!

Stop the container when done:
```bash
docker ps
docker rm -f <container_id>
```

---

## Task 4: Use Build Arguments for Flexibility

Build arguments (ARG) allow you to pass values during build time, making your Dockerfile more flexible.

### Step 4.1: Add ARG to Dockerfile

Modify your Dockerfile to accept a Node version argument. Update the **first line after the syntax declaration**:

```dockerfile
# syntax=docker/dockerfile:1

# Build argument for Node version
ARG NODE_VERSION=24

# Stage 1: Build stage
FROM node:${NODE_VERSION}-alpine AS builder
WORKDIR /app

# Copy package files first (for better caching)
COPY package*.json ./

# Install ALL dependencies (including dev dependencies for building)
RUN npm install

# Copy application source code
COPY . .

# Stage 2: Production stage
FROM node:${NODE_VERSION}-alpine AS production
WORKDIR /app

# Copy package files
COPY package*.json ./

# Install ONLY production dependencies
RUN npm install --omit=dev

# Copy application source from builder (or current directory)
COPY --from=builder /app/src ./src

# Set the command
CMD ["node", "src/index.js"]

# Expose port
EXPOSE 3000
```

### Step 4.2: Build with Default ARG

```bash
docker build -t getting-started:arg-default .
```

This uses Node 24 (the default value).

### Step 4.3: Build with Custom ARG

```bash
docker build --build-arg NODE_VERSION=22 -t getting-started:node22 .
```

This builds with Node 22!

### Step 4.4: Verify Different Versions

```bash
docker images | grep getting-started
```

**📝 Note:** You should see both versions listed.

**Action Required:** Take a screenshot showing multiple Node versions!

---

## Task 5: Analyze Image Layers

Understanding image layers helps you optimize builds further.

### Step 5.1: Inspect Image History

```bash
docker history getting-started:optimized
```

This shows all layers in the image, their size, and the commands that created them.

### Step 5.2: Inspect Baseline History

```bash
docker history getting-started:baseline
```

**📝 Compare:** Notice how the optimized version has more layers but is smaller overall?

**Action Required:** Take screenshots of both outputs!

---

## Task 6: Performance Test (Optional Challenge)

### Step 6.1: Measure Build Time

Clear Docker build cache:

```bash
docker builder prune -f
```

Build baseline and measure time:

```bash
time docker build -t getting-started:baseline-test .
```

Build optimized and measure time:

```bash
time docker build -t getting-started:optimized-test .
```

**📝 Note:** First builds might be similar, but rebuilds (after code changes) will be much faster with the optimized version due to better layer caching!

---

## Submission Requirements

### What to Submit

Submit a **single PDF file** containing the following sections:

#### 1. Cover Page
- Your name
- Date of completion
- Activity title: "Docker Image Optimization - Hands-On Activity"

#### 2. Commands Section
Copy and paste all commands you executed, organized by task:

```
Task 1: Baseline Measurement
$ docker build -t getting-started:baseline .
$ docker images | grep getting-started
...
```

#### 3. Screenshots Section

Include the following screenshots (properly labeled):

- [ ] **Screenshot 1:** Baseline image size (`docker images`)
- [ ] **Screenshot 2:** Image sizes after adding `.dockerignore`
- [ ] **Screenshot 3:** All three images compared (baseline, with-dockerignore, optimized)
- [ ] **Screenshot 4:** Working Todo app in browser (http://localhost:3000)
- [ ] **Screenshot 5:** Multi-stage build in progress showing both stages
- [ ] **Screenshot 6:** Images with different Node versions (Task 4)
- [ ] **Screenshot 7:** `docker history` output for optimized image
- [ ] **Screenshot 8:** `docker history` output for baseline image

#### 4. Code Files Section

Include the complete content of:

**`.dockerignore` file:**
```
(paste your complete .dockerignore content here)
```

**Final optimized `Dockerfile`:**
```dockerfile
(paste your complete multi-stage Dockerfile here)
```

#### 5. Comparison Table

Create a table comparing your results:

| Image Tag | Size | Node Version | Stages |
|-----------|------|--------------|--------|
| baseline | ___MB | 24 | 1 |
| with-dockerignore | ___MB | 24 | 1 |
| optimized | ___MB | 24 | 2 |
| node22 | ___MB | 22 | 2 |



### File Naming Convention

Submit your PDF with this naming format:
```
Docker_ImageOptimization_<YourName>_<Date>.pdf
```

Example: `Docker_ImageOptimization_DhimantBhuva.pdf`

---

**Good luck with your activity! 🐳**
