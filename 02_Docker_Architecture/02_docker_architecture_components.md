# Docker Architecture Components

# 1. Docker Engine
Docker Engine is the core client-server application that manages:
- Images
- Containers
- Networks
- Volumes

**It includes:**
1. Docker Daemon (`dockerd`)
2. REST API
3. Docker CLI (`docker`)

**Use Case in DevOps:** In your CI/CD pipeline (e.g., GitLab, Jenkins, AWS CodeBuild), Docker Engine builds the app container image and runs it.
```bash
docker build -t myapp:v1 .
docker run -p 8080:80 myapp:v1
```
These commands hit the Docker API, which tells dockerd to do the work.

### Docker CLI (`docker`) and Docker Daemon (`dockerd`)
#### A) `docker` – Docker CLI (Client)
This is the command-line interface you use.

It’s the tool you type commands into, like:
```bash
docker run nginx
docker build -t myapp .
docker ps
```
It doesn't run containers directly. Instead, it sends instructions (via HTTP REST API) to the Docker daemon.

Think of it as: "The voice/remote control that tells Docker what to do."

#### B) `dockerd` – Docker Daemon (Server)
This is the background service that actually does the work.  
It:
- Listens to API calls from the CLI (docker)
- Pulls images
- Builds images
- Starts/stops containers
- Manages networks and volumes

It is typically started at boot and runs as a system service.

**Think of it as:** "The engine inside Docker that listens and executes actual tasks."

You can check it with:
```bash
ps aux | grep dockerd
```

#### How `docker` and `dockerd` work together:
Example:
```bash
docker run -d nginx
```
This is what happens:
- The `docker` CLI sends a REST API request (like `POST /containers/create`) to the Docker Daemon (`dockerd`)
- `dockerd` checks if the `nginx` image exists
- If not, it pulls from DockerHub
- Then it starts the container using the underlying runtime


# 2. Docker Daemon (`dockerd`)
- It's the core background service that manages all Docker objects.
- Listens on a UNIX socket /var/run/docker.sock.
- It pulls images, starts/stops containers, manages volumes, networks, etc.

**How it works:** 
1. When you run `docker run ubuntu`, the CLI sends a request to `dockerd` through the Docker API.
2. Daemon looks for `ubuntu` image locally.
   - If not found: pulls from **DockerHub** (or private registry)
3. It then creates and starts a container from that image.

**Real-World Scenario:**  
You are deploying a Python app in your CI/CD system (like Jenkins or GitLab). The pipeline runs `docker build`, which calls `dockerd` to build an image from your Dockerfile, and then runs a container to execute tests.

# 3. Docker CLI (`docker` command)
- It's the command-line client that you use to talk to Docker.
- Sends commands via the Docker REST API to dockerd.

```bash
docker build -t myimage .
docker run -it --name dev-container myimage
docker ps
```
These are all client commands. Internally:
- CLI → REST API → dockerd → OCI runtime (runc/containerd)

**Example:** You use docker ps on a production host to debug which containers are running.
```bash
docker ps -a  # See all containers (running and exited)
docker logs mycontainer  # Debug logs
```

# 4. Docker Images and Containers
## a) Docker Image:
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


## b) Docker Container:
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


# 5. Docker Registry
A Docker Registry is a central place (server) to store, manage, version, and share Docker images.  
It can be:
- Public (anyone can access your image)
- Private (only authorized users can access)

#### Why do we need a Docker Registry?
In DevOps workflows, we don’t build Docker images locally on every server.  
Instead, we:
1. Build the image in CI/CD pipeline
2. Push it to a Docker registry
3. Pull it from the registry onto the production/staging/test server
4. Run containers from it

### Example DevOps Pipeline (CI/CD):
✅ Developer commits code to GitHub
✅ CI pipeline builds Docker image from Dockerfile
✅ Tags the image as myapi:v1.0
✅ Pushes it to Amazon ECR
🚀 Production server pulls this image
✅ Runs the new version in a container

### Popular Docker Registries:
| Registry                      | Description                                                            | Usage                               |
| ----------------------------- | ---------------------------------------------------------------------- | ----------------------------------- |
| **DockerHub**                 | Official & public registry at [hub.docker.com](https://hub.docker.com) | Default unless you specify          |
| **Amazon ECR**                | AWS's private, secure registry                                         | Used in AWS-based pipelines         |
| **GitHub Container Registry** | Tied to your GitHub account                                            | Use if your code is on GitHub       |
| **Harbor**                    | Open-source, self-hosted registry                                      | Use in private clouds / enterprises |

#### DockerHub Example
```bash
# Step 1: Login
docker login

# Step 2: Pull official NGINX image from DockerHub
docker pull nginx:latest

# Step 3: Tag your image to match your repo name
docker tag myapp:latest username/myapp:1.0

# Step 4: Push your image to DockerHub
docker push username/myapp:1.0
```
Now anyone can pull it using:
```bash
docker pull username/myapp:1.0
```

#### DevOps Scenario: Using Amazon ECR in CI/CD
Imagine you're working on a microservice in a company that uses AWS.  
Your CI/CD pipeline will do this:
1. Build Docker image from code
2. Authenticate to Amazon ECR using AWS CLI:
```bash
aws ecr get-login-password | docker login --username AWS --password-stdin <account-id>.dkr.ecr.region.amazonaws.com
```
3. Tag image:
```bash
docker tag myapi:latest <account-id>.dkr.ecr.region.amazonaws.com/myapi:latest
```
4. Push to ECR:
```bash
docker push <account-id>.dkr.ecr.region.amazonaws.com/myapi:latest
```
5. In Production (e.g. EC2 or ECS), you just:
```bash
docker pull <account-id>.dkr.ecr.region.amazonaws.com/myapi:latest
```
This allows teams to version, secure, and centrally manage containers for real deployments.


# 6. Low-Level Container Runtime: `containerd` and `runc`
These are the underlying engines that Docker (and Kubernetes) use to actually run containers on Linux systems.

## 1. What is `containerd`?
`containerd` is a **container lifecycle manager** — a daemon (background process) that:
- Pulls images
- Manages storage
- Manages networking
- Creates and runs containers

It fully complies with the OCI (Open Container Initiative) specifications

Example:
When you run:
```bash
docker run nginx
```
Docker does not do all the work itself. Instead, it:  
**Uses `containerd` behind the scenes to:**
- Pull the image from DockerHub
- Create a container snapshot
- Manage the container’s life cycle

## 2. What is `runc`?
`runc` is the lowest level tool that actually interfaces with the Linux kernel to create a container.
- It is invoked by containerd
- It uses Linux namespaces and cgroups to isolate and limit the process
- It runs containers according to OCI Runtime Spec

`runc` is a binary — not something you often run manually.  

Technical Flow Summary:
```text
Docker CLI (docker run nginx)
       ↓
Docker Daemon (dockerd)
       ↓
containerd
       ↓
runc
       ↓
Linux Kernel → runs the containerized process
```

