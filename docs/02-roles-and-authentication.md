# Create a role and database, and understand local login

> **Your username.** Examples show the author's Linux username, `chisa`. Run `whoami` to see yours. Wherever you see `<your-username>`, type your own name. Wherever you see `chisa` in an output, you will see yours.

## The problem: no role that matches your Linux user

By default, `psql` connects as a PostgreSQL role with the **same name as your Linux user**. A fresh install has only the `postgres` role, so this fails:

```bash
psql -d practice
```

```text
psql: error: connection to server on socket "/var/run/postgresql/.s.PGSQL.5432" failed: FATAL:  role "chisa" does not exist
```

The error is about the role, not the database.

> **MySQL difference:** MySQL logs in as a named user (`mysql -u root -p`). PostgreSQL derives the role from your operating system user unless you say otherwise.

## Create a role and a database

Connect as the superuser, then run:

```bash
sudo -u postgres psql
```

```sql
CREATE ROLE <your-username> WITH LOGIN PASSWORD 'choose-a-password';
CREATE DATABASE practice OWNER <your-username>;
\du
\l
\q
```

```text
CREATE ROLE
CREATE DATABASE
                             List of roles
 Role name |                         Attributes
-----------+------------------------------------------------------------
 chisa     |
 postgres  | Superuser, Create role, Create DB, Replication, Bypass RLS
```

- Name the role after your Linux user so local login works without a password (see below).
- `WITH LOGIN` lets the role connect. `CREATE ROLE` alone does not.
- Replace `choose-a-password` with a password of your own, and keep the single quotes around it.
- Your role (`chisa` in this output) has no special attributes. It is an ordinary role that owns only the `practice` database.

`\l` shows the databases:

```text
   Name    |  Owner   | Encoding | Locale Provider | Collate |  Ctype  | ...
-----------+----------+----------+-----------------+---------+---------+
 postgres  | postgres | UTF8     | libc            | C.UTF-8 | C.UTF-8 |
 practice  | chisa    | UTF8     | libc            | C.UTF-8 | C.UTF-8 |
 template0 | postgres | UTF8     | libc            | C.UTF-8 | C.UTF-8 |
 template1 | postgres | UTF8     | libc            | C.UTF-8 | C.UTF-8 |
```

> **Template databases:** `CREATE DATABASE` copies `template1` by default. `template0` is a pristine copy you use when you need a different encoding or locale. MySQL has no equivalent. Its system schemas are `mysql`, `information_schema`, `performance_schema` and `sys`.

### Setting the password safely

Typing the password inside the command shows it on screen, and `psql` saves what you type to a history file. That is fine for a practice database. For anything real, set it privately instead:

1. Create the role without a password:

   ```sql
   CREATE ROLE <your-username> WITH LOGIN;
   ```

2. Set the password at a private prompt:

   ```sql
   \password <your-username>
   ```

PostgreSQL asks you to type the password twice. Nothing appears on screen as you type, and the password is not saved in the history file.

## Connect as yourself

```bash
psql -w -d practice -c "SELECT current_user, current_database();"
```

`-w` means never prompt for a password, so the command succeeds only if none is needed.

```text
 current_user | current_database
--------------+------------------
 chisa        | practice
```

Inside `psql`, `\conninfo` confirms it:

```text
 Client User          | chisa
 Socket Directory     | /var/run/postgresql
 Password Used        | false
 Superuser            | off
```

You connected with no password. This is **peer authentication**: on a local Unix socket, PostgreSQL asks the operating system which Linux user is connecting and trusts that name if a role with the same name exists.

The password you set is not used here. It applies to TCP connections, which is how applications, ORMs and GUI tools connect. See [who can connect, and how](04-config-and-access-rules.md).

## When peer authentication refuses you

Peer authentication requires the role name to match your Linux username. Asking for a different role over the socket fails:

```bash
psql -U postgres -d postgres
```

```text
psql: error: connection to server on socket "/var/run/postgresql/.s.PGSQL.5432" failed: FATAL:  Peer authentication failed for user "postgres"
```

A role that does not exist gives the **same** message:

```bash
psql -U alice -d practice
```

```text
psql: error: connection to server on socket "/var/run/postgresql/.s.PGSQL.5432" failed: FATAL:  Peer authentication failed for user "alice"
```

Peer authentication fails before PostgreSQL checks whether the role exists, so the message does not reveal which roles exist. Compare this with the first error on this page: with no `-U`, the names match, peer authentication passes, and PostgreSQL then reports `role "chisa" does not exist`.

To act as `postgres`, use `sudo -u postgres psql`, or connect as your own role.

## Roles, not users

PostgreSQL has **roles**. `CREATE USER` is shorthand for `CREATE ROLE ... WITH LOGIN`. In MySQL an account is `'user'@'host'` and the host is part of its identity. In PostgreSQL, where a role may connect from is configured separately in `pg_hba.conf`.

Next: [Run your first SQL](03-first-sql.md)
