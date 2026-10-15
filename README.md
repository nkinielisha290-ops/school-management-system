# School Management System

Django + PostgreSQL school management system, packaged with Docker and ready to deploy to Render or Railway.

## Stack

- Python 3.14, Django 6.1
- PostgreSQL 17 (SQLite fallback for quick local work)
- Gunicorn + WhiteNoise for production
- GitHub Actions CI (tests, deploy checks, Docker build)

## Local development

### Option A: Docker (recommended, uses Postgres)

```bash
cp .env.example .env
docker compose up --build
docker compose exec web python manage.py createsuperuser
```

App: http://localhost:8000 · Admin: http://localhost:8000/admin/ · Health: http://localhost:8000/healthz/

### Option B: Virtualenv (uses SQLite unless `DATABASE_URL` is set)

```powershell
python -m venv .venv
.\.venv\Scripts\activate
pip install -r requirements.txt
copy .env.example .env
python manage.py migrate
python manage.py createsuperuser
python manage.py runserver
```

### Tests

```bash
python manage.py test
```

## Environment variables

| Variable | Purpose | Default |
| --- | --- | --- |
| `DJANGO_SECRET_KEY` | Secret key (required in production) | insecure dev key |
| `DJANGO_DEBUG` | Debug mode | `False` |
| `DJANGO_ALLOWED_HOSTS` | Comma-separated hostnames | `localhost,127.0.0.1` |
| `DJANGO_CSRF_TRUSTED_ORIGINS` | Comma-separated origins, e.g. `https://school.example.com` | empty |
| `DATABASE_URL` | Postgres URL, e.g. `postgres://user:pass@host:5432/db` | SQLite |
| `DJANGO_SECURE_HSTS_SECONDS` | HSTS max-age in production | `0` |
| `WEB_CONCURRENCY` | Gunicorn workers | `3` |

## Deployment

The container runs migrations on start (`docker-entrypoint.sh`) and serves the app with Gunicorn on `$PORT`.

### Render

1. In Render, choose **New → Blueprint** and select this repository.
2. Render reads `render.yaml`, creates the Postgres database and the web service, and generates `DJANGO_SECRET_KEY`.
3. To use a custom domain, add it to `DJANGO_ALLOWED_HOSTS` and `DJANGO_CSRF_TRUSTED_ORIGINS`.

### Railway

1. Create a project from this GitHub repo; Railway builds from `Dockerfile` (see `railway.json`).
2. Add a PostgreSQL plugin and set `DATABASE_URL` to `${{Postgres.DATABASE_URL}}`.
3. Set `DJANGO_SECRET_KEY`, `DJANGO_ALLOWED_HOSTS` (your `*.up.railway.app` domain) and `DJANGO_CSRF_TRUSTED_ORIGINS` (`https://<that domain>`).

## Project layout

```
config/          Django project settings, URLs, WSGI/ASGI
core/            Core app (health check)
Dockerfile       Production image
docker-compose.yml  Local dev: web + Postgres
render.yaml      Render Blueprint
railway.json     Railway config
.github/workflows/ci.yml  CI pipeline
```
