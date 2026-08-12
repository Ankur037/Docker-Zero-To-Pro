## 🚨 Docker Production Troubleshooting | Scenario #02

### ❓ Interview Scenario

You have started an NGINX container:

```bash
docker run -d --name web nginx
```

The container is running:

```bash
docker ps
```

But someone checks the Dockerfile and sees:

```dockerfile
EXPOSE 80
```

They ask:

> **“Does `EXPOSE 80` make the application accessible on port 80 from outside the container?”**

### ❌ Answer: No.

`EXPOSE` **does not publish or open a port** on the Docker host.

It mainly acts as **documentation/metadata** indicating which port the application inside the container expects to use.

---

### 🔎 What happens with `EXPOSE`?

For example:

```dockerfile
FROM nginx
EXPOSE 80
```

The application listens on:

```text
Container → Port 80
```

But this does **not automatically mean**:

```text
Internet → Host Port 80 → Container Port 80
```

For external access, you normally need **port publishing**.

---

## 🌐 Publish the Port

Run:

```bash
docker run -d \
  --name web \
  -p 80:80 \
  nginx
```

Here:

```text
-p HOST_PORT:CONTAINER_PORT
```

So:

```text
-p 80:80
```

means:

```text
Host Port 80
      │
      ▼
Container Port 80
      │
      ▼
     NGINX
```

---

## 🔍 Verify Port Mapping

```bash
docker ps
```

You may see:

```text
PORTS
0.0.0.0:80->80/tcp
```

You can also inspect it:

```bash
docker port web
```

---

## 🧪 Real Production Troubleshooting

Suppose:

```bash
curl localhost:80
```

works **inside the Docker host**, but users cannot access the application remotely.

Don't immediately blame Docker.

Check the complete path:

```text
Internet
   │
   ▼
Cloud Security Group / Firewall
   │
   ▼
Host Port 80
   │
   ▼
Docker Port Mapping
   │
   ▼
Container Port 80
   │
   ▼
NGINX
```

For an EC2 environment, verify that the Security Group allows inbound TCP/80.

---

## ⚠️ Common Mistake

Many beginners assume:

```dockerfile
EXPOSE 8080
```

means Docker automatically opens port `8080`.

It doesn't.

You still need:

```bash
docker run -p 8080:8080 my-app
```

---

## 🎯 Interview Answer

**Q: What is the difference between `EXPOSE` and `-p` in Docker?**

**Answer:**

> `EXPOSE` documents the port that the application inside the image is expected to listen on. It does not publish the port to the host. The `-p` option creates an actual host-to-container port mapping and makes the container service reachable through that published host port, subject to host/cloud firewall rules.

### 💼 Production Tip

Don't publish ports unnecessarily.

If a database is only required by an application container, keep it on a private Docker network instead of exposing its port to the public internet.

**Expose only the required entry points.**

### 🎯 Key Takeaway

```text
EXPOSE
   ↓
Documentation / Metadata

-p
   ↓
Actual Port Publishing

Firewall / Security Group
   ↓
Controls External Reachability
```

**Remember:** `EXPOSE` ≠ `Publish`.
