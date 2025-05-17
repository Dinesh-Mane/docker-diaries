# Docker Core Components

## 1. Docker Image:
A Docker image is a read-only template that includes everything needed to run an application:  
✅ code + ✅ dependencies + ✅ base OS (like Ubuntu/Alpine)  

You build this image using a Dockerfile, and it becomes a blueprint to create running containers.
> Docker reads the Dockerfile and builds the image.

#### What does a Docker image contain?
1. Base OS (like `ubuntu`, `alpine`, `python:3.10-slim`, etc.)
2. App code (e.g. your `app.py`)
3. Libraries/dependencies (e.g. Flask, Django, numpy)
4. Startup command (e.g. `CMD ["python", "/app/app.py"]`)

Example:
```Dockerfile
FROM python:3.10-slim            # sets the base image for your new image.
COPY app.py /app/                # copies your local `app.py` file into the Docker image
CMD ["python", "/app/app.py"]    # tells Docker what command to run by default when the container starts
```
You can:
```bash
docker build -t myapp:latest .
```
| Part              | Meaning                                                                                             |
| ----------------- | --------------------------------------------------------------------------------------------------- |
| `docker build`    | This tells Docker to start **building a container image** using the instructions in the Dockerfile. |
| `-t myapp:latest` | Tags the resulting image with the name `myapp` and version `latest`.                                |
| `.`               | The **build context** – current directory. Docker will look here for the Dockerfile and `app.py`.   |

- You now have a layered, reusable Docker image.
- You can push it to **DockerHub**, use it in **Kubernetes**, or run it directly via `docker run`.

#### DevOps Use Case:
- In CI/CD, you build an image after code is merged into GitHub.
- The image (with version tag) is pushed to DockerHub or ECR.
- The deployment script (Ansible/Terraform/K8s) pulls the image and runs it on the target server.

#### Why Images Matter:
- Portability: "It works on my machine" problem disappears.
- Version Control: Tag images for release tracking (myapp:1.0, myapp:latest).
- Immutable: You always know what’s inside; no surprises.

---

# 2. Docker Container:
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

---

# 3. Docker Volumes
A Volume is a persistent storage mechanism for containers.  
By default, data inside a container is lost when it stops.  

Use Volumes to:
- Retain logs, database files, configs
- Share data across containers
- Backup container state

Example:
```bash
docker run -v mydata:/var/lib/mysql mysql
```
Here, mydata is a named volume. MySQL stores its DB files there.

#### Types:
1. **Named volumes:** managed by Docker (`docker volume create myvol`)
3. **Bind mounts:** map host path to container (`/home/user/data:/data`)

#### DevOps Use Case:
- Mount config files or SSL certs into a container (`/etc/nginx/conf.d`)
- Persist database data (`postgres`, `mongodb`)
- Store logs and rotate them from host (`/var/log/nginx`)
- Backup Jenkins home directory using volume

#### Why Volumes Matter:
- Without volumes: container data is lost on restart
- Useful for stateful apps (DBs, CI/CD tools)
- Easier to migrate or backup volumes

---

# 4. Docker Networks
A Docker network lets containers communicate with each other.
> By default, all containers run in an isolated bridge network.

## Types of Networks:
| Type        | Description                   | Use Case                                 |
| ----------- | ----------------------------- | ---------------------------------------- |
| **Bridge**  | Default local network         | Single-host setups                       |
| **Host**    | Shares host network           | Performance-sensitive (e.g., Prometheus) |
| **None**    | No networking                 | Security or IPC-only                     |
| **Overlay** | Multi-host networking (Swarm) | Production-grade clusters                |
| **Macvlan** | Assign MAC/IP to container    | Bare-metal IP requirement                |


Example:
Create a bridge network:
```bash
docker network create myapp-net
```
Run two containers in it:
```bash
docker run -d --name db --network myapp-net postgres
docker run -d --name api --network myapp-net myapi:latest
```
Now api container can access db using hostname db.

#### DevOps Use Case:
- Microservices talking to each other inside a common Docker bridge
- Isolating environments: staging vs production networks
- Setting up service discovery (especially in Docker Compose)
- Simulating real-world network configs in local setups

#### Why Networks Matter:
- Prevent cross-container leaks (security)
- Name-based DNS resolution (instead of using IPs)
- Scalable internal communication

## Summary Table

| Component     | Description             | Real Use Case                | Why It Matters                |
| ------------- | ----------------------- | ---------------------------- | ----------------------------- |
| **Image**     | Read-only template      | Build once, deploy anywhere  | Portability & version control |
| **Container** | Running app from image  | Microservices isolation      | Lightweight, scalable, fast   |
| **Volume**    | Persistent data storage | DB backups, config files     | Data durability               |
| **Network**   | Container communication | Service discovery, isolation | Secure internal connectivity  |

## Example Scenario (CI/CD DevOps)
Imagine you're deploying a Java + MySQL app:

1. Build image from Dockerfile and tag it:
```bash
docker build -t myapp:1.0 .
```
2. Push to DockerHub/ECR after tests pass:
```bash
docker push myrepo/myapp:1.0
```
3. In staging:
   - Pull the image
   - Start MySQL with volume
```bash
docker run -d --name mysql -v mysql_data:/var/lib/mysql mysql
```
   - Run app in same network

```bash
docker network create backend
docker run -d --name app --network backend myrepo/myapp:1.0
```

4. App talks to MySQL via hostname mysql

This is how images, containers, volumes, and networks work together to build real-world systems.



