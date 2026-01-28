# Docker Workshop - NYC Taxi Data Pipeline

A Docker-based data pipeline for ingesting NYC taxi data into PostgreSQL. This project demonstrates containerized data engineering with Docker and Docker Compose.

## Prerequisites

- Docker and Docker Compose installed
- Python 3.13+ (for local development)
- Git

## Project Structure

```
docker-workshop/
├── pipeline/
│   ├── Dockerfile              # Docker image definition
│   ├── docker-compose.yaml     # Multi-container orchestration
│   ├── ingest_data.py          # Data ingestion script
│   ├── main.py                 # Main entry point
│   ├── data_pipeline.py        # Pipeline logic
│   ├── pyproject.toml          # Python dependencies
│   └── notebook.ipynb          # Jupyter notebook for exploration
└── test/                       # Test files and utilities
```

## Quick Start

### Using Docker Compose (Recommended)

1. **Create environment configuration**

   Create a `.env` file in the `pipeline/` directory:

   ```env
      POSTGRES_USER=<username>
      POSTGRES_PASSWORD=<password>
      POSTGRES_DB=<db_name>
      PGADMIN_EMAIL=<user_email>
      PGADMIN_PASSWORD=<password>
      PG_USER=<username>
      PG_PASS=<password>
      PG_HOST=<postgres container name>
      PG_PORT=5432
      PG_DB=<db_name>
   ```

2. **Start the services**

   ```bash
   cd pipeline/
   docker-compose up -d
   ```

   This starts:
   - PostgreSQL database (port 5432)
   - pgAdmin interface (port 8085) - Access at http://localhost:8085
   - Taxi data ingest service - Automatically runs the data pipeline

3. **Monitor the ingestion**

   ```bash
   docker-compose logs -f taxi_ingest
   ```

4. **Stop the services**

   ```bash
   docker-compose down
   ```

   To remove volumes as well:
   ```bash
   docker-compose down -v
   ```

## Accessing the Database

### pgAdmin Web Interface

- **URL**: http://localhost:8085
- **Email**: admin@admin.com (or your PGADMIN_EMAIL)
- **Password**: admin (or your PGADMIN_PASSWORD)

### Command Line (pgcli)

```bash
pgcli -h localhost -U root -d ny_taxi
```

### Python Connection

```python
from sqlalchemy import create_engine

engine = create_engine('postgresql://root:root@localhost:5432/ny_taxi')
```

## Local Development

### Setup

Install dependencies using uv:

```bash
cd pipeline/
uv sync
```

### Run Locally

**Main script:**
```bash
uv run python main.py
```

**Data ingestion script:**
```bash
uv run python ingest_data.py \
  --pg-user <username> \
  --pg-pass <password> \
  --pg-host localhost \
  --pg-port 5432 \
  --pg-db <db name> \
  --target-table yellow_taxi_trips
```

**Jupyter Notebook:**
```bash
uv run jupyter notebook
```

Open `notebook.ipynb` to explore the data and test pipeline components.

## Key Dependencies

- `pandas` - Data manipulation
- `psycopg2-binary` - PostgreSQL adapter
- `pyarrow` - Arrow data format support
- `sqlalchemy` - SQL toolkit and ORM
- `tqdm` - Progress bars

## Docker Commands Reference

### Build and run manually

```bash
# Build the image
docker build -t taxi-ingest:v001 .

# Run the container
docker run -it \
  --network=pipeline_default \
  taxi-ingest:v001 \
  --pg-user=<username> \
  --pg-pass=<passowrd> \
  --pg-host=pgdatabase \
  --pg-port=5432 \
  --pg-db=<db_name> \
  --target-table=yellow_taxi_trips
```

### Useful commands

```bash
# View running containers
docker ps

# View all containers
docker ps -a

# View logs
docker logs <container-name>

# Execute command in container
docker exec -it <container-name> bash

# Clean up
docker system prune -a
```

## Troubleshooting

### Port already in use

```bash
# Change ports in docker-compose.yaml
ports:
  - "5433:5432"  # Use 5433 instead of 5432
  - "8086:80"    # Use 8086 instead of 8085
```

### Container exits immediately

```bash
# Check logs
docker-compose logs taxi_ingest

# Run interactively for debugging
docker run -it taxi-ingest:v001 bash
```

### Database connection refused

```bash
# Ensure database is ready
docker-compose ps

# Check network connectivity
docker network ls
docker network inspect pipeline_default
```

### Reset everything

```bash
# Stop and remove all containers, networks, and volumes
docker-compose down -v

# Prune system
docker system prune -a --volumes
```

## Architecture

```
┌─────────────────┐
│  taxi_ingest    │
│  (Python App)   │
└────────┬────────┘
         │
         │ Ingests data
         ↓
┌─────────────────┐
│   PostgreSQL    │
│   (Database)    │
└────────┬────────┘
         │
         │ Admin interface
         ↓
┌─────────────────┐
│    pgAdmin      │
│  (Web UI:8085)  │
└─────────────────┘
```

## Learning Objectives

This workshop demonstrates:
- Writing Dockerfiles for Python applications
- Multi-container orchestration with Docker Compose
- Database containerization and persistence
- Environment variable management
- Container networking
- Volume management for data persistence
- Service dependencies and health checks
