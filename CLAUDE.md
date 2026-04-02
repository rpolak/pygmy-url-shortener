# CLAUDE.md

## Project Overview

Pygmy is a Python URL shortener with a dual-server architecture:
- **Flask REST API** (`pygmy/`) - core shortening engine and API (port 9119)
- **Django Web UI** (`pygmyui/`) - frontend that communicates with the API via HTTP (port 8000)

## Quick Start

```bash
# Install dependencies
pip install -r requirements.txt

# Run both servers (API + UI)
python run.py

# Run API only
python pygmy_api_run.py
```

## Project Structure

- `pygmy/app/` - Business logic (link shortening, auth)
- `pygmy/model/` - SQLAlchemy models (Link, User, ClickMeta)
- `pygmy/rest/` - Flask REST API views and routing
- `pygmy/database/` - DB abstraction (SQLite, PostgreSQL, MySQL)
- `pygmy/config/` - Config files (`pygmy.cfg` is git-ignored, `pygmy_test.cfg` for tests)
- `pygmy/validator/` - Marshmallow schemas
- `pygmy/helpers/` - Short code generation (base62)
- `pygmyui/` - Django frontend app
- `pygmyui/restclient/` - HTTP client that talks to the Flask API
- `tests/` - Integration tests
- `pygmy/tests/` - Unit tests

## Running Tests

```bash
# Install pytest if needed
pip install pytest

# Run all tests from root directory
py.test

# Run with coverage
pip install coverage
coverage run --omit="*/templates*,*/venv*,*/tests*" -m py.test
coverage report
```

Tests start live API (port 9118) and UI (port 8001) servers via fixtures in `tests/fixture.py`. The test config is `pygmy/config/pygmy_test.cfg` using SQLite.

## Configuration

- Main config: `pygmy/config/pygmy.cfg` (git-ignored, create from `pygmy_test.cfg` as template)
- Django settings: `pygmyui/pygmyui/settings.py`
- DB engine is set in config: `sqlite3`, `postgresql`, or `mysql`
- DB credentials can be overridden via env vars: `DB_HOST`, `DB_PORT`, `DB_USER`, `DB_PASSWORD`, `DB_NAME`

## Key Patterns

- **Database sessions**: Use `@dbconnection` decorator from `pygmy/database/dbutil.py` — it injects a `db` session parameter into model manager methods
- **Model managers**: Each model (Link, User, ClickMeta) has a companion Manager class in the same file for DB operations
- **Auth**: JWT-based via Flask-JWT-Extended. Access tokens are short-lived; refresh tokens are long-lived
- **URL resolution**: `/<code>` routes through `resolve()` in `pygmy/rest/shorturl.py`
- **Short codes**: Generated via base62 encoding in `pygmy/core/hashdigest.py`

## Common Pitfalls

- The UI server must be able to reach the API server — they communicate over HTTP
- `pygmy.cfg` must exist for non-test runs (copy from `pygmy_test.cfg`)
- Integration tests spawn subprocesses; ensure ports 9118 and 8001 are free
- SQLite is the default DB; PostgreSQL/MySQL require additional pip packages (`pymysql` for MySQL)
