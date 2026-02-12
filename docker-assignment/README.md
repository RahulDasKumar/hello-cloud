# Docker Assignment #1: Two-Container Stack

## Overview
This project implements a simple two-container Docker stack consisting of:
- **PostgreSQL database**: Stores trip data (city, minutes, fare)
- **Python application**: Connects to the database, queries data, computes statistics, and outputs results

The Python app calculates total trips, average fare by city, and identifies the top N longest trips by duration.

## Prerequisites
- Docker Desktop installed and running
- Docker Compose installed
- Git (for version control)

## Project Structure
```
docker-assignment/
├── app/
│   ├── Dockerfile        # Python app container definition
│   └── main.py          # Application entrypoint
├── db/
│   ├── Dockerfile       # Database container definition
│   └── init.sql         # Database schema + seed data
├── out/                 # Output directory (created at runtime)
│   └── summary.json     # Query results
├── compose.yml          # Docker Compose configuration
├── Makefile            # Build/run commands
└── README.md           # This file
```

## Commands to Build and Run

### Using Make (Recommended)
```bash
# Build and run the entire stack
make all

# Or just run (rebuilds if needed)
make up

# Stop and clean up
make clean
```

### Using Docker Compose Directly
```bash
# Build and run
docker compose up --build

# Stop containers
docker compose down -v

# Clean output directory
rm -rf out && mkdir -p out
```

## Example Output

### Console Output (stdout)
```json
=== Summary ===
{
  "total_trips": 6,
  "avg_fare_by_city": [
    {
      "city": "Charlotte",
      "avg_fare": 16.25
    },
    {
      "city": "New York",
      "avg_fare": 19.0
    },
    {
      "city": "San Francisco",
      "avg_fare": 20.25
    }
  ],
  "top_by_minutes": [
    {
      "city": "San Francisco",
      "minutes": 28,
      "fare": 29.3
    },
    {
      "city": "New York",
      "minutes": 26,
      "fare": 27.1
    },
    {
      "city": "Charlotte",
      "minutes": 21,
      "fare": 20.0
    },
    {
      "city": "Charlotte",
      "minutes": 12,
      "fare": 12.5
    },
    {
      "city": "San Francisco",
      "minutes": 11,
      "fare": 11.2
    }
  ]
}
```

### File Output
The same JSON data is written to `out/summary.json`

## How It Works

1. **Database Initialization**: When the `db` container starts, PostgreSQL automatically executes `init.sql` which creates the `trips` table and inserts sample data.

2. **Health Check**: The compose file ensures the database is fully ready before starting the app container using a health check.

3. **App Execution**: The Python app connects to the database, runs three queries:
   - Count total trips
   - Calculate average fare grouped by city
   - Find top N trips ordered by duration (minutes)

4. **Output**: Results are both printed to stdout and saved to `/out/summary.json`

## Troubleshooting

### Database Not Ready
If you see "Waiting for database..." messages, the app is waiting for PostgreSQL to be ready. This is normal and handled automatically with retries.

### Permission Issues on out/
If you encounter permission errors:
```bash
# Linux/Mac
sudo chown -R $USER:$USER out/

# Or recreate the directory
rm -rf out && mkdir -p out
```

### Port 5432 Already in Use
If port 5432 is already in use, either:
- Stop the conflicting service
- Or modify `compose.yml` to use a different port:
  ```yaml
  db:
    ports:
      - "5433:5432"  # Map to different host port
  ```

### Container Won't Start
```bash
# View logs
docker compose logs

# View specific service logs
docker compose logs db
docker compose logs app

# Rebuild from scratch
docker compose down -v
docker compose build --no-cache
docker compose up
```

## Environment Variables

You can customize the app behavior by modifying the `compose.yml` file:

- `APP_TOP_N`: Number of top trips to retrieve (default: 10)
- `DB_USER`: Database username (default: appuser)
- `DB_PASS`: Database password (default: secretpw)
- `DB_NAME`: Database name (default: appdb)

## Testing with Different Data

To test with different data, modify `db/init.sql` and rebuild:
```bash
make clean
make up
```
