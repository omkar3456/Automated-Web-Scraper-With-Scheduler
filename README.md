# Automated Web Scraper with Scheduler 🚀

A production-ready web scraping platform built with Flask, Celery, Redis, Selenium, BeautifulSoup, and SQLite. The platform supports automated scheduling, dynamic and static web scraping, database storage, and CSV export through a user-friendly dashboard.

---

## Features

| Feature | Details |
|---|---|
| **Two scraper engines** | `requests` + BeautifulSoup for static HTML; Selenium + Chrome for JS-heavy pages |
| **Celery scheduler** | Async task queue + beat scheduler for cron-based runs |
| **SQLite storage** | All jobs, runs, and scraped records persisted automatically |
| **CSV export** | One-click export per job or per run via the dashboard or API |
| **Flask web UI** | Dashboard, job management, records viewer |
| **REST API** | Full CRUD for jobs; trigger runs; export data |
| **Logging** | Colour console + rotating file logs |
| **Error handling** | Auto-retry with exponential back-off |
| **Tests** | Pytest suite with mocked HTTP calls |

---

## Screenshots

### Dashboard
![Dashboard](data/screenshots/demo/dashboard.png)

### Job Management
![Jobs](data/screenshots/demo/jobs.png)

### CSV Export
![Export](data/screenshots/demo/export.png)

---

## Project Structure

```
web_scraper/
├── app/
│   ├── __init__.py          # Flask app factory
│   ├── api/
│   │   └── routes.py        # REST API blueprints
│   ├── database/
│   │   ├── models.py        # SQLAlchemy models
│   │   └── db_manager.py    # CRUD + CSV export helpers
│   ├── scrapers/
│   │   ├── base_scraper.py  # Abstract base with retry logic
│   │   ├── requests_scraper.py
│   │   └── selenium_scraper.py
│   ├── scheduler/
│   │   ├── celery_app.py    # Celery factory
│   │   └── tasks.py         # Celery tasks
│   ├── ui/
│   │   └── views.py         # Server-rendered Flask views
│   └── utils/
│       ├── logger.py        # Logging setup
│       └── helpers.py       # Shared utilities
├── config/
│   ├── settings.py          # Environment-based config
│   ├── sample_targets.json  # Ready-to-use scraper configs
│   └── celery_beat_schedule.py
├── static/
│   ├── css/style.css
│   └── js/app.js
├── templates/               # Jinja2 HTML templates
├── tests/                   # Pytest test suite
├── data/
│   ├── exports/             # Generated CSV files
│   └── screenshots/         # Selenium screenshots
├── logs/                    # Rotating log files
├── run.py                   # App entry point
├── seed.py                  # Seed sample jobs
├── Makefile                 # Developer shortcuts
├── requirements.txt
├── .env.example
└── README.md
```

---

## Quick Start

### 1. Clone & create virtual environment

```bash
git clone <repo-url>
cd web_scraper
python -m venv venv
source venv/bin/activate   # Windows: venv\Scripts\activate
```

### 2. Install dependencies

```bash
pip install -r requirements.txt
```

### 3. Configure environment

```bash
cp .env.example .env
# Edit .env — at minimum set SECRET_KEY
```

### 4. Start Redis (required for Celery)

```bash
# Docker (easiest)
docker run -d -p 6379:6379 redis:7-alpine

# Or install locally: https://redis.io/download
```

### 5. Seed sample jobs (optional)

```bash
python seed.py
```

### 6. Run everything

**Terminal 1 — Flask web app:**
```bash
python run.py
```

**Terminal 2 — Celery worker:**
```bash
celery -A app.scheduler.celery_app worker --loglevel=info
```

**Terminal 3 — Celery beat scheduler (optional):**
```bash
celery -A app.scheduler.celery_app beat --loglevel=info
```

Open **http://localhost:5000** in your browser.

Or use the Makefile:
```bash
make install
make seed
make run          # Flask
make worker       # Celery worker (separate terminal)
make beat         # Beat scheduler (separate terminal)
```

---

## REST API

