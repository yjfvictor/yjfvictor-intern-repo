# 📌 Setting Up Docker and Docker Compose

## 🎯 Goal

Install **Docker and Docker Compose** and learn basic commands to manage containers.

---

## ✅ Installation Status

Docker and Docker Compose are already installed on this system:

- **Docker version**: 28.2.2
- **Docker Compose version**: 1.29.2

### Verification Commands

```bash
docker --version
docker-compose --version
```

Both commands confirmed successful installation.

---

## ✅ Key Docker Commands

### Container Management

- **`docker ps`**: Lists all running containers
  - Shows container ID, image, command, status, ports, and names
  - Example output shows `focusbear-postgres` running PostgreSQL 15-alpine

- **`docker ps -a`**: Lists all containers (running and stopped)
  - Useful for seeing containers that have exited or been stopped

- **`docker stop <container_name>`**: Stops a running container gracefully
  - Sends SIGTERM signal, allowing the container to shut down cleanly

- **`docker start <container_name>`**: Starts a stopped container
  - Container resumes from its previous state

- **`docker restart <container_name>`**: Restarts a running container
  - Equivalent to `docker stop` followed by `docker start`

- **`docker rm <container_name>`**: Removes a stopped container
  - Use `docker rm -f` to force remove a running container

### Logging and Monitoring

- **`docker logs <container_name>`**: Displays logs from a container
  - Shows all log output from the container's stdout and stderr

- **`docker logs --tail <n> <container_name>`**: Shows the last N lines of logs
  - Example: `docker logs --tail 20 focusbear-postgres` shows the last 20 log lines

- **`docker logs -f <container_name>`**: Follows logs in real-time (like `tail -f`)
  - Continuously streams new log entries as they are generated

### Image Management

- **`docker images`**: Lists all Docker images on the system
  - Shows repository, tag, image ID, creation date, and size
  - Current images include `postgres:15-alpine`, `alpine:latest`, and others

- **`docker pull <image>`**: Downloads an image from Docker Hub
  - Example: `docker pull postgres:15-alpine`

- **`docker build -t <name> <path>`**: Builds an image from a Dockerfile
  - Creates a new image based on the Dockerfile in the specified path

### Container Inspection

- **`docker inspect <container_name>`**: Returns detailed information about a container
  - Includes configuration, network settings, volumes, and more

- **`docker exec -it <container_name> <command>`**: Executes a command in a running container
  - `-it` flags enable interactive terminal mode
  - Example: `docker exec -it focusbear-postgres psql -U focusbear_user -d focusbear_db`

---

## ✅ Reflection

### 1. What is the difference between `docker run` and `docker-compose up`?

`docker run` and `docker-compose up` serve different purposes, though both start containers.

#### `docker run`

- **Single container**: Starts one container at a time
- **Command-line configuration**: All settings (ports, volumes, environment variables) must be specified as command-line arguments
- **Manual orchestration**: If multiple containers are needed, each must be started separately with individual `docker run` commands
- **Example**:

  ```bash
  docker run -d \
    --name focusbear-postgres \
    -e POSTGRES_USER=focusbear_user \
    -e POSTGRES_PASSWORD=focusbear_password \
    -e POSTGRES_DB=focusbear_db \
    -p 5432:5432 \
    -v postgres_data:/var/lib/postgresql/data \
    postgres:15-alpine
  ```

#### `docker-compose up`

- **Multi-container orchestration**: Starts multiple containers defined in a `docker-compose.yml` file
- **Declarative configuration**: All settings are defined in YAML format, making it easier to read, maintain, and version control
- **Automatic networking**: Containers defined in the same compose file are automatically connected to the same network and can communicate using service names
- **Dependency management**: Can define service dependencies (e.g., application waits for database to be healthy)
- **Single command**: One command starts all services: `docker-compose up -d`
- **Example** (`docker-compose.yml`):

  ```yaml
  version: '3.8'
  services:
    postgres:
      image: postgres:15-alpine
      environment:
        POSTGRES_USER: focusbear_user
        POSTGRES_PASSWORD: focusbear_password
        POSTGRES_DB: focusbear_db
      ports:
        - "5432:5432"
      volumes:
        - postgres_data:/var/lib/postgresql/data
  ```

#### Key Differences Summary

| Aspect | `docker run` | `docker-compose up` |
| --- | --- | --- |
| **Scope** | Single container | Multiple containers |
| **Configuration** | Command-line arguments | YAML file |
| **Complexity** | Verbose for complex setups | Clean and readable |
| **Networking** | Manual network creation | Automatic service networking |
| **Dependencies** | Manual ordering | Declarative dependencies |
| **Use case** | Quick testing, single services | Multi-service applications |

For Focus Bear's backend, which requires PostgreSQL, Redis, and the NestJS API, `docker-compose up` is the preferred approach as it manages all services together with a single command.

---

### 2. How does Docker Compose help when working with multiple services?

