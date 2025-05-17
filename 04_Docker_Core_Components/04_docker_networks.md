# Docker Networks
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
