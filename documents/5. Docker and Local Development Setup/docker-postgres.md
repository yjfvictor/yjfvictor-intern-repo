# 📌 Running PostgreSQL in Docker

## 🎯 Goal

Set up and run a **PostgreSQL database in Docker** for local development.

---

## ✅ Why is this important?

Focus Bear's backend uses **PostgreSQL** as its primary database. Running it in Docker ensures a **consistent development environment** without installing PostgreSQL directly on your machine.

---

## ✅ Setup Instructions

### 1. Create `docker-compose.yml`

Create a `docker-compose.yml` file in the project root with the following configuration:

```yaml
version: '3.8'

services:
  postgres:
    image: postgres:15-alpine
    container_name: focusbear-postgres
    restart: always
    environment:
      POSTGRES_USER: focusbear_user
      POSTGRES_PASSWORD: focusbear_password
      POSTGRES_DB: focusbear_db
      LANG: en_GB.UTF-8
      LC_CTYPE: en_GB.UTF-8
      LC_COLLATE: en_GB.UTF-8
    ports:
      - "5432:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U focusbear_user -d focusbear_db"]
      interval: 10s
      timeout: 5s
      retries: 5

volumes:
  postgres_data:
    driver: local
```

### 2. Start PostgreSQL

Run the following command to start the PostgreSQL container:

```bash
docker-compose up -d
```

This starts PostgreSQL in detached mode (background). The `-d` flag runs containers in the background.

### 3. Verify the container is running

Check that the container is running:

```bash
docker ps
```

You should see `focusbear-postgres` in the list of running containers.

### 4. Connect to PostgreSQL

#### Using `psql` (command-line client)

Connect using the `psql` command-line client:

```bash
docker exec -it focusbear-postgres psql -U focusbear_user -d focusbear_db
```

Alternatively, if you have `psql` installed locally:

```bash
psql -h localhost -p 5432 -U focusbear_user -d focusbear_db
```

When prompted, enter the password: `focusbear_password`

#### Using pgAdmin (GUI client)

1. Install **pgAdmin** (if not already installed):
   - Download from: <https://www.pgadmin.org/download/>
   - Or install via package manager: `sudo apt-get install pgadmin4` (Linux)

2. Open pgAdmin and create a new server connection:
   - **Host**: `localhost`
   - **Port**: `5432`
   - **Database**: `focusbear_db`
   - **Username**: `focusbear_user`
   - **Password**: `focusbear_password`

3. Save and connect to the database.

### 5. Test the connection

Once connected, try running a simple SQL query:

```sql
SELECT version();
```

This should display the PostgreSQL version information.

---

## ✅ Reflection

### 1. What are the benefits of running PostgreSQL in a Docker container?

Running PostgreSQL in a Docker container offers several significant advantages:

#### Consistency across environments

- The same PostgreSQL version and configuration runs identically on any machine (development, staging, production).
- Eliminates "it works on my machine" issues by ensuring all developers use the same database setup.
- Reduces environment-specific bugs and configuration drift.

#### Isolation and portability

- The database runs in an isolated environment, separate from the host system.
- No need to install PostgreSQL directly on the host machine, avoiding conflicts with system packages.
- Easy to switch between different PostgreSQL versions by changing the image tag (e.g., `postgres:14-alpine` vs `postgres:15-alpine`).

#### Easy setup and teardown

- Start the database with a single command: `docker-compose up -d`.
- Stop and remove everything cleanly: `docker-compose down`.
- Perfect for quick prototyping, testing, and development workflows.

#### Resource management

- Docker allows setting memory and CPU limits for the container.
- Easy to run multiple database instances (different versions or configurations) on the same machine without port conflicts.

#### Version control integration

- The `docker-compose.yml` file can be committed to version control, ensuring all team members use identical database configurations.
- Changes to database setup are tracked and reviewed like any other code change.

#### Integration with other services

- Easy to orchestrate multiple services (database, application, Redis, etc.) together using Docker Compose.
- Services can communicate via Docker networks without exposing ports to the host.

---

### 2. How do Docker volumes help persist PostgreSQL data?

Docker volumes are essential for data persistence in containerised databases. Without volumes, all data stored in a container's filesystem is lost when the container is removed.

#### What happens without volumes

