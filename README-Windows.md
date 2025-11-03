# OpenProject on Windows (PowerShell)

This short guide shows how to run the docker-compose setup from this repository on Windows using PowerShell, with persistent storage.

Two persistence approaches:

- Recommended: use Docker named volumes (default in `docker-compose.yml`). No host permissions changes are required.
- Alternative: use host bind mounts (useful when you want files directly on disk). This can cause permission issues with Postgres on Windows; prefer this only if you understand host permissions or run Docker Desktop with WSL2.

## Quick (recommended) — named volumes

1. Ensure you have Docker Desktop (or Docker Engine/Compose) installed and running.
2. Optional: create an `.env` from the example and adjust values:

```powershell
cp .env.example .env
# Edit .env with an editor like notepad or code.exe as needed
code .env
```

3. Start the stack (recommended to disable https on first run):

```powershell
$env:OPENPROJECT_HTTPS = "false"; docker compose up -d --build --pull always
```

This will create named volumes (by default `pgdata` and `opdata`) that persist your PostgreSQL database and OpenProject assets across container restarts and upgrades.

## Alternative — host bind mounts (example)

If you prefer to keep data in repository subfolders, create directories and use the example override file `docker-compose.override.yml.example` (next to `docker-compose.yml`).

1. Create local folders:

```powershell
New-Item -ItemType Directory -Force -Path .\pgdata
New-Item -ItemType Directory -Force -Path .\opdata
```

2. Create an `.env` (or set env vars) to use those paths, e.g.:

```text
PGDATA=./pgdata
OPDATA=./opdata
```

3. Start the stack:

```powershell
$env:OPENPROJECT_HTTPS = "false"; docker compose up -d --build --pull always
```

Notes and caveats:

- Postgres running with a Windows bind mount may run into permission problems because the container expects Unix permissions. If you see startup errors from the `db` service, switch to named volumes or run Docker under WSL2 where Linux permissions are available.
- For production, prefer mounting only the `opdata` to a host path and keep the DB in a named volume, or run a managed DB outside the container.

## Backups (control plane)

To run the built-in backup helper (will write archives to `./backups` when using host bind mounts, or into the container file system when using named volumes — ensure you mount `./backups` if you want them on disk):

```powershell
docker compose -f docker-compose.yml -f docker-compose.control.yml build
docker compose -f docker-compose.yml -f docker-compose.control.yml run --rm backup
```

This creates gzipped archives for the Postgres data and OpenProject assets inside `./backups` when using the example host bind mounts.

## Stopping and removing containers

Stop the stack:

```powershell
docker compose down
```

Remove named volumes if you want to wipe persisted data (only do this if you really want data removed):

```powershell
docker volume ls
docker volume rm <volume_name>
```

Replace `<volume_name>` with the volumes created by Docker Compose (for example, `openproject_pgdata` or similar — run `docker volume ls` to list them).

---

If you want, I can also add a `docker-compose.override.yml` file into this repository that maps host directories by default (or create a `.env.example.windows`). Tell me which you prefer and I will add it.
