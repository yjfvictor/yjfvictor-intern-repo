# Docker for NestJS Development

## Goal

Learn how to containerise a NestJS application using Docker and run it alongside PostgreSQL.

---

## Example repository

A working example with a multi-stage Dockerfile and Docker Compose (NestJS plus PostgreSQL) is available at [Using-Docker-for-NestJS-Development](https://github.com/yjfvictor/Using-Docker-for-NestJS-Development).

### Features

- **Multi-stage Dockerfile.** A builder stage installs dependencies and runs `npm run build`, and a runner stage copies only `dist` and production `node_modules`, keeping the final image small.
- **Security.** The runner stage uses a non-root user (`nestjs`) and `chown` so the app does not run as root.
- **Alpine base.** Both stages use `node:20-alpine` and the runner adds `libc6-compat` for compatibility.
- **Build context.** `.dockerignore` excludes `node_modules`, `dist`, `.git`, tests, and logs so builds are faster and more reliable.
- **Docker Compose.** PostgreSQL has a health check and the API uses `depends_on` with `condition: service_healthy` so the app starts after the database is ready. A named volume persists database data.

---

## Reflection

### How does a Dockerfile define a containerised NestJS application?

A Dockerfile defines a containerised NestJS application by specifying the build steps and runtime environment. It starts from a base image such as `node:20-alpine`, sets the working directory, and copies the application code. The Dockerfile installs dependencies with `npm ci`, compiles TypeScript with `npm run build`, and copies the resulting `dist` folder into the final image. The `CMD` instruction runs `node dist/main.js` to start the application. Environment variables such as `PORT` can be set to configure the server. A `.dockerignore` file excludes `node_modules`, `dist`, and other unnecessary files from the build context, reducing image size and build time.

### What is the purpose of a multi-stage build in Docker?

A multi-stage build uses multiple `FROM` instructions in a single Dockerfile. Each stage can have a different base image and perform different tasks. The purpose is to produce a smaller final image by separating the build environment from the runtime environment. In the builder stage, the application installs all dependencies (including devDependencies) and compiles TypeScript. In the runner stage, only the compiled `dist` output and production dependencies are copied. Build tools, source code, and development packages stay in the builder stage and do not appear in the final image. This can reduce image size by several hundred megabytes compared to a single-stage build.

### How does Docker Compose simplify running multiple services together?

Docker Compose defines multiple services in a single `docker-compose.yml` file. Instead of running separate `docker run` commands for each service, a single `docker-compose up -d` starts all services. Compose automatically creates a shared network so services can communicate using service names as hostnames. Dependencies between services can be declared with `depends_on`, and health checks ensure the database is ready before the API starts. Ports, environment variables, and volumes are defined in one place, making the stack easy to version control and share with the team. Stopping the stack is equally simple with `docker-compose down`.

### How can you expose API logs and debug a running container?

Use `docker logs <container_name>` to view logs from a container. The `-f` flag follows logs in real time. Use `--tail N` to see only the last N lines. For Docker Compose stacks, `docker-compose logs` shows logs from all services, and `docker-compose logs -f api` streams logs for the API service only. To debug interactively, run `docker exec -it <container_name> sh` to get a shell inside the container. From there you can inspect files, check processes, or run commands. Use `docker inspect <container_name>` for detailed configuration and network information. These tools help trace errors and understand container behaviour without modifying the image.