When a container is created, it has its own isolated filesystem. Any data written to this filesystem (including PostgreSQL's data directory at `/var/lib/postgresql/data`) exists only within the container. If the container is stopped and removed, all data is permanently deleted.

#### How volumes solve this

Docker volumes are managed storage areas that exist independently of containers:

1. **Named volumes** (as used in our `docker-compose.yml`):
   - Created with a specific name (e.g., `postgres_data`).
   - Managed by Docker and stored in Docker's storage directory (typically `/var/lib/docker/volumes/` on Linux).
   - Persist even when containers are removed.
   - Can be shared between multiple containers if needed.

2. **Volume mounting**:
   - In our configuration, `postgres_data:/var/lib/postgresql/data` mounts the named volume to PostgreSQL's data directory.
   - All database files, tables, and data are stored in this volume.
   - When the container restarts or is recreated, it reattaches to the same volume, preserving all data.

#### Practical example

```bash
# Start PostgreSQL and create some data
docker-compose up -d
docker exec -it focusbear-postgres psql -U focusbear_user -d focusbear_db -c "CREATE TABLE test (id SERIAL PRIMARY KEY, name TEXT);"
docker exec -it focusbear-postgres psql -U focusbear_user -d focusbear_db -c "INSERT INTO test (name) VALUES ('Hello World');"

# Stop and remove the container
docker-compose down

# Start again - data is still there!
docker-compose up -d
docker exec -it focusbear-postgres psql -U focusbear_user -d focusbear_db -c "SELECT * FROM test;"
# The data persists because it's stored in the volume, not in the container.
```

#### Volume management commands

- **List volumes**: `docker volume ls`
- **Inspect a volume**: `docker volume inspect docker-foucusbear_postgres_data`
- **Remove a volume** (⚠️ **deletes all data**): `docker volume rm docker-foucusbear_postgres_data`
- **Remove volumes with containers**: `docker-compose down -v` (⚠️ **deletes all data**)

#### Best practices

- Always use named volumes for production databases (better isolation and portability than bind mounts).
- Avoid `docker-compose down -v` unless you intentionally want to delete all database data.
- Regularly back up volumes using `docker run --rm -v docker-foucusbear_postgres_data:/data -v $(pwd):/backup alpine tar -zcvf /backup/postgres_backup.tar.gz /data`.

---

### 3. How can you connect to a running PostgreSQL container?

There are several methods to connect to a PostgreSQL container, depending on your needs and available tools.

#### Method 1: Using `psql` inside the container

The simplest method is to use the `psql` client that comes with the PostgreSQL Docker image:

```bash
docker exec -it focusbear-postgres psql -U focusbear_user -d focusbear_db
```

- `docker exec -it`: Runs an interactive command in the running container.
- `focusbear-postgres`: The container name.
- `psql`: The PostgreSQL command-line client.
- `-U focusbear_user`: Specifies the username.
- `-d focusbear_db`: Specifies the database name.

This method does not require a password prompt because `psql` connects as a local user within the container (using peer authentication).

#### Method 2: Using `psql` from the host machine

If you have `psql` installed on your host machine, you can connect via the exposed port:

```bash
psql -h localhost -p 5432 -U focusbear_user -d focusbear_db
```

- `-h localhost`: Connect to localhost (the Docker host).
- `-p 5432`: Use port 5432 (mapped from the container).
- `-U focusbear_user`: Database username.
- `-d focusbear_db`: Database name.

You will be prompted for the password: `focusbear_password`.

#### Method 3: Using pgAdmin (GUI client)

1. **Install pgAdmin** (if not already installed):
   - Visit <https://www.pgadmin.org/download/>
   - Or use package manager: `sudo apt-get install pgadmin4` (Linux)

2. **Open pgAdmin** and right-click "Servers" → "Register" → "Server"

3. **Configure the connection**:
   - **General tab**:
     - **Name**: `Focus Bear Local` (any descriptive name)
   - **Connection tab**:
     - **Host name/address**: `localhost`
     - **Port**: `5432`
     - **Maintenance database**: `focusbear_db`
     - **Username**: `focusbear_user`
     - **Password**: `focusbear_password`
     - **Save password**: ✓ (optional, for convenience)

4. **Save and connect**. You should now see the database in pgAdmin's object browser.

#### Method 4: Using other database clients

Many database clients support PostgreSQL connections:

- **DBeaver**: Free, cross-platform database tool
- **DataGrip**: JetBrains' database IDE
- **TablePlus**: Modern database client (macOS, Windows, Linux)
- **Postico**: PostgreSQL client for macOS

All of these use the same connection parameters:

- **Host**: `localhost`
- **Port**: `5432`
- **Database**: `focusbear_db`
- **Username**: `focusbear_user`
- **Password**: `focusbear_password`

#### Method 5: Using connection strings

Many applications and tools accept PostgreSQL connection strings:

```text
postgresql://focusbear_user:focusbear_password@localhost:5432/focusbear_db
```

This format is commonly used in:

- Environment variables (e.g., `DATABASE_URL`)
- Application configuration files
- ORM libraries (TypeORM, Sequelize, etc.)

#### Verifying the connection

Once connected, test with a simple query:

```sql
SELECT version();
SELECT current_database();
SELECT current_user;
```

These commands confirm you are connected and show the PostgreSQL version, current database, and current user.

---

## ✅ Summary

Running PostgreSQL in Docker provides a consistent, isolated, and portable database environment for local development. Key takeaways:

- **Use `docker-compose.yml`** to define PostgreSQL services with proper environment variables, port mappings, and volumes.
- **Always use volumes** (`postgres_data:/var/lib/postgresql/data`) to persist database data across container restarts and removals.
- **Connect using `docker exec -it`** for quick command-line access, or use GUI clients like pgAdmin for visual database management.
- **Expose port 5432** to allow connections from the host machine and other applications.
- **Use healthchecks** to ensure the database is ready before other services attempt to connect.

This setup ensures that all developers working on Focus Bear's backend have an identical PostgreSQL environment, reducing configuration issues and improving development efficiency.
