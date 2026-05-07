# Deployment runbook

Procedures for initial setup, verifying health, upgrading and rolling back, and Boost-specific configuration pointers. For architecture and environment tables, see [deployment-overview.md](deployment-overview.md).

## Prerequisites

- **Docker** and **Docker Compose v2** on the host.
- **Directory layout:** Git repo **`weblate-docker`** (this tree) must live as **`weblate-docker/`** under the root of the **`weblate`** application checkout so [`docker-compose.yml`](../docker-compose.yml) can use `build.context: ..` and `dockerfile: weblate-docker/Dockerfile`. The parent folder’s name is arbitrary (for example **`boost-weblate`** locally or **`/opt/boost-weblate`** on the deployment server). See [deployment-overview.md](deployment-overview.md#repository-layout) and the **`weblate`** repo’s **`.github/workflows/cd.yml`** for an automated deploy sequence. Initialize the submodule or reproduce the same tree manually.
- **PostgreSQL** reachable from the Weblate container. The sample [`environment.example`](../environment.example) uses `POSTGRES_HOST=host.docker.internal` for a database on the host; adjust for your network (same Docker network as a `postgres` service, cloud RDS, etc.).
- **Resources:** Follow [upstream production guidance](https://docs.weblate.org/en/latest/admin/install.html) for sizing; migrations may take several minutes on first start (`HEALTHCHECK` start period is five minutes in [`Dockerfile`](../Dockerfile)).

## Initial setup

1. From the **`weblate`** application root (example path **`boost-weblate/`**), ensure **`weblate-docker/`** is populated (submodule or clone).
2. **Recommended** when mirroring production CI: from that same application root, copy the Docker ignore file into the build context (see **`weblate`** `.github/workflows/cd.yml`):

   ```bash
   cp weblate-docker/.dockerignore .dockerignore
   ```

3. Change into this directory:

   ```bash
   cd weblate-docker
   ```

4. Create local env file:

   ```bash
   cp environment.example environment
   ```

5. Edit **`environment`** (do not commit). At minimum set PostgreSQL variables (`POSTGRES_*`), `WEBLATE_ADMIN_PASSWORD` (or `WEBLATE_ADMIN_PASSWORD_FILE`), and secrets required for your deployment. See [Weblate generic Docker settings](https://docs.weblate.org/en/latest/admin/install/docker.html#generic-settings).

6. Build and start:

   ```bash
   docker compose build
   docker compose up -d
   ```

7. **Port mapping:** Default Compose publishes **8000** on the host → **8080** in the container. Browse `http://localhost:8000/` (or your host) and sign in with the admin user configured via `WEBLATE_ADMIN_EMAIL` / `WEBLATE_ADMIN_PASSWORD`.

8. Optional sanity check inside the stack:

   ```bash
   docker compose exec weblate weblate check --deploy
   ```

   Resolve reported issues before relying on the instance in production.

## Production deploy (matches `weblate` CI/CD)

If you use the **`workflow_dispatch`** CD workflow in the **`weblate`** repo (`.github/workflows/cd.yml`), the remote script effectively runs:

```bash
cd /opt/boost-weblate
git pull origin develop
git submodule update --init weblate-docker
cp weblate-docker/.dockerignore .dockerignore
cd weblate-docker
docker compose down
docker compose up -d --build
```

After the containers restart, that workflow waits five minutes then verifies **`http://localhost:8000/healthz/`** from the host (same port mapping as [Initial setup](#initial-setup)). Align branch names (`develop`) and paths with your fork if they differ.

## Health checks

### Container health (Docker `HEALTHCHECK`)

The image defines a `HEALTHCHECK` that runs [`health_check`](../health_check). It:

1. **HTTP:** If the web stack is enabled, requests **`/healthz/`** — either `http://localhost:8080/healthz/` or `https://localhost:4443/healthz/` when TLS certs exist under `/app/data/ssl/`.
2. **Supervisor:** Runs `supervisorctl status`. Exit code **3** is treated as success (expected when one-shot services differ); other failures fail the check.
3. **Processes:** Fails if any supervised program is not in the expected running/stopped state (see script comments for `check`).

Inspect status:

```bash
docker compose ps
docker inspect --format='{{.State.Health.Status}}' <weblate_container_id>
```

### External monitoring

Probe the same path your users reach, for example:

- `http://<host>:8000/healthz/` with default port mapping, or
- `https://<your-domain>/healthz/` behind a reverse proxy.

Use timeouts consistent with your SLA; the script uses `curl --max-time 30` for the internal check.

## Upgrades

1. **Read** [Weblate upgrade notes](https://docs.weblate.org/en/latest/admin/upgrade.html) and your release’s changelog.
2. **Back up the database** (and optional file backup for `/app/data`) per [Weblate backup](https://docs.weblate.org/en/latest/admin/backup.html).
3. Check out updated **`weblate`** and **`weblate-docker`** revisions (matching tags or branches your team uses).
4. Rebuild and recreate:

   ```bash
   cd weblate-docker
   docker compose build --no-cache   # optional, when you need a clean dependency layer
   docker compose up -d
   ```

   The entrypoint runs migrations as part of startup; allow time for the health check to pass after deploy.

5. Verify **`/healthz/`**, application login, and background tasks (Celery) if you use them heavily.

## Rollback

Weblate upgrades apply **forward-only Django migrations**. Rolling back **only** the container to an older image while the database has already migrated forward can fail or corrupt data.

**Safer rollback pattern**

1. **Redeploy the previous image** you used before the failed release (tag or digest recorded in your environment table).
2. If **database migrations ran** during the failed deploy, **restore PostgreSQL from the backup taken immediately before that upgrade**, then start the old image against that restored database.

If you did not take a backup before upgrading, treat rollback as **high risk**; prefer fixing forward or restoring from the last known-good backup.

Operational steps:

```bash
cd weblate-docker
# Example: use a previously tagged image (adjust tag to your registry workflow)
docker compose pull weblate   # if using a registry tag
docker compose up -d weblate
```

Confirm `docker compose ps` and `/healthz/` before declaring the rollback complete.

## Boost-specific configuration

Fork-only environment variables (OpenRouter batch translation, Boost endpoint timing, etc.) are documented in the Boost Weblate additions chapter in the **`weblate`** repo: **`docs/admin/boost-weblate.rst`**. Cross-check [`environment.example`](../environment.example) for variables such as `OPENROUTER_API_KEY`, `OPENROUTER_MODEL`, `AUTO_BATCH_TRANSLATE_VIA_OPENROUTER`, and `BOOST_ENDPOINT_ADD_TRANSLATION_SECONDS`.

## See also

- [deployment-overview.md](deployment-overview.md)
- [Weblate: Docker](https://docs.weblate.org/en/latest/admin/install/docker.html)
- [Weblate: Backup](https://docs.weblate.org/en/latest/admin/backup.html)
- [Weblate: Upgrade](https://docs.weblate.org/en/latest/admin/upgrade.html)