Docker Compose simplifies managing multi-service applications by providing orchestration, configuration management, and service coordination.

#### Unified Configuration

- **Single file**: All services are defined in one `docker-compose.yml` file
- **Version control**: The entire application stack configuration can be committed to Git
- **Team consistency**: All developers use the same service configuration
- **Easy updates**: Changing a service version or configuration only requires editing the YAML file

#### Automatic Service Networking

- **Default network**: All services in a compose file are automatically connected to the same Docker network
- **Service discovery**: Services can communicate using their service names as hostnames
  - Example: The NestJS API can connect to PostgreSQL using `postgres:5432` instead of `localhost:5432`
- **Isolation**: Services are isolated from containers outside the compose file

#### Dependency Management

- **Startup order**: Docker Compose can wait for services to be healthy before starting dependent services
- **Health checks**: Services can define health checks, and other services wait for them to be ready
- **Example**:

  ```yaml
  services:
    api:
      depends_on:
        postgres:
          condition: service_healthy
    postgres:
      healthcheck:
        test: ["CMD-SHELL", "pg_isready -U focusbear_user"]
  ```

#### Resource Management

- **Volume sharing**: Multiple services can share the same volumes
- **Port management**: Port conflicts are easier to manage when all port mappings are in one file
- **Environment variables**: Shared environment variables can be defined once and used by multiple services

#### Simplified Operations

- **Single command start**: `docker-compose up -d` starts all services
- **Single command stop**: `docker-compose down` stops and removes all services
- **Log aggregation**: `docker-compose logs` shows logs from all services
- **Service scaling**: `docker-compose up --scale api=3` can scale specific services

#### Example: Focus Bear Backend Stack

A typical `docker-compose.yml` for Focus Bear might include:

```yaml
version: '3.8'

services:
  postgres:
    image: postgres:15-alpine
    # Database configuration

  redis:
    image: redis:7-alpine
    # Cache configuration

  api:
    build: .
    depends_on:
      - postgres
      - redis
    environment:
      DATABASE_URL: postgresql://focusbear_user:focusbear_password@postgres:5432/focusbear_db
      REDIS_URL: redis://redis:6379
```

With Docker Compose, starting the entire stack is as simple as `docker-compose up -d`, and all services can communicate using their service names (`postgres`, `redis`, `api`).

---

### 3. What commands can you use to check logs from a running container?

Several Docker commands are available for viewing container logs, each suited to different use cases.

#### Basic Log Viewing

- **`docker logs <container_name>`**: Displays all logs from the container's start
  - Shows complete log history
  - Useful for reviewing what happened since the container started

- **`docker logs --tail <n> <container_name>`**: Shows only the last N lines
  - Example: `docker logs --tail 50 focusbear-postgres` shows the last 50 log lines
  - Useful when you only need recent log entries

- **`docker logs --since <time> <container_name>`**: Shows logs since a specific time
  - Example: `docker logs --since 10m focusbear-postgres` shows logs from the last 10 minutes
  - Supports formats like `10m`, `1h`, `2024-01-23T10:00:00`

- **`docker logs --until <time> <container_name>`**: Shows logs until a specific time
  - Useful for viewing logs within a specific time range

#### Real-time Log Following

- **`docker logs -f <container_name>`**: Follows logs in real-time
  - Continuously streams new log entries as they are generated
  - Similar to `tail -f` in Unix/Linux
  - Press `Ctrl+C` to stop following

- **`docker logs -f --tail <n> <container_name>`**: Combines following with tail
  - Shows the last N lines, then continues following new entries
  - Useful for catching up on recent logs before following

#### Docker Compose Log Commands

- **`docker-compose logs`**: Shows logs from all services in the compose file
  - Aggregates logs from all containers defined in `docker-compose.yml`

- **`docker-compose logs <service_name>`**: Shows logs from a specific service
  - Example: `docker-compose logs postgres` shows only PostgreSQL logs

- **`docker-compose logs -f`**: Follows logs from all services in real-time
  - Useful for monitoring the entire application stack

- **`docker-compose logs -f <service_name>`**: Follows logs from a specific service

#### Practical Examples

```bash
# View all logs from PostgreSQL container
docker logs focusbear-postgres

# View last 20 lines of logs
docker logs --tail 20 focusbear-postgres

# Follow logs in real-time
docker logs -f focusbear-postgres

# View logs from last hour
docker logs --since 1h focusbear-postgres

# View logs with timestamps (if container supports it)
docker logs -t focusbear-postgres

# Using Docker Compose for multi-service applications
docker-compose logs
docker-compose logs -f api
```

#### Log Output Example

When running `docker logs focusbear-postgres --tail 20`, typical output might include:

