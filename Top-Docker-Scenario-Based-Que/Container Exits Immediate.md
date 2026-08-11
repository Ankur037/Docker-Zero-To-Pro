🚨 **Docker Production Troubleshooting | Scenario #01**

### ❓ Interview Scenario

You deployed a Docker container using:

```bash
docker run -d --name my-app my-app:v1
```

But when you check the container:

```bash
docker ps
```

👉 **The container is not running.**

You check:

```bash
docker ps -a
```

And find:

```text
CONTAINER ID   IMAGE       STATUS
abc123         my-app:v1   Exited (1) 10 seconds ago
```

### 🤔 What would you check first?

Don't immediately restart the container.

Start with **logs**.

---

## 🔎 Step 1: Check Container Logs

```bash
docker logs my-app
```

Logs often reveal the actual application error.

For example:

```text
Error: Cannot connect to database
```

or:

```text
Permission denied
```

or:

```text
python: can't open file 'app.py'
```

---

## 🔍 Step 2: Inspect the Container

```bash
docker inspect my-app
```

Check:

* Exit code
* Entrypoint
* CMD
* Environment variables
* Mounts
* Network configuration

A particularly important field is:

```text
State.ExitCode
```

---

## 🧪 Step 3: Check the Exit Code

Common examples:

```text
Exit Code 0
```

Usually means the main process completed successfully.

```text
Exit Code 1
```

Generic application error.

```text
Exit Code 126
```

Command exists but cannot be executed.

```text
Exit Code 127
```

Command not found.

```text
Exit Code 137
```

Often indicates the process was killed, commonly due to an OOM condition.

---

## 🧠 The Most Important Docker Concept

A Docker container is designed to run as long as its **main process (PID 1)** is running.

For example:

```bash
docker run ubuntu
```

The container may immediately exit because there is no long-running foreground process.

But:

```bash
docker run -it ubuntu /bin/bash
```

keeps the container running while the interactive shell is active.

---

## ⚠️ Common Root Causes

A container can exit immediately because of:

🔹 Application startup failure

🔹 Incorrect `CMD` or `ENTRYPOINT`

🔹 Missing configuration/environment variables

🔹 Missing files

🔹 Permission problems

🔹 Dependency/database connection failure

🔹 Port/configuration issues

🔹 Application process completing immediately

🔹 OOM / resource exhaustion

---

## 🛠️ Troubleshooting Flow

```bash
docker ps -a
        ↓
docker logs <container>
        ↓
docker inspect <container>
        ↓
Check CMD / ENTRYPOINT
        ↓
Check Environment
        ↓
Check Mounts
        ↓
Check Resources
        ↓
Fix Root Cause
        ↓
Recreate Container
```

---

## 💼 Production Best Practice

Don't treat:

```bash
docker restart <container>
```

as the solution.

A restart may temporarily bring the container back, but if the underlying application or configuration is broken, it will simply fail again.

**Find the root cause first.**

For production workloads, also configure appropriate health checks, resource limits, centralized logging, and restart policies according to the application's requirements.

---

## 🎯 Interview Answer

**Q: A Docker container starts and immediately exits. How would you troubleshoot it?**

**Answer:**

> First, I would check `docker ps -a` to confirm the exit status and exit code. Then I would check `docker logs` for application-level errors and use `docker inspect` to verify the container's CMD, ENTRYPOINT, environment variables, mounts, and state. I would then identify whether the issue is related to the application process, configuration, permissions, dependencies, or resource constraints before recreating or restarting the container.

This demonstrates **structured production troubleshooting** instead of simply restarting the workload.

---

### 🎯 Key Takeaway

**A stopped container is a symptom—not the root cause.**

The first three commands I would remember are:

```bash
docker ps -a
docker logs <container>
docker inspect <container>
```

**Observe → Investigate → Identify Root Cause → Fix → Validate**

