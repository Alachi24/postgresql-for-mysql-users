# PostgreSQL on Ubuntu (WSL2) for MySQL Users

A hands-on guide to installing PostgreSQL on Ubuntu under WSL2 and getting oriented if you already know MySQL. Every command and output in this guide was captured from a real install, and each page points out where PostgreSQL behaves differently from MySQL.

## Who this is for

You have used MySQL (or MariaDB) and want to start using PostgreSQL on a Linux command line. You do not need any PostgreSQL experience.

## Tested on

| | |
| --- | --- |
| PostgreSQL | 18.6 (Ubuntu package `18.6-0ubuntu0.26.04.1`) |
| Operating system | Ubuntu 26.04.1 LTS ("resolute") |
| Environment | WSL2 (kernel 6.18.58.1-microsoft-standard-WSL2) |
| Tested | November 2026 |

## Prerequisites

- Windows 10 or 11 with WSL2 and an Ubuntu distribution installed (`wsl --install` from an administrator PowerShell, then restart)
- A Linux user with `sudo` access
- Basic SQL knowledge

## Before you start: find your username

This guide creates a PostgreSQL role with the **same name as your Linux username**. Find yours:

```bash
whoami
```

The examples show the author's username, `chisa`. Wherever a command contains `<your-username>`, replace it, brackets included, with the name `whoami` printed. Wherever an output shows `chisa`, yours will show your own name.

## What you will do

1. [Install PostgreSQL and verify it is running](docs/01-install-and-verify.md)
2. [Create a role and database, and understand local login](docs/02-roles-and-authentication.md)
3. [Run your first SQL and see what differs from MySQL](docs/03-first-sql.md)
4. [Find the config files and read the access rules](docs/04-config-and-access-rules.md)
5. [MySQL to PostgreSQL command cheat sheet](docs/05-mysql-cheatsheet.md)
6. [Troubleshooting](docs/troubleshooting.md)

## Conventions

- Commands run in the Ubuntu (WSL) terminal unless stated.
- A `postgres=#` or `practice=>` prompt means you are inside `psql`. Lines starting with `\` are `psql` shortcuts, not SQL, and take no semicolon.
- Passwords in examples are placeholders. Use your own.

## Author

Chisaneme Aloni, technical writer. [Portfolio](https://alachi24-portfolio.netlify.app/) · [GitHub](https://github.com/Alachi24)