```text
PostgreSQL Database directory appears to contain a database; Skipping initialization

2026-01-23 05:06:57.928 UTC [1] LOG:  starting PostgreSQL 15.15 on x86_64-pc-linux-musl
2026-01-23 05:06:57.928 UTC [1] LOG:  listening on IPv4 address "0.0.0.0", port 5432
2026-01-23 05:06:57.928 UTC [1] LOG:  listening on IPv6 address "::", port 5432
2026-01-23 05:06:57.933 UTC [1] LOG:  listening on Unix socket "/var/run/postgresql/.s.PGSQL.5432"
2026-01-23 05:06:57.941 UTC [29] LOG:  database system was shut down at 2026-01-23 05:06:45 UTC
2026-01-23 05:06:57.949 UTC [1] LOG:  database system is ready to accept connections
```

These log commands are essential for debugging issues, monitoring application health, and understanding container behaviour.

---

### 4. What happens when you restart a container? Does data persist?

Container restarts behave differently depending on how data is stored, but with proper volume configuration, data persists across restarts.

#### Container Restart Behaviour

When a container is restarted using `docker restart <container_name>` or `docker-compose restart`:

1. **Container process stops**: The main process (e.g., PostgreSQL server) receives a termination signal and shuts down
2. **Container filesystem remains**: The container's filesystem is preserved (unless the container is removed)
3. **Container restarts**: The container starts again with the same configuration
4. **Process resumes**: The main process starts from the beginning (not from a saved state)

#### Data Persistence Mechanisms

Data persistence depends on where data is stored:

##### 1. **Data in Volumes (Persists)**

When data is stored in Docker volumes (as configured in `docker-compose.yml`):

```yaml
volumes:
  - postgres_data:/var/lib/postgresql/data
```

- **Data persists**: All data in the volume remains intact across restarts
- **Volume independence**: Volumes exist independently of containers
- **Example**: PostgreSQL data stored in `postgres_data` volume persists even if the container is removed and recreated

##### 2. **Data in Container Filesystem (Temporary)**

Data written directly to the container's filesystem (not in a volume):

- **Lost on container removal**: If the container is removed (`docker rm`), all data in the container filesystem is deleted
- **Preserved on restart**: Data remains if the container is only restarted (not removed)
- **Not recommended**: Should not be used for important data

##### 3. **Data in Bind Mounts (Persists)**

When using bind mounts (mounting host directories):

```yaml
volumes:
  - ./data:/var/lib/postgresql/data
```

- **Data persists**: Data is stored on the host filesystem
- **Host dependency**: Data location depends on the host machine
- **Portability**: Less portable than named volumes

#### Practical Example: PostgreSQL Container

For the `focusbear-postgres` container:

1. **Database data is in a volume**: `postgres_data:/var/lib/postgresql/data`
2. **Restarting the container**:

   ```bash
   docker restart focusbear-postgres
   ```

   - PostgreSQL stops and starts again
   - **All database data persists** because it is stored in the `postgres_data` volume
   - Tables, records, and configurations remain intact

3. **Removing and recreating the container**:

   ```bash
   docker-compose down
   docker-compose up -d
   ```

   - Old container is removed, new container is created
   - **Data still persists** because the volume is not removed (unless using `docker-compose down -v`)

4. **Removing the volume** (⚠️ **deletes all data**):

   ```bash
   docker-compose down -v
   ```

   - Removes containers and volumes
   - **All data is permanently deleted**

#### Verification

To verify data persistence:

```bash
# Create some data
docker exec -it focusbear-postgres psql -U focusbear_user -d focusbear_db -c "CREATE TABLE test (id SERIAL PRIMARY KEY, name TEXT);"
docker exec -it focusbear-postgres psql -U focusbear_user -d focusbear_db -c "INSERT INTO test (name) VALUES ('Test Data');"

# Restart the container
docker restart focusbear-postgres

# Wait for container to be ready, then verify data still exists
docker exec -it focusbear-postgres psql -U focusbear_user -d focusbear_db -c "SELECT * FROM test;"
# Output should show: "Test Data" - data persists!
```

#### Key Takeaways

- **Restart preserves data**: Restarting a container does not delete data stored in volumes
- **Volume configuration is critical**: Always use volumes for databases and other persistent data
- **Container removal**: Removing a container does not delete volumes (data persists)
- **Volume removal**: Only removing volumes deletes the data permanently
- **Application state**: Application processes restart from the beginning, but data in volumes remains

For Focus Bear's PostgreSQL container, the database data persists across restarts because it is properly configured with a named volume (`postgres_data`), ensuring that database records, tables, and configurations are not lost when containers are restarted or recreated.

---

## ✅ Summary

Docker and Docker Compose are essential tools for managing containerised applications. Key takeaways:

- **Docker commands**: `docker ps`, `docker logs`, `docker stop`, `docker restart` are fundamental for container management
- **Docker Compose**: Simplifies multi-service applications with declarative YAML configuration
- **Log management**: Multiple commands available for viewing and following container logs
- **Data persistence**: Properly configured volumes ensure data persists across container restarts

Understanding these concepts and commands is crucial for working with Focus Bear's backend, which relies on Docker Compose to orchestrate PostgreSQL, Redis, and the NestJS API services.
