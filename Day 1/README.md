# Day 1: Docker Fundamentals for Kubernetes | CKA Certification Course 2025

## Video reference for Day 1 is the following:

[![Day 1: Docker Fundamentals for Kubernetes](https://img.youtube.com/vi/p8VQmk_zbVo/1.jpg)](https://www.youtube.com/watch?v=p8VQmk_zbVo)

## How Were Development and Deployment Done Before Docker? Why do We Need Docker? What Challenges Does it Solve?

![Alt text](/images/1a.png)

Before Docker, the software lifecycle involved:

 1. **Development:**
	- Developers manually installed required software on their machines.
	- Teams wrote detailed setup instructions for different environments (development, testing, production).
 2. **Testing:**
	- Test environments were manually configured to resemble production as closely as possible.
	- Issues often arose because environments weren’t identical.
3. **Deployment:**
	- Applications were packaged into .tar.gz files or .zip archives.
	- Scripts or manual processes were used to copy files, configure dependencies, and start services.

Example of the Problem:
 - Developer A writes code using Python 3.8, but the production server has Python 3.9, causing compatibility issues.
- Fixing these issues required manual intervention, leading to delays.

## What is Docker?

![Alt text](/images/1b.png)

**Before Docker:** Applications were “manually loaded” with their dependencies into each environment. If your production server had a different configuration than your development system, things could break.

**With Docker:** Applications and dependencies are “packed” into containers, ensuring seamless movement between development, testing, and production.

## Virtual machine (VM) vs Docker Architecture

![Alt text](/images/1c.png)

| Feature                  | Virtual Machines (VMs)                        | Docker Containers                              |
|--------------------------|-----------------------------------------------|-----------------------------------------------|
| **Architecture**         | Full OS with hypervisor                      | Shares the host OS kernel                     |
| **Weight**               | Heavyweight (includes guest OS)              | Lightweight (no guest OS)                     |
| **Startup Time**         | Slow (minutes to boot a VM)                  | Fast (seconds to start a container)           |
| **Resource Usage**       | High (duplicates OS for each VM)             | Low (shares host OS resources)                |
| **Isolation**            | Full hardware-level isolation                | Process-level isolation                       |
| **Deployment**           | Slower and resource-intensive                | Faster and efficient                          |
| **Use Case**             | Running multiple OSs or legacy systems       | Cloud-native apps and microservices           |
| **Example**              | Running Ubuntu on a Windows host (Hyper-V)            | Running an app in a containerized environment |

## High-Level Workflow with Docker

![Alt text](/images/1d.png)

## Low-Level Workflow with Docker

**Key Concepts:**

 1. Docker Images: The blueprint for containers. Code+Dependencies.
 2. Docker Containers: The running instances of images.
 3. Docker Registries: Where images are stored (e.g., Docker Hub).
 4. Docker Engine:
	- Docker CLI: Tool used to interact with Docker
	- REST API: Translates CLI commands (like docker run or docker pull) into HTTP requests that the Docker Daemon understands. It allows external tools, scripts, or applications to interact programmatically with Docker, making automation and integration possible.
	- Docker Daemon: Brains of Docker. It listens for requests from the CLI or REST API.

**Using the car analogy here:**

- Docker CLI: Visualize it as the car dashboard where user commands are entered.
- Docker REST API: Visualize it as wires transmitting commands from the dashboard to the engine.
- Docker Daemon: Visualize it as the engine performing tasks like pulling images and running containers.

![Alt text](/images/1e.png)

