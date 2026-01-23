# 📌 Debugging & Managing Docker Containers

## 🎯 Goal

Learn how to **inspect, debug, and manage** running Docker containers effectively.

---

## ✅ Key Commands Reference

### Inspecting running containers

- **`docker ps`** (or `docker container ls`): Lists running containers. Use `-a` or `--all` to include stopped containers. Options include `--size` / `-s` (file sizes), `--no-trunc` (full output), `--filter` (filter by state, name, etc.), and `--format` (custom output).
- **`docker inspect <name|id>`**: Returns detailed low-level information (JSON by default) about a container: config, network settings, mounts, environment, and more. Use `--format` / `-f` with Go templates to extract specific fields (e.g. IP address, env vars).

### Checking logs

- **`docker logs <name|id>`**: Displays a container’s **STDOUT** and **STDERR**. Useful flags:
  - `-f` / `--follow`: Stream logs in real time (like `tail -f`).
  - `-t` / `--timestamps`: Add timestamps to each line.
  - `-n` / `--tail <N>`: Show only the last N lines.
  - `--since` / `--until`: Filter by time (e.g. `--since 1h`).
  - `--details`: Include extra metadata (e.g. env vars, labels).

### Entering a running container

- **`docker exec -it <name|id> <command>`**: Runs a **new** command inside a running container. Typically used with `bash` or `sh` for an interactive shell:
  - `-i`: Keep STDIN open (interactive).
  - `-t`: Allocate a pseudo-TTY.
  - Example: `docker exec -it myapp sh`

### Removing, restarting, and rebuilding

- **`docker stop <name|id>`**: Stops a running container gracefully (SIGTERM, then SIGKILL after timeout).
- **`docker rm <name|id>`**: Removes a **stopped** container. Use `-f` to force-remove a running one.
- **`docker restart <name|id>`**: Stops and starts the same container again.
- **`docker-compose down`**: Stops and removes containers, networks, and (optionally with `-v`) volumes defined in the Compose file.
- **`docker-compose up -d`**: Starts services in detached mode. Use **`docker-compose up -d --build`** to rebuild images and then start. For a full reset: **`docker-compose down && docker-compose up -d --build`**.

---

## ✅ Reflection

### 1. How can you check logs from a running container?

Use **`docker logs`** with the container name or ID:

```bash
docker logs <container_name_or_id>
```

For ongoing monitoring, use **`-f`** to follow output, **`-t`** for timestamps, and **`--tail N`** to limit to the last N lines:

```bash
docker logs -f -t --tail 100 <container_name_or_id>
```

Logs come from the container’s **STDOUT** and **STDERR**. If nothing appears, ensure the application writes to stdout/stderr rather than to log files. Some apps (e.g. certain web servers) write only to files; they must be configured to also log to stdout/stderr, or you will need to `docker exec` in and inspect those files directly.

---

### 2. What is the difference between `docker exec` and `docker attach`?

| Aspect | `docker exec` | `docker attach` |
| ------ | ------------- | --------------- |
| **Process** | Starts a **new** process (e.g. a shell) inside the container. | Connects your terminal to the **existing** main process (PID 1) of the container. |
| **Effect on container** | The main process keeps running. Exiting the exec session does **not** stop the container. | You are interacting with the main process. Sending **Ctrl+C** can signal and **stop** that process (and thus the container). |
| **Use case** | Debugging, running one-off commands, opening an interactive shell **without** affecting the app. | Watching or interacting **directly** with the main process output. |

**Practical takeaway:** Prefer **`docker exec -it`** for inspection and debugging (e.g. `docker exec -it myapp sh`). Use **`docker attach`** only when you intentionally want to attach to the main process; be aware that exiting or pressing Ctrl+C can stop the container.

---

### 3. How do you restart a container without losing data?

Data persistence depends on **where** the data lives:

- **Inside the container filesystem only**: Restarting with `docker restart` keeps the same container and its filesystem, so data persists. However, **removing** the container (`docker rm`) destroys that filesystem and any data not stored elsewhere.

- **In Docker volumes**: Data in **volumes** (named or anonymous) persists across container restarts, stops, and even removal. To restart without losing data:
  1. Use **`docker restart <name|id>`** for an existing container that already has volumes attached; both the container and its volume data are preserved.
  2. If you **recreate** the container (e.g. `docker-compose down && docker-compose up -d`), avoid `docker-compose down -v` so that **named volumes** are not removed. Ensure the new container mounts the **same** volumes (e.g. via Compose `volumes:`).

Summary: **Use volumes for anything that must persist.** Restart with `docker restart` or `docker-compose restart` when possible; when recreating containers, keep volumes and remount them so data is not lost.

---

### 4. How can you troubleshoot database connection issues inside a containerized NestJS app?

Common causes and checks:

1. **Hostname configuration**  
   Inside Docker, **`localhost`** refers to the **container itself**, not the host or other containers. The NestJS app must use the **database service name** (e.g. `db`, `postgres`, `pgsqldb`) as the host, as defined in `docker-compose.yml`. Verify `TYPEORM_HOST` or equivalent in your NestJS config and environment variables.

2. **Docker network**  
   The app and database must be on the **same Docker network**. In Compose, they usually share the default project network; if you define custom networks, attach both services to the same one. Use **`docker network inspect <network>`** and **`docker inspect <container>`** to confirm connectivity.

3. **Startup order**  
   The database may not be ready when NestJS starts. Use **`depends_on`** in Compose so the app starts after the DB. For stricter guarantees, use a healthcheck on the database and `depends_on` with `condition: service_healthy`, or add retry logic in the NestJS database module.

4. **Ports and connectivity**  
   Between containers, use the **internal** port (e.g. `5432` for PostgreSQL). Port mapping (e.g. `5432:5432`) is for **host** access. Ensure no typos in host, port, user, or password in config and env vars.

5. **Inspection and debugging**  
   - **`docker logs <nestjs_container>`**: Check for connection errors, timeouts, or “connection refused”.
   - **`docker exec -it <nestjs_container> sh`**: Verify env vars (`printenv`), and optionally use `nc` or similar to test connectivity to `db:5432`.
   - **`docker exec -it <db_container> sh`**: Confirm the DB is up (e.g. `pg_isready` for PostgreSQL) and that it accepts connections from the app container.

6. **Running app locally, DB in Docker**  
   Use **`localhost`** as the DB host in your NestJS config and ensure the DB service exposes its port (e.g. `5432:5432`). The situation is different from “app and DB both in Docker,” where the service name must be used.

By checking hostname, network, startup order, ports, and logs (and optionally using `docker exec` to inspect env and connectivity), you can systematically resolve most database connection issues in a containerized NestJS setup.

---

## ✅ Summary

When developing or debugging Focus Bear’s backend:

- Use **`docker ps`** and **`docker inspect`** to see what is running and inspect configuration.
- Use **`docker logs -f`** to watch application and error output.
- Use **`docker exec -it`** to run shells or one-off commands inside containers without stopping them.
- Use **`docker stop`** / **`docker rm`** for single containers, and **`docker-compose down`** / **`docker-compose up -d --build`** for full stacks.
- Rely on **volumes** for persistent data and avoid `docker-compose down -v` (or `docker volume rm`) when you need to keep that data.
- For NestJS and DB issues, verify **hostname** (service name), **networks**, **depends_on**/healthchecks, and **logs**, then use **`docker exec`** to validate env and connectivity.

These practices help diagnose and fix container and database issues quickly during local development and debugging.
