<p align="center">
  <picture>
    <source media="(prefers-color-scheme: dark)" srcset="assets/icon-dark.svg">
    <img src="assets/icon.svg" alt="Firefly III logo" width="128">
  </picture>
</p>

<h1 align="center">aac-firefly-iii</h1>

<p align="center">Self-hosted deployment definition for Firefly III, a free and open source personal finance manager.</p>

<p align="center">
  <a href="https://github.com/firefly-iii/firefly-iii"><img alt="Upstream" src="https://img.shields.io/badge/upstream-firefly--iii%2Ffirefly--iii-blue?logo=github"></a>
  <img alt="Image" src="https://img.shields.io/badge/image-fireflyiii%2Fcore%3Aversion--5.5.13-2496ED?logo=docker&logoColor=white">
  <a href="https://github.com/firefly-iii/firefly-iii/blob/main/LICENSE"><img alt="License" src="https://img.shields.io/badge/license-AGPL--3.0-green"></a>
</p>

## About

Firefly III is a self-hosted personal finance manager. It keeps track of expenses, income,
transfers, budgets, savings goals and recurring transactions across any number of accounts,
so you can see where your money goes without handing your data to a third party.

Key features (as provided by upstream):

- Accounts, transactions, categories, tags and budgets
- Recurring transactions and bills, savings goals and piggy banks
- Reports and charts over custom periods
- Multi-currency support and rule-based transaction handling
- REST API and webhooks; a separate Data Importer for CSV and other bank exports

This deployment runs the Firefly III core application together with the Data Importer,
a MariaDB database, a Redis cache and an Alpine-based cron container that triggers the
scheduled jobs through the static cron endpoint.

## Upstream

- Project: <https://github.com/firefly-iii/firefly-iii>
- Documentation: <https://docs.firefly-iii.org>
- Container image (core): `fireflyiii/core:version-5.5.13`
- Container image (importer): `fireflyiii/data-importer:version-0.9.12`
- Dependencies: `mariadb:12.3.1-ubi10-rc` (deployed as `aac-firefly-db`), `redis:8.6.1` (deployed as `aac-firefly-redis`), `alpine:3.24.3` (cron, deployed as `aac-firefly-cron`)
- License: AGPL-3.0 (upstream project)

## Quick start (Docker Compose)

The Compose file is rendered from `service.yml` by the App-as-Code template engine. For a
manual run, the external networks used for routing must exist first:

```bash
docker network create services-secured
docker network create services-exposed

# Provide the values listed under "Configuration" in .env / stack.env
docker compose up -d
```

Secrets are not stored in this repository. Provide them through the deployment pipeline
or create the `.env` and `stack.env` files yourself (see below). Do not commit them.

## Configuration

Environment variables (defaults from `service.yml`; `CHANGE_ME` must be set per deployment):

| Variable | Default | Description |
|---|---|---|
| `APP_KEY` | `CHANGE_ME` (secret) | Laravel application key (32 characters) |
| `APP_URL` | `CHANGE_ME` | Public URL of the Firefly III instance, e.g. `https://finance.example.com` |
| `APP_ENV` | `local` | Application environment |
| `APP_DEBUG` | `false` | Debug mode; keep disabled outside development |
| `APP_LOG_LEVEL` | `notice` | Application log level |
| `LOG_LEVEL` | `debug` | Log level of the core container |
| `LOG_CHANNEL` | `stack` | Logging channel |
| `SITE_OWNER` | `CHANGE_ME` | E-mail address of the site owner |
| `ALLOW_WEBHOOKS` | `true` | Allow Firefly III webhooks |
| `DEFAULT_LANGUAGE` | `de_DE` | Default UI language |
| `DEFAULT_LOCALE` | `equal` | Default number and date locale |
| `TZ` | `Europe/Berlin` | Time zone |
| `PUID` / `PGID` | `1000` | User and group ID the containers run as |
| `TRUSTED_PROXIES` | `**` | Proxies whose forwarded headers are trusted |
| `VANITY_URL` | `CHANGE_ME` | Public URL used by the Data Importer to reach Firefly III |
| `FIREFLY_III_URL` | `CHANGE_ME` | Internal URL of the core service as seen by the Data Importer |
| `FIREFLY_III_CLIENT_ID` | `CHANGE_ME` | OAuth client ID used by the Data Importer |
| `VERIFY_TLS_SECURITY` | `false` | Verify TLS certificates for outgoing requests |
| `STATIC_CRON_TOKEN` | `CHANGE_ME` (secret) | Token for the static cron endpoint, triggered daily at 03:00 |
| `DB_CONNECTION` | `mysql` | Database driver |
| `DB_HOST` | `CHANGE_ME` | Database host, e.g. the `aac-firefly-db` dependency |
| `DB_PORT` | `3306` | Database port |
| `DB_DATABASE` | `firefly` | Database name |
| `DB_USERNAME` | `firefly` | Database user |
| `DB_PASSWORD` | `CHANGE_ME` (secret) | Database password |
| `MARIADB_DATABASE` / `MARIADB_USER` | `firefly` | Database and user created by the MariaDB container |
| `MARIADB_PASSWORD` | `CHANGE_ME` (secret) | Password of the MariaDB user |
| `MARIADB_ROOT_PASSWORD` | `CHANGE_ME` (secret) | Root password of the MariaDB container |
| `REDIS_HOST` | `CHANGE_ME` | Redis host, e.g. the `aac-firefly-redis` dependency |
| `REDIS_PORT` | `6379` | Redis port |
| `CACHE_DRIVER` / `SESSION_DRIVER` | `redis` | Cache and session storage |
| `MAIL_MAILER` | `log` | Mail transport; `log` writes mails to the log only |
| `MAIL_HOST` | `CHANGE_ME` | SMTP host |
| `MAIL_PORT` | `465` | SMTP port |
| `MAIL_ENCRYPTION` | `SSL` | SMTP encryption |
| `MAIL_USERNAME` | `CHANGE_ME` | SMTP user |
| `MAIL_PASSWORD` | `CHANGE_ME` (secret) | SMTP password |
| `MAIL_FROM` | `CHANGE_ME` | Sender address |

Ports:

| Port | Protocol | Purpose |
|---|---|---|
| `8080` | TCP | Firefly III web interface (`web`, published as `28765`) |
| `8080` | TCP | Second published entry (`importer`, published as `28766`), mapped to the core container |

Volumes:

| Mount | Description |
|---|---|
| `configs` -> `/var/www/html/storage/upload` | Configuration and upload storage of Firefly III |
| `importer_upload` -> `/var/www/html/storage/upload` | Upload storage of the Data Importer |
| `db_data` -> `/var/lib/mysql` | MariaDB data (database dependency) |
| `redis_data` -> `/data` | Redis data (cache dependency) |

## Deploy with App-as-Code

1. Edit `service.yml`. It is the single source of truth for image, ports, environment,
   volumes, dependencies and per-stage overrides. Do not edit the rendered output.
2. Pushing to `main` (or a configured stage branch) runs the service pipeline, which renders
   the manifests with `aac-template-engine` and triggers the deployment.
3. Releases of this definition are published under <https://github.com/app-as-code/aac-firefly-iii>.

## Credits and license

Firefly III is developed by the Firefly III project and licensed under AGPL-3.0. The Data
Importer is a separate upstream project (`firefly-iii/data-importer`, also AGPL-3.0). This
repository only contains the deployment definition for them and is not a fork of either
application. The logo is taken from the selfh.st icon collection; see
[`assets/ICON_SOURCE.md`](assets/ICON_SOURCE.md) for icon sources and licenses.
