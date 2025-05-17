# Docker Volumes
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
