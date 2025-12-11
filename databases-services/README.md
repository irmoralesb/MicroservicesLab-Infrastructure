# Database Services

This directory contains Docker Compose configuration for running multiple database services in containerized environments. The `docker-container.yaml` file defines three database services: Microsoft SQL Server, PostgreSQL, and Redis.

## Goal

The `docker-container.yaml` file provides a unified configuration to spin up multiple database services simultaneously using Docker Compose. This setup is designed for development and testing environments where you need multiple database systems running in isolated containers with persistent data storage, health checks, and resource management.

## Services Overview

### 1. Microsoft SQL Server 2022

**Container Name:** `mssql-server-2022`  
**Hostname:** `mssql-server`  
**Image:** `mcr.microsoft.com/mssql/server:2022-latest`

#### Port Configuration
- **Default Port:** `1433`
- **Custom Port:** Set via `MSSQL_PORT` environment variable
- **Format:** `host_port:container_port`

#### Authentication

**Default Username:** `sa` (System Administrator)

**Default Password:** `YourStrong@Password123`

**Custom Password:**
```bash
# Set via environment variable
export SA_PASSWORD=YourCustomPassword123
docker-compose -f docker-container.yaml up -d
```

Or create a `.env` file in the same directory:
```
SA_PASSWORD=YourCustomPassword123
MSSQL_PORT=1433
```

#### Additional Configuration Options

- **MSSQL_PID:** SQL Server Product ID (default: `Developer`)
  - Options: `Developer`, `Express`, `Standard`, `Enterprise`, `EnterpriseCore`
- **MSSQL_AGENT_ENABLED:** Enable SQL Server Agent (default: `true`)
- **TZ:** Timezone (default: `UTC`)

#### Connection String Example
```
Server=localhost,1433;Database=master;User Id=sa;Password=YourStrong@Password123;
```

#### Resource Limits
- **CPU Limit:** 2.0 cores
- **Memory Limit:** 4GB
- **CPU Reservation:** 1.0 core
- **Memory Reservation:** 2GB

---

### 2. PostgreSQL 18

**Container Name:** `postgres-server`  
**Hostname:** `postgres-server`  
**Image:** `postgres:18-alpine`

#### Port Configuration
- **Default Port:** `5432`
- **Custom Port:** Set via `POSTGRES_PORT` environment variable
- **Format:** `host_port:container_port`

#### Authentication

**Default Username:** `postgres`

**Default Password:** `postgres`

**Custom Username and Password:**
```bash
# Set via environment variables
export POSTGRES_USER=myuser
export POSTGRES_PASSWORD=mypassword
docker-compose -f docker-container.yaml up -d
```

Or create a `.env` file:
```
POSTGRES_USER=myuser
POSTGRES_PASSWORD=mypassword
POSTGRES_DB=mydatabase
POSTGRES_PORT=5432
```

#### Additional Configuration Options

- **POSTGRES_DB:** Default database name (default: `postgres`)
- **TZ:** Timezone (default: `UTC`)

#### Connection String Example
```
postgresql://postgres:postgres@localhost:5432/postgres
```

#### Resource Limits
- **CPU Limit:** 2.0 cores
- **Memory Limit:** 4GB
- **CPU Reservation:** 1.0 core
- **Memory Reservation:** 2GB

---

### 3. Redis 8

**Container Name:** `redis-server`  
**Hostname:** `redis-server`  
**Image:** `redis:8-alpine`

#### Port Configuration
- **Default Port:** `6379`
- **Custom Port:** Set via `REDIS_PORT` environment variable
- **Format:** `host_port:container_port`

#### Authentication

**Default Password:** `redis`

**Note:** Redis does not use a username by default. Authentication is password-based only.

**Custom Password:**
```bash
# Set via environment variable
export REDIS_PASSWORD=myredispassword
docker-compose -f docker-container.yaml up -d
```

Or create a `.env` file:
```
REDIS_PASSWORD=myredispassword
REDIS_PORT=6379
```

#### Redis Configuration

The Redis server is configured with:
- **Password Protection:** Enabled via `--requirepass`
- **Persistence:** RDB snapshots every 60 seconds if at least 1 key changed
- **AOF (Append Only File):** Enabled for durability
- **Log Level:** Warning

#### Connection Example
```bash
# Using redis-cli
redis-cli -h localhost -p 6379 -a redis
```

