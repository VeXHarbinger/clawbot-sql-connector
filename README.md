# clawbot-sql-connector

> ⚠️ **ALPHA — Use at your own risk.** API is functional and tested but may change. We're actively using this in production and will stabilize the API after 30 days of community feedback. Please open issues for anything that breaks.

A sealed, retry-capable SQL Server connector for OpenClaw agents. Built on **pymssql** — no `sqlcmd` or `mssql-tools` required.

## Features

- `get_connector('cloud')` / `get_connector('local')` factory
- Abstract base (`SQLConnector`) with `_LockCoreMethods` metaclass — `execute()` and `query()` cannot be overridden in subclasses, keeping all queries parameterized
- Automatic retry with exponential backoff on transient failures
- `execute()` — INSERT/UPDATE/DELETE, returns bool
- `query()` — SELECT, returns list of dicts
- `scalar()` — single value (e.g. `INSERTED.id`)
- `ping()` — connectivity check
- Environment-based credentials via `.env` — nothing hardcoded

## Requirements

```bash
pip install pymssql python-dotenv
```

> **Note:** `pymssql` bundles its own TDS driver. No `sqlcmd`, no ODBC drivers, no `mssql-tools` needed.

## Installation

```bash
clawhub install sql-connector
```

## .env Setup

```env
# Local instance
SQL_SERVER=10.0.0.110
SQL_PORT=1433
SQL_DATABASE=YourDatabase
SQL_USER=your_user
SQL_PASSWORD=your_password

# Cloud instance (Azure / site4now / etc.)
SQL_CLOUD_SERVER=yourserver.database.windows.net
SQL_CLOUD_PORT=1433
SQL_CLOUD_DATABASE=your_cloud_db
SQL_CLOUD_USER=your_cloud_user
SQL_CLOUD_PASSWORD=your_cloud_password
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

## Architecture

```
SQLConnector (ABC, _LockCoreMethods metaclass)
  ├── execute() / query() / scalar()   ← SEALED — parameterized only, no overrides
  ├── ping()
  ├── _connect()                        ← abstract
  └── MSSQLConnector (pymssql)          ← concrete implementation
```

Extend by subclassing `MSSQLConnector` to add domain-specific repository methods. See [clawbot-sql-memory](https://github.com/VeXHarbinger/clawbot-sql-memory) for an example.

## Security Note

All queries are parameterized. The metaclass seals `execute()` and `query()` so subclasses cannot bypass parameterization. Never pass user input via f-strings or string concatenation into SQL — the connector will not prevent it at the call site if you build your query string before passing it in.

## Related

- [clawbot-sql-memory](https://github.com/VeXHarbinger/clawbot-sql-memory) — Semantic memory layer built on this connector
- [oblio-heart-and-soul](https://github.com/VeXHarbinger/oblio-heart-and-soul) — Full agent system reference implementation

## Community

Found a bug? Have an improvement? Open an issue — this is alpha and community feedback shapes the v1 API.

## License

MIT
