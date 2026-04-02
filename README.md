<p align="center"><img src="pygmyui/static/logo/logov2.png" alt="pygmy" height="200px"></p>

<div align="center">
  <h1>Pygmy</h1>

[![Coverage Status](https://img.shields.io/coveralls/github/amitt001/pygmy.svg?color=yellowgreen)](https://coveralls.io/github/amitt001/pygmy?branch=master)
![PyPI - Python Version](https://img.shields.io/pypi/pyversions/Django.svg)
[![PyPI license](https://img.shields.io/pypi/l/ansicolortags.svg)](https://github.com/amitt001/pygmy/blob/master/LICENSE)
![Docker Pulls](https://img.shields.io/docker/pulls/amit19/pygmy.svg)

Demo Version: [https://demo.pygy.co](https://demo.pygy.co)

Link stats (add **+** to the URL) example: [demo.pygy.co/pygmy+](https://demo.pygy.co/pygmy+)

</div>

Pygmy is an open-source, extensible & easy-to-use URL shortener. It's built with a loosely coupled architecture that makes it easy to self-host and customize.

## Architecture

The project has three major parts:

| Component | Framework | Description |
|-----------|-----------|-------------|
| **Core API** (`pygmy/`) | Flask | URL shortening engine and REST API |
| **Web UI** (`pygmyui/`) | Django | User-facing web interface |
| **Database** | SQLAlchemy | Supports SQLite, PostgreSQL, and MySQL |

The UI communicates with the API over HTTP — they run as separate processes.

## Features

- Shorten URLs with auto-generated or custom short codes (e.g. `pygy.co/pygmy`)
- Auto-expiring URLs
- Secret key protected URLs
- User accounts with dashboard to manage links
- Link analytics: click counts, referrer tracking, country stats (via GeoIP2)
- Append `+` to any short URL to view its stats

## Quick Start

### Docker (recommended)

```bash
docker pull amit19/pygmy
docker run -it -p 8000:8000 amit19/pygmy
```

Open http://localhost:8000 in your browser.

### Docker Compose

For a production-like setup with PostgreSQL:

```bash
docker-compose up
```

This starts three services: PostgreSQL, the Flask API, and the Django UI. DB credentials are configured in `docker-compose.yml`.

### From Source

```bash
git clone https://github.com/amitt001/pygmy.git && cd pygmy

# (Recommended) Create a virtualenv
python3 -m venv env
source env/bin/activate

# Install dependencies
pip install -r requirements.txt

# Start both API and UI servers
python run.py
```

- UI available at http://127.0.0.1:8000
- API available at http://127.0.0.1:9119
- Logs at `pygmy/data/pygmy.log`

> **Note:** Python 3 is required. SQLite is the default database — no extra setup needed.

## Configuration

There are two config files:

| File | Purpose |
|------|---------|
| `pygmy/config/pygmy.cfg` | API and core settings (DB, auth, logging) |
| `pygmyui/pygmyui/settings.py` | Django UI settings |

The `pygmy.cfg` file is git-ignored. Copy `pygmy/config/pygmy_test.cfg` as a starting template. Database credentials can also be set via environment variables: `DB_HOST`, `DB_PORT`, `DB_USER`, `DB_PASSWORD`, `DB_NAME`.

### Database Setup

**SQLite** (default — no setup required):
```ini
[database]
engine: sqlite3
sqlite_data_dir: data
sqlite_db_file_name: pygmy.db
```

**PostgreSQL:**
```ini
[database]
engine: postgresql
user: root
password: root
host: 127.0.0.1
port: 5432
db_name: pygmy
```

**MySQL** (requires `pip install pymysql`):
```ini
[database]
engine: mysql
user: root
password: root
host: 127.0.0.1
port: 3306
db_name: pygmy
```

For MySQL, create the database first: `CREATE DATABASE pygmy;`. MySQL version > 5.6.5 is recommended.

## API Usage

The REST API runs on port 9119 by default.

**Create a user:**
```bash
curl -XPOST http://127.0.0.1:9119/api/user/1 \
  -H 'Content-Type: application/json' \
  -d '{"email": "user@example.com", "f_name": "Jane", "l_name": "Doe", "password": "secure_password"}'
```

**Shorten a URL:**
```bash
curl -XPOST http://127.0.0.1:9119/api/shorten \
  -H 'Content-Type: application/json' \
  -d '{"long_url": "https://example.com"}'
```

**Expand a short URL:**
```bash
curl http://127.0.0.1:9119/api/unshorten?short_url=http://127.0.0.1:9119/AbCdE
```

### API Endpoints

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/api/shorten` | POST | Create a shortened URL |
| `/api/unshorten` | GET | Expand a short URL |
| `/api/user` | POST | Create a new user |
| `/api/login` | POST | Login (returns JWT tokens) |
| `/api/user/<id>/links` | GET | List user's links |
| `/<code>` | GET | Redirect to the long URL |
| `/<code>+` | GET | View link statistics |

### Authentication

Pygmy uses JWT for API authentication. On login, two tokens are returned:
- **Access token** — short-lived, used for API requests
- **Refresh token** — long-lived, used to obtain new access tokens via `/token/refresh`

User passwords are hashed with [bcrypt](https://en.wikipedia.org/wiki/Bcrypt).

## Development

### Running Tests

```bash
pip install pytest
py.test
```

Integration tests automatically start API (port 9118) and UI (port 8001) test servers.

**With coverage:**
```bash
pip install coverage
coverage run --omit="*/templates*,*/venv*,*/tests*" -m py.test
coverage report
```

### Contributing

1. Clone the repo and create a branch for your changes
2. Make your changes and add tests if applicable
3. Run `py.test` to verify everything passes
4. Submit a pull request

If you find a bug or have a feature request, please [open an issue](https://github.com/amitt001/pygmy/issues).

## Tech Stack

- **Languages:** Python 3, JavaScript, HTML, CSS
- **API:** Flask, Flask-JWT-Extended, Flask-CORS
- **UI:** Django, jQuery
- **ORM:** SQLAlchemy
- **Validation:** Marshmallow
- **Server:** Gunicorn
- **Analytics:** GeoIP2 / MaxMindDB
- **Containerization:** Docker, Docker Compose

## Sponsorship

The demo version of this website is made possible due to the generous sponsorship of DigitalOcean

<img src="https://github.com/amitt001/pygmy/assets/7390944/e8df3143-d9a5-4582-8e73-6f1c1822d046" width="200" />

## License

MIT License — Copyright (c) 2022 Amit Tripathi

[Read full license](https://github.com/amitt001/pygmy/blob/master/LICENSE)
