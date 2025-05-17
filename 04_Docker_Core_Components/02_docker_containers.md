# Docker Container:
- Runtime Instance of an Image
- It's a process on the host machine, isolated using kernel features (namespaces, cgroups).
- Containers are ephemeral(Temporary) — unless you persist volumes.
- Immutable Infrastructure

### 1. Runtime Instance of an Image
A Docker container is a running instance of a Docker image. So, when you run an image, it becomes a container.
- You build an image once.
- You run containers from it multiple times.

Example:
```bash
docker run -d my-python-api
```
This creates a running container from your `my-python-api` image.

### 2. It’s Just a Process on the Host OS
- A container is not a virtual machine.
- It's a lightweight process running directly on the host Linux kernel.
- But it feels isolated because of:
  1. **Namespaces**: isolate the container’s process, network, file system
  2. **Cgroups**: control how much CPU/RAM the container can use

So, even though it's just a process, Docker makes it feel like a mini-machine.


### 3. Ephemeral (Temporary) by Default
If you don’t configure storage, data inside the container is lost once it stops or is deleted.

Example:
```bash
docker run -it ubuntu
touch test.txt  # you create a file
exit            # stop container

# When you restart it, test.txt will be gone
```
To keep data safe, we use **volumes** or **bind mounts**.

```bash
docker run -d -p 5000:5000 myapp:latest
```

This starts a new container from the image myapp:latest.

**Use Case:** In a microservices setup, you run 5–10 containers:
- Backend service
- Database
- Redis
- NGINX reverse proxy

Each is built as a separate image and deployed as a container.

### 4. Immutable Infrastructure
Immutable infrastructure means:
- You do not modify a running container (or VM or server).
- Instead, you destroy the old one and replace it with a new version.

**In terms of Docker:** Let’s say you have a container running your app:
```bash
docker run -d --name my-api v1.0
```
Now you want to update the app (e.g., bug fix or feature added).

In traditional systems, you might:
- SSH into the server
- Change some files
- Restart the service

❌ This is mutable infrastructure — prone to bugs, drift, and inconsistent environments.

**With Docker (Immutable Approach):**
1. You change your code
2. You rebuild your Docker image:
```bash
docker build -t my-api:v1.1 .
```
3. You stop and remove the old container:
```bash
docker stop my-api && docker rm my-api
```
4. You run a new one:
```bash
docker run -d --name my-api my-api:v1.1
```
No in-place patching. You treat containers as disposable.

#### DevOps Use Case:
- Run microservices like `user-service`, `auth-service`, `payment-service` in isolated containers.
- You don’t pollute host machine with Python, Java, Node, etc.
- You can restart, scale, or destroy individual services.

#### Why Containers Matter:
- Ephemeral: Easily recreate the same environment from image.
- Lightweight: Start in milliseconds.
- Scalable: Use Docker Compose or Kubernetes to scale.
- Isolation: Keeps each service cleanly separated.
