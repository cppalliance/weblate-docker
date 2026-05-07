<a href="https://weblate.org/"><img alt="Weblate" src="https://s.weblate.org/cdn/Logo-Darktext-borders.png" height="80px" /></a>

**Weblate is libre software web-based continuous localization system,
used by over 2500 libre projects and companies in more than 165 countries.**

# Official Docker container for Weblate

[![Website](https://img.shields.io/badge/website-weblate.org-blue.svg)](https://weblate.org/)
[![Translation status](https://hosted.weblate.org/widgets/weblate/-/svg-badge.svg)](https://hosted.weblate.org/engage/weblate/?utm_source=widget)
[![CII Best Practices](https://bestpractices.coreinfrastructure.org/projects/552/badge)](https://bestpractices.coreinfrastructure.org/projects/552)
[![Documentation](https://readthedocs.org/projects/weblate/badge/)][doc]

## Running Weblate

- [Weblate docker-compose](https://github.com/WeblateOrg/docker-compose)
- [OpenShift](https://docs.weblate.org/en/latest/admin/install/openshift.html)
- [Helm chart for Weblate](https://hub.helm.sh/charts/weblate/weblate)

## Exposed ports

The webserver is running on the port 8080.

## Operations / deployment

**Repositories:** **`weblate`** (application fork) and **`weblate-docker`** (this repo). Your checkout folder name may differ—many installs clone **`weblate`** into a directory called **`boost-weblate`** (the Docker build context), with **`weblate-docker/`** as a submodule beside it.

[`docker-compose.yml`](docker-compose.yml) uses `context: ..` and `dockerfile: weblate-docker/Dockerfile`. Production CD (see **`weblate`** `.github/workflows/cd.yml`) also copies [`weblate-docker/.dockerignore`](.dockerignore) to the application root before `docker compose up --build`.

- [Deployment overview](docs/deployment-overview.md) — scope, components, environments, ops handoff
- [Deployment runbook](docs/deployment-runbook.md) — setup, health checks, upgrade and rollback, CI/CD sequence

## Documentation

Detailed documentation is available in [Weblate documentation][doc].

[doc]: https://docs.weblate.org/en/latest/admin/install/docker.html