| Method | Endpoint | Description |
|--------|----------|-------------|
| `GET` | `/api/jobs` | List all jobs |
| `POST` | `/api/jobs` | Create a job |
| `GET` | `/api/jobs/:id` | Get a job |
| `PUT` | `/api/jobs/:id` | Update a job |
| `DELETE` | `/api/jobs/:id` | Delete a job |
| `POST` | `/api/jobs/:id/run` | Trigger a run |
| `GET` | `/api/jobs/:id/runs` | List runs for a job |
| `GET` | `/api/records` | List records (`?job_id=&run_id=&limit=`) |
| `GET` | `/api/export` | Download CSV (`?job_id=&run_id=`) |
| `GET` | `/api/health` | Health check |

### Create a job — example

```bash
curl -X POST http://localhost:5000/api/jobs \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Books to Scrape",
    "url": "https://books.toscrape.com/catalogue/page-1.html",
    "scraper_type": "requests",
    "schedule_cron": "0 8 * * *",
    "config_json": "{\"list_selector\": \"article.product_pod\", \"selectors\": {\"title\": \"h3 > a\", \"price\": \"p.price_color\"}, \"pagination\": {\"next_selector\": \"li.next > a\", \"max_pages\": 3}}"
  }'
```

### Trigger a run

```bash
curl -X POST http://localhost:5000/api/jobs/1/run
```

### Export CSV

```bash
curl "http://localhost:5000/api/export?job_id=1" -o results.csv
```

---

## Selector Config (config_json)

```json
{
  "list_selector": "article.product_pod",
  "selectors": {
    "title":       "h3 > a",
    "url":         "h3 > a",
    "price":       "p.price_color",
    "rating":      "p.star-rating",
    "description": "p.description"
  },
  "pagination": {
    "next_selector": "li.next > a",
    "max_pages": 5
  },
  "delay": 1.5,
  "max_retries": 3
}
```

**Selenium extras:**

```json
{
  "wait_selector":    ".product-grid",
  "scroll_to_bottom": true,
  "screenshot":       true,
  "actions": [
    {"type": "click",  "selector": "#accept-cookies"},
    {"type": "wait",   "seconds": 1},
    {"type": "input",  "selector": "#search", "value": "laptop"}
  ]
}
```

---

## Running Tests

```bash
pytest tests/ -v
# With coverage:
pytest tests/ -v --cov=app --cov-report=term-missing
```

---

## Environment Variables

| Variable | Default | Description |
|---|---|---|
| `SECRET_KEY` | `dev-secret-key` | Flask secret key |
| `FLASK_ENV` | `development` | `development` / `production` |
| `DATABASE_URL` | `sqlite:///data/scraper.db` | Database URI |
| `REDIS_URL` | `redis://localhost:6379/0` | Redis connection |
| `HEADLESS` | `True` | Run Chrome headless |
| `DEFAULT_DELAY` | `2` | Seconds between requests |
| `MAX_RETRIES` | `3` | Retry attempts per scrape |
| `LOG_LEVEL` | `INFO` | Logging verbosity |

---


## Skills Demonstrated

- Python Programming
- Data Collection & Web Scraping
- REST API Development
- Database Management (SQLite)
- Background Task Processing (Celery)
- Browser Automation (Selenium)
- Data Export & Reporting
- Error Handling & Retry Logic
- Docker Containerization
- Software Architecture Design

---


## Technology Stack

- **Python 3.10+**
- **Flask** — web framework & REST API
- **BeautifulSoup4 + lxml** — HTML parsing
- **Selenium + webdriver-manager** — browser automation
- **Celery + Redis** — task queue & scheduler
- **SQLAlchemy + SQLite** — ORM & database
- **Pandas** — data export utilities
- **pytest** — test suite

---

## Future Enhancements

- PostgreSQL/MySQL Support
- User Authentication & Authorization
- Email Notifications
- Proxy Rotation
- Cloud Deployment (AWS/Azure)
- Dashboard Analytics
- Data Cleaning Pipelines
- AI-Powered Data Extraction

---

## License

MIT