#### Resource Limits
- **CPU Limit:** 1.0 core
- **Memory Limit:** 2GB
- **CPU Reservation:** 0.5 core
- **Memory Reservation:** 512MB

---

## Usage

### Starting Services

```bash
# Start all services
docker-compose -f docker-container.yaml up -d

# Start specific service
docker-compose -f docker-container.yaml up -d mssql-server
docker-compose -f docker-container.yaml up -d postgres
docker-compose -f docker-container.yaml up -d redis
```

### Stopping Services

```bash
# Stop all services
docker-compose -f docker-container.yaml down

# Stop specific service
docker-compose -f docker-container.yaml stop mssql-server
```

### Using Custom Credentials

#### Method 1: Environment Variables

```bash
export SA_PASSWORD=MySqlServerPassword123
export POSTGRES_USER=myuser
export POSTGRES_PASSWORD=mypassword
export POSTGRES_DB=mydb
export REDIS_PASSWORD=myredispass
export MSSQL_PORT=1433
export POSTGRES_PORT=5432
export REDIS_PORT=6379

docker-compose -f docker-container.yaml up -d
```

#### Method 2: .env File

Docker Compose **automatically** looks for a `.env` file in the **current working directory** where you run the command.

**Important:** When using `-f docker-container.yaml`, Docker Compose looks for `.env` in the directory where you execute the command, not necessarily where the compose file is located.

**Recommended approach:** Create a `.env` file in the `databases-services` directory and run the command from that same directory:

```bash
cd databases-services
```

Create the `.env` file (you can copy from `.env_template` and customize):

```env
# SQL Server
SA_PASSWORD=MySqlServerPassword123
MSSQL_PORT=1433
MSSQL_PID=Developer
MSSQL_AGENT_ENABLED=true

# PostgreSQL
POSTGRES_USER=myuser
POSTGRES_PASSWORD=mypassword
POSTGRES_DB=mydb
POSTGRES_PORT=5432

# Redis
REDIS_PASSWORD=myredispass
REDIS_PORT=6379

# Common
TZ=UTC
```

Then run:
```bash
docker-compose -f docker-container.yaml up -d
```

**Note:** If you run the command from a different directory, make sure the `.env` file exists in that directory, or use the `--env-file` flag to specify a custom path:
```bash
docker-compose -f docker-container.yaml --env-file databases-services/.env up -d
```

#### Method 3: Inline with docker-compose

```bash
SA_PASSWORD=MyPassword123 POSTGRES_USER=admin POSTGRES_PASSWORD=admin123 REDIS_PASSWORD=redis123 \
docker-compose -f docker-container.yaml up -d
```

## Data Persistence

All services use Docker volumes to persist data:

- **MSSQL:** `mssql_data` → `/var/opt/mssql`
- **PostgreSQL:** `postgres_data` → `/var/lib/postgresql/data`
- **Redis:** `redis_data` → `/data`

Data persists even when containers are stopped or removed (unless volumes are explicitly deleted).

## Networks

Each service runs on its own isolated network:

- `mssql-network` (bridge)
- `postgres-network` (bridge)
- `redis-network` (bridge)

## Health Checks

All services include health checks to ensure they're ready:

- **MSSQL:** Checks SQL Server connectivity every 30s
- **PostgreSQL:** Checks server readiness every 10s
- **Redis:** Pings Redis server every 10s

## Security Notes

⚠️ **Important:** The default passwords in this configuration are for development purposes only. For production environments:

1. Always use strong, unique passwords
2. Use environment variables or secrets management
3. Never commit passwords to version control
4. Consider using Docker secrets for production deployments
5. Restrict network access appropriately

## Troubleshooting

### Check Service Status
```bash
docker-compose -f docker-container.yaml ps
```

### View Logs
```bash
# All services
docker-compose -f docker-container.yaml logs

# Specific service
docker-compose -f docker-container.yaml logs mssql-server
docker-compose -f docker-container.yaml logs postgres
docker-compose -f docker-container.yaml logs redis
```

### Check Health Status
```bash
docker inspect --format='{{.State.Health.Status}}' mssql-server-2022
docker inspect --format='{{.State.Health.Status}}' postgres-server
docker inspect --format='{{.State.Health.Status}}' redis-server
```
