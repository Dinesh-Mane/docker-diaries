# How All Docker Components Work Together
What happens when you run:
```bash
docker run nginx
```
Let’s trace the flow step-by-step — just like how it happens under the hood.

## 1. You run a Docker CLI command
```bash
docker run nginx
```
This command is sent to the Docker Daemon using a REST API call over a UNIX socket:
```bash
/var/run/docker.sock
```
This is the communication bridge between Docker CLI and the Docker backend.

## 2. Docker Daemon (`dockerd`) gets the request
It’s responsible for managing images, containers, networks, volumes, etc.

The daemon:
- Checks if `nginx` image is available locally
- If not found, it pulls from the registry (by default: DockerHub)
```bash
docker pull nginx
```

## 3. Docker Daemon delegates to `containerd`
At this point, `dockerd` offloads the actual container creation task to `containerd` — the container runtime.

`containerd` is responsible for:
- Snapshotting the container filesystem
- Preparing the image layers
- Managing networking + storage
- Starting & stopping the container

It follows the OCI Runtime Spec and is used internally by Docker, though it can also be used standalone (like in Kubernetes).

## 4. `containerd` calls `runc`
`runc` is the actual binary that spawns the container process using:
- Namespaces (for isolation of process, user, network, etc.)
- Cgroups (for resource limits like CPU, memory)
- Seccomp/AppArmor (for syscall filtering)
- Mounts (to set up rootfs, volumes)

So `runc` creates the actual Linux process that becomes the `nginx` container.

## 5. You see the nginx container running
You can now check:
```bash
docker ps
```
Or attach:
```bash
docker exec -it <container-id> bash
```

---

## Docker Socket Security Note
```bash
/var/run/docker.sock
```
**Why it's dangerous:** This file is like root access to your Docker daemon. Anyone who can access this socket can:
- Run containers
- Mount filesystems
- Exfiltrate secrets
- Control the host

**Common mistake in DevOps:** Mounting the Docker socket into containers like this:
```yaml
volumes:
  - /var/run/docker.sock:/var/run/docker.sock
```
This gives the container full control over Docker — which is often used in tools like Docker-in-Docker (DinD) or Jenkins Docker builds, but it's risky.

#### Security Best Practices:
| Rule                                                                | Why                                          |
| ------------------------------------------------------------------- | -------------------------------------------- |
| Avoid mounting the Docker socket unless 100% needed              | Prevent privilege escalation                 |
| Use `docker group` wisely                                        | Group members can access root via the socket |
| Use tools like `gVisor` or `Kata Containers` for added isolation | Adds extra kernel-level protection           |
| Use tools like `dockle` or `docker-bench-security`               | Scan containers for security issues          |

## Real DevOps Scenario:
You’re configuring a CI/CD pipeline that builds Docker images. Some developers suggest mounting `/var/run/docker.sock` into a Jenkins agent container.

Instead, you: Use Docker **BuildKit** or **Kaniko** which builds containers without needing privileged socket access.

---

# Summary Flow:
```pgsql
You run → `docker run nginx`
          ↓
Docker CLI → talks to Docker Daemon (`dockerd`) over REST socket
          ↓
If needed → Daemon pulls image from DockerHub
          ↓
Then → Daemon asks `containerd` to create container
          ↓
Then → `containerd` uses `runc` to isolate and run the actual container
          ↓
Finally → You get an nginx process running in a container!
```
