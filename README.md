<a href="https://weblate.org/"><img alt="Weblate" src="https://s.weblate.org/cdn/Logo-Darktext-borders.png" height="80px" /></a>

# Boost-Weblate Docker image

This repository packages **Boost-Weblate**—Weblate plus the `boost-weblate` Python distribution—for deployment as **`boost-weblate:latest`**. It is derived from the [WeblateOrg/docker](https://github.com/WeblateOrg/docker) layout (multi-stage `weblate/dev` + `weblate/base`, nginx, supervisor, Celery).

Upstream Weblate remains libre software; see [weblate.org](https://weblate.org/) and the [official Docker install docs][doc].

## Repository layout (required for builds)

The `Dockerfile` expects a **parent directory** as the Docker build context:

| Path (relative to parent) | Role |
|---------------------------|------|
| `weblate-docker/` | This repo (Dockerfile, `requirements.txt`, `etc/`, `start`, …) |
| Parent root | **`boost-weblate`** sources (`COPY . /app/boost-weblate/` installs `/app/boost-weblate[extras]`) |

`docker-compose.yml` is written for that layout: build **`context: ..`**, **`dockerfile: weblate-docker/Dockerfile`**. Clone or arrange your monorepo so the docker tree lives at **`weblate-docker/`** next to the application root that pip installs as `boost-weblate`.

## Quick start

1. Place this tree at **`../weblate-docker`** relative to your **`boost-weblate`** repo root (see above).
2. Copy and edit environment variables:

   ```bash
   cp environment.example environment
   ```

   Set at least **`WEBLATE_SITE_DOMAIN`**, **`POSTGRES_*`**, and any admin/email/Git credentials you need. PostgreSQL is expected **outside** the compose file (for example on the host); **`POSTGRES_HOST`** defaults to `host.docker.internal` in `environment.example`.

3. From **`weblate-docker/`**, build and run:

   ```bash
   docker compose up --build -d
   ```

4. Open the UI at **http://localhost:8000** (compose maps **8000 → 8080**; nginx listens on **8080** inside the container).

## Fork-specific settings

See **`environment.example`** for Boost-oriented options, including:

- **`AUTO_BATCH_TRANSLATE_VIA_OPENROUTER`** — optional batch translation via OpenRouter.
- **`BOOST_ENDPOINT_ADD_TRANSLATION_SECONDS`** — wait time before treating a component/translation as ready when adding languages.

Generic Weblate Docker variables (debug, hosts, mail, LDAP, Git hosting tokens, and more) are documented in the [Weblate Docker documentation][doc].

## Image details

- **Image name (compose):** `boost-weblate:latest`
- **Weblate version:** pinned via **`WEBLATE_VERSION`** in the `Dockerfile` (see that file for the current value).
- **Patches:** any `*.patch` files under `weblate-docker/patches` are applied to the installed Weblate packages at build time.

[doc]: https://docs.weblate.org/en/latest/admin/install/docker.html
