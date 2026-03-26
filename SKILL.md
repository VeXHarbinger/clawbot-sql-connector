---
name: sql-connector
version: 2.1.0-alpha
status: alpha
description: "Generic SQL Server connectivity for OpenClaw agents. Use when: (1) executing parameterized queries against SQL Server, (2) building repository layers that need a sealed, retry-capable SQL transport, (3) any agent that needs reliable MSSQL access without subprocess/sqlcmd. Provides execute/query/scalar/ping APIs via pymssql with automatic retry, connection pooling, and structured error handling. ALPHA: use at your own risk, API may change."
---

# SQL Connector Skill
> Generic SQL Server connectivity for OpenClaw agents — pymssql transport

## Overview

A sealed, retry-capable SQL Server connector for OpenClaw agents. Built on **pymssql** — no `sqlcmd`, no ODBC drivers, no `mssql-tools` required.

- `execute()` — INSERT/UPDATE/DELETE → returns bool
- `query()` — SELECT → returns list of dicts
- `scalar()` — single value (COUNT, MAX, INSERTED.id, etc.)
- `ping()` — connectivity check
- Automatic retry with exponential backoff (3 attempts)
- Credentials loaded from `.env` only — nothing hardcoded
- `execute()` and `query()` are **sealed** via metaclass — subclasses cannot override them

## Installation

```bash
clawhub install sql-connector
```

## Quick Start

```python
from sql_connector import get_connector

db = get_connector('cloud')   # or 'local'

# INSERT / UPDATE / DELETE
ok = db.execute(
    "INSERT INTO memory.Logs (category, msg) VALUES (%s, %s)",
    ("info", "hello world")
)

# SELECT → list of dicts
rows = db.query(
    "SELECT id, content FROM memory.Memories WHERE category = %s",
    ("facts",)
)

# Single value
count = db.scalar("SELECT COUNT(*) FROM memory.TaskQueue WHERE status = %s", ("pending",))

# Connectivity check
if db.ping():
    print("connected")
```

## .env Setup

```env
# Local SQL Server
SQL_local_server=10.0.0.110
SQL_local_port=1433
SQL_local_database=YourDatabase
SQL_local_user=your_user
SQL_local_password=your_password

# Cloud SQL Server (Azure / site4now / etc.)
SQL_cloud_server=yourserver.database.windows.net
SQL_cloud_port=1433
SQL_cloud_database=your_cloud_db
SQL_cloud_user=your_cloud_user
SQL_cloud_password=your_cloud_password

# Add new backends with the same pattern:
# SQL_<backend>_server, SQL_<backend>_database, SQL_<backend>_user, SQL_<backend>_password
```

Then connect:

```python
db = get_connector('local')    # Uses SQL_local_* vars
db = get_connector('cloud')    # Uses SQL_cloud_* vars
db = get_connector('staging')  # Uses SQL_staging_* vars
```

**To add a new backend:** add 4 env vars following the pattern — no code changes needed.

## Architecture

```
SQLConnector (ABC, _LockCoreMethods metaclass)
  ├── execute() / query() / scalar() / ping()  ← SEALED
  └── MSSQLConnector (pymssql, TDS 7.4)
        └── get_connector(backend) factory
```

Extend by subclassing `MSSQLConnector` to add domain-specific repository methods.
See [clawbot-sql-memory](https://github.com/VeXHarbinger/clawbot-sql-memory) for an example.

## Requirements

```bash
pip install pymssql python-dotenv
```

## Related

- [clawbot-sql-memory](https://github.com/VeXHarbinger/clawbot-sql-memory) — Semantic memory layer built on this connector

## License

MIT
