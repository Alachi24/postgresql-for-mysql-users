# MySQL to PostgreSQL cheat sheet

The PostgreSQL column was tested on PostgreSQL 18.6. The MySQL column is standard MySQL syntax, and the behavior claims in this guide are checked against the official documentation (see [Sources](#sources)). MySQL was not installed for this guide, so those claims are documented, not tested.

| Task | MySQL | PostgreSQL (`psql`) |
| --- | --- | --- |
| Connect as admin | `mysql -u root -p` | `sudo -u postgres psql` |
| Connect to a database | `mysql -u user -p dbname` | `psql -d dbname` |
| List databases | `SHOW DATABASES;` | `\l` |
| Switch database | `USE dbname;` | `\c dbname` |
| List tables | `SHOW TABLES;` | `\dt` |
| Describe a table | `DESCRIBE notes;` | `\d notes` |
| List users or roles | `SELECT user, host FROM mysql.user;` | `\du` |
| Auto-numbered ID | `AUTO_INCREMENT` | `GENERATED ... AS IDENTITY` |
| Quit | `exit` | `\q` |

## Concepts that map differently

| Concept | MySQL | PostgreSQL |
| --- | --- | --- |
| Account | `'user'@'host'` | Role (host rules live in `pg_hba.conf`) |
| Schema | Same as a database | A namespace inside a database (`public` by default) |
| Server config | `/etc/mysql/my.cnf` | `postgresql.conf` plus `pg_hba.conf` under `/etc/postgresql/<version>/<cluster>/` |
| Data directory | `/var/lib/mysql` | `/var/lib/postgresql/<version>/<cluster>` |
| Local passwordless login | `auth_socket` plugin for `root` | Peer authentication via `pg_hba.conf` |

## Sources

| Claim | Source |
| --- | --- |
| `TIMESTAMP` covers `'1970-01-01 00:00:01'` UTC to `'2038-01-19 03:14:07'` UTC. Use `DATETIME` for later dates. | [MySQL 8.4 Reference Manual: Date and Time Data Types](https://dev.mysql.com/doc/refman/8.4/en/datetime.html) |
| `AUTO_INCREMENT` accepts a manual value and continues the sequence from the largest value. | [MySQL 8.4: Using AUTO_INCREMENT](https://dev.mysql.com/doc/refman/8.4/en/example-auto-increment.html) |
| On Unix, `-h localhost` connects through the socket file, not TCP. | [MySQL 8.4: Connection Transport Protocols](https://dev.mysql.com/doc/refman/8.4/en/transport-protocols.html) |
| On Ubuntu, `sudo mysql` needs no password because `root` uses `auth_socket`. | [Ubuntu Server docs: Install and configure MySQL](https://ubuntu.com/server/docs/how-to/databases/install-mysql/) |
| `auth_socket` matches the operating system user name to the MySQL user name. | [MySQL 8.4: Socket Peer-Credential Pluggable Authentication](https://dev.mysql.com/doc/refman/8.4/en/socket-pluggable-authentication.html) |
| Indexes on `BLOB` and `TEXT` columns require a prefix length. | [MySQL 8.4: Column Indexes](https://dev.mysql.com/doc/refman/8.4/en/column-indexes.html) |

Back to the [overview](../README.md)
