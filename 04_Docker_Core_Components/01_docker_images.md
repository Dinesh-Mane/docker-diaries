Docker Image:
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

