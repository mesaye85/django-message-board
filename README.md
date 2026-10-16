# Message board (mb-app)

A one-page Django message board: posts are added through the admin and listed on the home page.

## Context

I built this in October 2020 while learning Django, before I knew the framework's conventions for settings, static files and deployment. I keep it public as a record of how I learned. The same code, with the same history, is in my `django_112` repository.

## Stack

- Python 3.8.2 (from the virtualenv that was committed)
- Django 3.1.2, asgiref 3.2.10, pytz 2020.1, sqlparse 0.4.1
- gunicorn 19.9.0 (used by the Heroku `Procfile`)
- SQLite

## Running it

Use Python 3.8, the version in the original virtualenv; Django 3.1.2 isn't supported on newer versions. I checked these steps with Python 3.8.

    python3.8 -m venv venv
    source venv/bin/activate
    pip install -r requirements.txt
    export DJANGO_SECRET_KEY='any-long-random-string'   # optional locally; see .env.example
    python manage.py migrate          # creates db.sqlite3, which is no longer tracked
    python manage.py createsuperuser
    python manage.py runserver

Add posts at `/admin/` and they appear at `/`. `python manage.py test` runs four tests, and they pass.

## Status

It runs locally. I set it up for Heroku (`Procfile`, gunicorn, `.herokuapp.com` in `ALLOWED_HOSTS`). As committed, it likely would not have built there, and it would not have kept its data (see below).

## Known gaps

- `config/settings.py:24`: the secret key was hardcoded. It now comes from `DJANGO_SECRET_KEY`. The original key is still in git history; it was a development key.
- `config/settings.py:27`: `DEBUG` was hardcoded to `True`. It now reads `DJANGO_DEBUG` and still defaults to `True`.
- `config/settings.py:122`: only `STATIC_URL` is set. Without `STATIC_ROOT`, `collectstatic` fails with `ImproperlyConfigured`, and Heroku's Python build runs that step.
- `config/settings.py:78-83`: SQLite is the only database. Heroku's filesystem is ephemeral, so posts would be lost whenever the dyno restarted.
- The requirements file was committed as `requirments.txt`. Heroku's Python buildpack looks for `requirements.txt`, so it would not have found the dependencies. I renamed it in this cleanup without changing the pins.
- No `runtime.txt`, so the Python version on Heroku was not pinned.
- `manage.py check --deploy` reports five warnings: HSTS, SSL redirect, secure session and CSRF cookies, and `DEBUG`.
- Posts can only be created in the admin. There is no form and no pagination.
- `posts/tests.py`: the test class name has a typo (`PostModedTest`), and the model test uses `assertAlmostEqual` on strings where `assertEqual` was meant.
- The virtualenv, `__pycache__` and `db.sqlite3` were committed. They're untracked now but remain in history. The `.gitignore` had a typo (`.DS_ignore`); it's fixed now.
- Django 3.1 and Python 3.8 are both past end of life.

## What I'd do differently now

- Keep secrets, the virtualenv and the database out of git from the first commit.
- Configure static files, a hosted database and environment-based settings before deploying.
- Run `manage.py check --deploy` and a local run with `DEBUG=False` before pushing to a host.
