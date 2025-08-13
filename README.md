# Metabase on Fly.io

This repository contains the configuration to deploy [Metabase](https://www.metabase.com/) to [Fly.io](https://fly.io/).  
Metabase is an open-source business intelligence tool for running queries, building dashboards, and visualizing your data.


## Prerequisites

- A [Fly.io](https://fly.io/) account and CLI installed (`brew install flyctl` on macOS)
- A running PostgreSQL database (can be Fly Postgres or external)
- Docker image: [`metabase/metabase:latest`](https://hub.docker.com/r/metabase/metabase)
- Fly.io app created:
  ```bash
  fly apps create my-metabase
  ```

## Configuration

### Environment Variables

We configure the following in `fly.toml` and Fly secrets.

#### Database Connection

Metabase requires a PostgreSQL connection URI.
You must set this as a **Fly secret**:

```bash
fly secrets set MB_DB_CONNECTION_URI="postgres://USER:PASSWORD@HOST:5432/DBNAME"
```

* Replace `USER`, `PASSWORD`, `HOST`, `DBNAME` with your actual Postgres details.
* If using Fly Postgres, you can retrieve your connection string with:

  ```bash
  fly postgres connect --app <your-db-app> --url
  ```

#### Custom Encryption Key

Metabase uses an application encryption key to secure sensitive information (like saved database credentials).
You **must** set this before first startup, or it will auto-generate one (not recommended in ephemeral environments):

```bash
fly secrets set MB_ENCRYPTION_SECRET_KEY="your-32-character-random-key"
```

Generate a strong 32-character key:

```bash
openssl rand -hex 16
```

## Deploying

1. Update `fly.toml` with your app name, memory, and CPU configuration.
2. Set the secrets as described above.
3. Deploy:

   ```bash
   fly deploy
   ```

## Scaling Memory / CPU

Metabase is a Java app and benefits from more RAM.

To set in `fly.toml`:

```toml
[vm]
cpu_kind = "shared"
cpus = 1
memory_mb = 2048
```

Or resize an existing machine:

```bash
fly machines update <MACHINE_ID> --memory 2048 --cpus 1 --cpu-kind shared
```

## Health Checks

The Fly config includes a health check on `/api/health`:

```toml
[[http_service.checks]]
grace_period = "180s"
interval = "30s"
method = "GET"
timeout = "5s"
path = "/api/health"
```

## Accessing Metabase

Once deployed, Metabase will be available at:

```
https://<your-app>.fly.dev
```

First-time setup will prompt you to create an admin account.

## Useful Commands

* **View logs**:

  ```bash
  fly logs
  ```
* **Open a shell**:

  ```bash
  fly ssh console
  ```
* **Restart app**:

  ```bash
  fly apps restart
  ```

## Notes

* Always keep your `MB_ENCRYPTION_SECRET_KEY` safe; losing it will prevent access to stored database credentials.
* Back up your Postgres database regularly; all Metabase configurations are stored there.
* Do not commit secrets into your repository—always use Fly secrets.
