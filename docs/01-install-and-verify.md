# Install PostgreSQL and verify it is running

## Install

In the Ubuntu terminal:

```bash
sudo apt update
sudo apt install postgresql
```

On PostgreSQL 18, the contrib modules ship in the main package, so you do not need to name `postgresql-contrib` separately.

## Check the client version

```bash
psql --version
```

```text
psql (PostgreSQL) 18.6 (Ubuntu 18.6-0ubuntu0.26.04.1)
```

Note the exact version, Paths and behavior depend on it.

## Check the server is running

PostgreSQL on Ubuntu is organized into **clusters**. Check yours with:

```bash
pg_lsclusters
```

```text
Ver Cluster Port Status Owner    Data directory              Log file
18  main    5432 online postgres /var/lib/postgresql/18/main /var/log/postgresql/postgresql-18-main.log
```

You want one row with your version, the cluster name `main`, port `5432`, and status **online**. If the status is `down`, start it and check again:

```bash
sudo pg_ctlcluster 18 main start
```

## Confirm it accepts connections

```bash
sudo -u postgres psql
```

```text
psql (18.6 (Ubuntu 18.6-0ubuntu0.26.04.1))
Type "help" for help.

postgres=#
```

Reaching the `postgres=#` prompt as the `postgres` superuser means the server is accepting connections. Type `\q` to leave.

## Check the server version

The `psql` banner shows the **client** version. To ask the **server**, run:

```sql
SELECT version();
```

```text
 PostgreSQL 18.6 (Ubuntu 18.6-0ubuntu0.26.04.1) on x86_64-pc-linux-gnu, compiled by gcc (Ubuntu 15.2.0-16ubuntu1) 15.2.0, 64-bit
(1 row)
```

It also reports the platform, the compiler, and that this is a 64-bit build. Any role can run it. The MySQL equivalent is `SELECT VERSION();`.

## Why `service postgresql status` shows `active (exited)`

```bash
sudo service postgresql status
```

```text
● postgresql.service - PostgreSQL RDBMS
     Loaded: loaded (/usr/lib/systemd/system/postgresql.service; enabled; preset: enabled)
     Active: active (exited) since Mon 2026-11-16 01:44:04 WAT
    Process: 403 ExecStart=/bin/true (code=exited, status=0/SUCCESS)
```

This is **normal, not a failure**. `postgresql.service` is only a wrapper whose `ExecStart=/bin/true` runs and exits immediately. The real server runs as a separate unit for each cluster:

```bash
systemctl status postgresql@18-main --no-pager
```

```text
● postgresql@18-main.service - PostgreSQL Cluster 18-main
     Active: active (running) since Mon 2026-11-16 01:44:04 WAT
   Main PID: 299 (postgres)
```

To confirm the server is really running, use either `pg_lsclusters` (status `online`) or the `postgresql@18-main` unit (`active (running)`).

If the output stops with `(END)` and a column of `~`, you are in a pager. Press `q` to leave it, or add `--no-pager` to the command.

## Startup behavior in WSL

The service is `enabled`, so PostgreSQL starts automatically when WSL starts, and it shuts down cleanly when WSL stops.

Next: [Create a role and database](02-roles-and-authentication.md)
