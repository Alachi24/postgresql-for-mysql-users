# Find the config files and read the access rules

> **Your username.** Examples show the author's Linux username, `chisa`. Run `whoami` to see yours. Wherever you see `<your-username>`, type your own name. Wherever you see `chisa` in an output, you will see yours.

## Where things live

Run these as the superuser, because the settings are visible only to one:

```bash
sudo -u postgres psql -c "SHOW config_file;"
sudo -u postgres psql -c "SHOW hba_file;"
sudo -u postgres psql -c "SHOW data_directory;"
```

```text
               config_file
-----------------------------------------
 /etc/postgresql/18/main/postgresql.conf

              hba_file
-------------------------------------
 /etc/postgresql/18/main/pg_hba.conf

       data_directory
-----------------------------
 /var/lib/postgresql/18/main
```

| File or folder | Purpose | Readable by |
| --- | --- | --- |
| `postgresql.conf` | Server settings: port, memory, logging, `listen_addresses` | Everyone (`-rw-r--r--`) |
| `pg_hba.conf` | **Who** may connect, **from where**, and **how** they authenticate | `postgres` only (`-rw-r-----`) |
| Data directory | The database files | `postgres` only (`drwx------`) |

Active settings in `postgresql.conf` on this install:

```text
data_directory = '/var/lib/postgresql/18/main'
hba_file = '/etc/postgresql/18/main/pg_hba.conf'
port = 5432
max_connections = 100
```

> **Path layout:** `/etc/postgresql/<version>/<cluster>/`. Ubuntu can run several versions and clusters side by side (`pg_lsclusters`), so the version appears in every path. MySQL uses a single `/etc/mysql/my.cnf` (plus `conf.d/`) and keeps data in `/var/lib/mysql`.

**Applying changes.** After editing `pg_hba.conf`, run `sudo systemctl reload postgresql`. No restart is needed. Some `postgresql.conf` settings, such as `port` and `max_connections`, are marked `(change requires restart)`.

## Who can connect, and how: `pg_hba.conf`

Print the active rules without comments or blank lines:

```bash
sudo grep -vE '^\s*(#|$)' /etc/postgresql/18/main/pg_hba.conf
```

```text
local   all             postgres                                peer
local   all             all                                     peer
host    all             all             127.0.0.1/32            scram-sha-256
host    all             all             ::1/128                 scram-sha-256
local   replication     all                                     peer
host    replication     all             127.0.0.1/32            scram-sha-256
host    replication     all             ::1/128                 scram-sha-256
```

The columns are TYPE, DATABASE, USER, ADDRESS and METHOD. PostgreSQL checks the rules **top to bottom** and uses the **first match**.

| Rule | Meaning |
| --- | --- |
| `local all postgres peer` | Over the Unix socket, `postgres` is allowed only if the Linux user is `postgres`. This is why you use `sudo -u postgres psql`. |
| `local all all peer` | Over the Unix socket, any role is allowed if its name matches the Linux user. This lets your role in without a password. |
| `host all all 127.0.0.1/32 scram-sha-256` | Over TCP from this machine (IPv4), a password is required, checked with SCRAM-SHA-256. |
| `host all all ::1/128 scram-sha-256` | The same, for IPv6 localhost. |
| `... replication ...` | The same pattern for replication connections. |

There are no rules for other machines, so remote connections are refused. The server also listens only on `127.0.0.1`.

## Same role, same database, different connection type

| Command | Connection | Rule matched | Result |
| --- | --- | --- | --- |
| `psql -w -d practice` | Unix socket (`local`) | `local all all peer` | Connected. `Password Used: false` |
| `psql -w -h 127.0.0.1 -d practice` | TCP (`host`) | `host all all 127.0.0.1/32 scram-sha-256` | Refused: password required |

```text
psql: error: connection to server at "127.0.0.1", port 5432 failed: fe_sendauth: no password supplied
```

The password you set when creating the role is used whenever you connect over TCP. That covers applications, ORMs, GUI tools such as pgAdmin and DBeaver, and connection strings like `postgresql://<your-username>:<your-password>@localhost:5432/practice`.

## What differs from MySQL

- **Authentication depends on how you connect.** The same role needs no password over the local socket but needs one over TCP. MySQL's closest equivalent is the `auth_socket` plugin, which checks that the operating system user name matches the MySQL user name. On Ubuntu, `sudo mysql` works without a password because Ubuntu's MySQL packaging uses `auth_socket` for `root`. A plain MySQL install from Oracle uses a password instead.
- **Access rules live in a file.** MySQL stores them as `'user'@'host'` rows and changes them with SQL (`CREATE USER`, `GRANT`). In PostgreSQL, `pg_hba.conf` decides who may connect, and SQL `GRANT` decides what they can do afterwards.
- **`localhost` is not the same in both.** In MySQL, `-h localhost` uses the **socket**. In PostgreSQL, `-h localhost` uses **TCP** and needs a password. To use the socket, leave out `-h`.

Next: [MySQL to PostgreSQL cheat sheet](05-mysql-cheatsheet.md)
