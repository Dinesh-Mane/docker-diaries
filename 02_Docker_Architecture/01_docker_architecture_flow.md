# What is Docker Architecture?
At a high level, Docker architecture consists of the following:
```lua
+-----------------------------+
|   Docker CLI (Client)      |  <-- you interact here via terminal
+-----------------------------+
             |
             v
+-----------------------------+
|     Docker Daemon (dockerd) |  <-- manages containers/images
+-----------------------------+
             |
             v
+-----------------------------+
|     Containerd / runc       |  <-- low-level container runtime
+-----------------------------+
             |
             v
+-----------------------------+
|   Linux Kernel (Namespaces, cgroups) |
+-----------------------------+
```
Also involved:
- Docker Registry (e.g., DockerHub)
- Docker Images & Containers
- Docker Compose / Swarm (for orchestration) — optional


