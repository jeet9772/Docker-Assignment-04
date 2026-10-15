# Docker Compose Assignment-4: OT-Microservices

**Presented by:** Jeetendra Singh  
**Technology:** Docker, Docker Compose, MySQL  
**Repository:** https://github.com/opstree/OT-Microservices

## Overview

The objective of this assignment is to deploy the OT-Microservices application using Docker Compose, run it without source code on the deployment machine, share it with a teammate, and achieve high availability for the application services.

## Assignment Objectives

- Clone and run the complete OT-Microservices application.
- Run the application without keeping the source code locally.
- Run the application on a teammate's system without cloning the repository.
- Configure high availability for Employee, Salary, and Attendance services.
- **Bonus:** Run only Attendance and MySQL containers.

## Prerequisites

- Docker
- Docker Compose V2
- Git
- Docker Hub account for sharing container images

Verify installations:

```bash
docker --version
docker compose version
git --version
```

## Step 1: Clone and Run the Application

Clone the repository:

```bash
git clone https://github.com/opstree/OT-Microservices.git
cd OT-Microservices
```

<img width="1440" height="900" alt="Screenshot 2026-10-15 at 7 03 26 PM" src="https://github.com/user-attachments/assets/f7118670-2d63-407c-9d09-a8bdd2afc686" />

Check the project files:

```bash
ls -la
docker compose config --services
```

Build and start the application:

```bash
docker compose up -d --build
```

Verify running containers:

```bash
docker compose ps
docker compose logs --tail=100
```

**Expected result:** All required microservices and their dependencies start successfully.

## Step 2: Run the Application Without Source Code

To run the application without the source code, publish the required application images to Docker Hub and create a runtime Compose file that uses the published images.

### Build and Push Images

Use the actual Dockerfiles and service directories in the repository.

```bash
docker login
docker build -t DOCKERHUB_USERNAME/employee-service:1.0 PATH_TO_EMPLOYEE_SERVICE
docker build -t DOCKERHUB_USERNAME/salary-service:1.0 PATH_TO_SALARY_SERVICE
docker build -t DOCKERHUB_USERNAME/attendance-service:1.0 PATH_TO_ATTENDANCE_SERVICE
```

Push the images:

```bash
docker push DOCKERHUB_USERNAME/employee-service:1.0
docker push DOCKERHUB_USERNAME/salary-service:1.0
docker push DOCKERHUB_USERNAME/attendance-service:1.0
```

Replace the example username and paths with your actual values. Publish any other custom images required by the application.

### Prepare the Runtime Compose File

Create `compose.runtime.yml` using the published image names.

```yaml
services:
  employee:
    image: DOCKERHUB_USERNAME/employee-service:1.0

  salary:
    image: DOCKERHUB_USERNAME/salary-service:1.0

  attendance:
    image: DOCKERHUB_USERNAME/attendance-service:1.0

  # Add MySQL and other required services.
  # Preserve the original environment variables,
  # networks, ports, volumes and dependencies.
```

This is a template. Copy all required configuration from the original Compose file and replace the illustrative service names and image names with the actual ones.

Validate the configuration:

```bash
docker compose -f compose.runtime.yml config
```

Keep the runtime Compose file and required environment configuration outside the source directory. After confirming that all images have been pushed, remove the source checkout if required.

Start the application from the deployment directory:

```bash
docker compose -f compose.runtime.yml pull
docker compose -f compose.runtime.yml up -d
docker compose -f compose.runtime.yml ps
```

**Expected result:** The application runs using published images without cloning or building the source code on the deployment machine.

## Step 3: Run on a Teammate's System Without Cloning

Share these files and details with your teammate:

- `compose.runtime.yml`
- Required environment configuration, shared securely
- Docker image names and tags
- Application ports and access instructions

On the teammate's system, install Docker and Docker Compose, save the shared files, and execute:

```bash
docker compose -f compose.runtime.yml pull
docker compose -f compose.runtime.yml up -d
docker compose -f compose.runtime.yml ps
docker compose -f compose.runtime.yml logs --tail=100
```

If the images are private, log in to Docker Hub first:

```bash
docker login
```

**Expected result:** The teammate can run the application without cloning the OT-Microservices repository.

## Step 4: High Availability for Microservices

Docker Compose supports multiple instances of a service when the application supports scaling.

Check the service names:

```bash
docker compose config --services
```

Scale Employee, Salary, and Attendance to three instances each, adjusting names as necessary:

```bash
docker compose up -d --scale employee=3 --scale salary=3 --scale attendance=3
```

Verify the services:

```bash
docker compose ps
docker compose logs --tail=100
```

### Important Considerations

- Remove fixed `container_name` values from services that need multiple replicas.
- Avoid publishing the same fixed host port for every replica.
- Use a load balancer or reverse proxy for incoming traffic where required.
- Configure health checks and appropriate retry and timeout behavior.
- Ensure application instances can safely share databases and other dependencies.
- A single Docker host or database can still be a single point of failure. Production-grade high availability requires a suitable multi-host architecture.

**Expected result:** Multiple instances of the Employee, Salary, and Attendance services run successfully where supported.

## Bonus: Run Only Attendance and MySQL

List the Compose service names:

```bash
docker compose config --services
```

If the services are named `attendance` and `mysql`, run:

```bash
docker compose up -d mysql attendance
```

Check their status and logs:

```bash
docker compose ps
docker compose logs --tail=100 mysql attendance
```

Compose will also start dependencies required by the selected services according to the configuration.

**Note:** MySQL may take time to become ready. Configure a health check and suitable readiness conditions if Attendance must wait for the database to be ready.

## Useful Docker Compose Commands

| Command | Description |
|---|---|
| `docker compose up -d` | Start containers in the background |
| `docker compose up -d --build` | Build images and start containers |
| `docker compose ps` | Display container status |
| `docker compose logs -f` | Follow application logs |
| `docker compose pull` | Download images |
| `docker compose config` | Validate Compose configuration |
| `docker compose config --services` | List service names |
| `docker compose restart` | Restart services |
| `docker compose stop` | Stop services |
| `docker compose down` | Remove project containers and network |
| `docker compose up -d --scale SERVICE=3` | Scale a service to three instances |

## Troubleshooting

### Container Is Not Starting

```bash
docker compose ps
docker compose logs SERVICE_NAME
```

### Port Conflict

```bash
docker ps
```

Check the conflicting port and update the host port mapping if necessary.

### MySQL Connection Error

- Verify database credentials and environment variables.
- Check the service name and network configuration.
- Use the Compose service name as the database hostname instead of `localhost` when connecting from another container.
- Check MySQL health and logs.

### Image Pull Error

- Verify the image name and tag.
- Ensure the image has been pushed to the registry.
- Authenticate if the image is private.

## Assignment Checklist

- [ ] Cloned and ran the complete application.
- [ ] Built and pushed application images to Docker Hub.
- [ ] Created a runtime Compose file using published images.
- [ ] Ran the application without the source checkout.
- [ ] Ran the application on a teammate's system without cloning.
- [ ] Configured multiple instances of Employee, Salary, and Attendance where supported.
- [ ] Tested the bonus task using only Attendance and MySQL.
- [ ] Verified container status, logs, and application endpoints.

## Conclusion

This assignment demonstrates how Docker Compose simplifies multi-container application deployment, enables image-based portability, supports service scaling, and allows selected services to be started independently.

**Source Repository:** https://github.com/opstree/OT-Microservices

**Regards,**  
**Jeetendra Singh**
