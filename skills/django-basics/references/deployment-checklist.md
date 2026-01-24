# Django Production Deployment Checklist

**Pre-deployment checklist for Django applications**

---

## Security Settings

### SECRET_KEY

```python
# ❌ NEVER hardcode in settings.py
SECRET_KEY = 'django-insecure-hardcoded-key'

# ✅ Use environment variables
import os
SECRET_KEY = os.environ.get('DJANGO_SECRET_KEY')

# ✅ Or use python-decouple
from decouple import config
SECRET_KEY = config('SECRET_KEY')

# Generate new secret key:
# python -c 'from django.core.management.utils import get_random_secret_key; print(get_random_secret_key())'
```

- [ ] SECRET_KEY is not hardcoded
- [ ] SECRET_KEY is loaded from environment variable
- [ ] SECRET_KEY is not in version control
- [ ] SECRET_KEY is sufficiently random (50+ characters)

### DEBUG

```python
# ❌ NEVER True in production
DEBUG = True

# ✅ Set to False
DEBUG = False

# ✅ Or use environment variable
DEBUG = os.environ.get('DJANGO_DEBUG', 'False') == 'True'
```

- [ ] DEBUG = False
- [ ] Custom error pages created (404.html, 500.html)
- [ ] ADMINS configured for error emails

### ALLOWED_HOSTS

```python
# ❌ Empty list won't work
ALLOWED_HOSTS = []

# ✅ Add your domain(s)
ALLOWED_HOSTS = ['example.com', 'www.example.com']

# ✅ Or from environment variable
ALLOWED_HOSTS = os.environ.get('DJANGO_ALLOWED_HOSTS', '').split(',')
```

- [ ] ALLOWED_HOSTS configured with production domains
- [ ] No wildcard ('*') in production
- [ ] Includes www and non-www versions if needed

---

## Database

```python
# ❌ Don't use SQLite in production
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.sqlite3',
        'NAME': BASE_DIR / 'db.sqlite3',
    }
}

# ✅ Use PostgreSQL
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql',
        'NAME': os.environ.get('DB_NAME'),
        'USER': os.environ.get('DB_USER'),
        'PASSWORD': os.environ.get('DB_PASSWORD'),
        'HOST': os.environ.get('DB_HOST', 'localhost'),
        'PORT': os.environ.get('DB_PORT', '5432'),
        'CONN_MAX_AGE': 600,  # Connection pooling
    }
}
```

- [ ] Using PostgreSQL (or MySQL/MariaDB)
- [ ] Database credentials in environment variables
- [ ] Database backups configured
- [ ] Connection pooling enabled (CONN_MAX_AGE)
- [ ] Database indexed appropriately

---

## Static Files

```python
STATIC_URL = '/static/'
STATIC_ROOT = BASE_DIR / 'staticfiles'
STATICFILES_DIRS = [BASE_DIR / 'static']

# For production
STATICFILES_STORAGE = 'whitenoise.storage.CompressedManifestStaticFilesStorage'
```

- [ ] STATIC_ROOT configured
- [ ] `python manage.py collectstatic` runs successfully
- [ ] Static files served by web server (nginx) or CDN
- [ ] WhiteNoise configured if not using separate server
- [ ] Static files compressed (gzip)
- [ ] Cache headers set for static files

### Media Files

```python
MEDIA_URL = '/media/'
MEDIA_ROOT = BASE_DIR / 'media'

# For production with cloud storage
DEFAULT_FILE_STORAGE = 'storages.backends.s3boto3.S3Boto3Storage'
```

- [ ] MEDIA_ROOT configured
- [ ] Media files not served by Django in production
- [ ] User uploads stored securely (S3, cloud storage)
- [ ] File upload size limits configured
- [ ] Virus scanning for uploads (if needed)

---

## Security Headers

```python
if not DEBUG:
    # HTTPS
    SECURE_SSL_REDIRECT = True
    SECURE_PROXY_SSL_HEADER = ('HTTP_X_FORWARDED_PROTO', 'https')

    # HSTS
    SECURE_HSTS_SECONDS = 31536000  # 1 year
    SECURE_HSTS_INCLUDE_SUBDOMAINS = True
    SECURE_HSTS_PRELOAD = True

    # Cookies
    SESSION_COOKIE_SECURE = True
    CSRF_COOKIE_SECURE = True
    SESSION_COOKIE_HTTPONLY = True
    CSRF_COOKIE_HTTPONLY = True

    # Other
    X_FRAME_OPTIONS = 'DENY'
    SECURE_CONTENT_TYPE_NOSNIFF = True
    SECURE_BROWSER_XSS_FILTER = True
```

- [ ] HTTPS enabled (SSL certificate installed)
- [ ] HSTS configured
- [ ] Secure cookie settings enabled
- [ ] X-Frame-Options configured
- [ ] Content type sniffing disabled
- [ ] XSS filter enabled

### Django 6.0: Content Security Policy

```python
from django.utils.csp import CSP

MIDDLEWARE = [
    # ...
    'django.middleware.csp.ContentSecurityPolicyMiddleware',
]

SECURE_CSP = {
    "default-src": [CSP.SELF],
    "script-src": [CSP.SELF, CSP.NONCE],
    "style-src": [CSP.SELF, CSP.NONCE],
    "img-src": [CSP.SELF, "https:", "data:"],
    "font-src": [CSP.SELF, "https://fonts.gstatic.com"],
}
```

- [ ] CSP configured
- [ ] CSP tested and working
- [ ] No inline scripts without nonces

---

## Middleware

```python
MIDDLEWARE = [
    'django.middleware.security.SecurityMiddleware',
    'whitenoise.middleware.WhiteNoiseMiddleware',  # If using WhiteNoise
    'django.contrib.sessions.middleware.SessionMiddleware',
    'django.middleware.common.CommonMiddleware',
    'django.middleware.csrf.CsrfViewMiddleware',
    'django.middleware.csp.ContentSecurityPolicyMiddleware',  # Django 6.0
    'django.contrib.auth.middleware.AuthenticationMiddleware',
    'django.contrib.messages.middleware.MessageMiddleware',
    'django.middleware.clickjacking.XFrameOptionsMiddleware',
]
```

- [ ] SecurityMiddleware first
- [ ] WhiteNoise after SecurityMiddleware (if used)
- [ ] No debug middleware in production
- [ ] Custom middleware reviewed for security

---

## Logging

```python
LOGGING = {
    'version': 1,
    'disable_existing_loggers': False,
    'formatters': {
        'verbose': {
            'format': '{levelname} {asctime} {module} {message}',
            'style': '{',
        },
    },
    'handlers': {
        'file': {
            'level': 'ERROR',
            'class': 'logging.FileHandler',
            'filename': '/var/log/django/error.log',
            'formatter': 'verbose',
        },
    },
    'loggers': {
        'django': {
            'handlers': ['file'],
            'level': 'ERROR',
            'propagate': True,
        },
    },
}
```

- [ ] Logging configured
- [ ] Error logs being written
- [ ] Log rotation configured
- [ ] Sensitive data not logged (passwords, tokens)
- [ ] Monitoring/alerting set up for errors

---

## Email

```python
# ❌ Console backend (dev only)
EMAIL_BACKEND = 'django.core.mail.backends.console.EmailBackend'

# ✅ SMTP (production)
EMAIL_BACKEND = 'django.core.mail.backends.smtp.EmailBackend'
EMAIL_HOST = os.environ.get('EMAIL_HOST')
EMAIL_PORT = int(os.environ.get('EMAIL_PORT', 587))
EMAIL_USE_TLS = True
EMAIL_HOST_USER = os.environ.get('EMAIL_HOST_USER')
EMAIL_HOST_PASSWORD = os.environ.get('EMAIL_HOST_PASSWORD')
DEFAULT_FROM_EMAIL = 'noreply@example.com'
SERVER_EMAIL = 'server@example.com'

# ADMINS for error emails
ADMINS = [
    'Admin Name <admin@example.com>',
]
MANAGERS = ADMINS
```

- [ ] Email backend configured
- [ ] Email credentials in environment variables
- [ ] ADMINS configured for error emails
- [ ] Test email sending works
- [ ] SPF/DKIM configured for email domain

---

## Caching

```python
# ❌ Dummy cache (dev only)
CACHES = {
    'default': {
        'BACKEND': 'django.core.cache.backends.dummy.DummyCache',
    }
}

# ✅ Redis (production)
CACHES = {
    'default': {
        'BACKEND': 'django.core.cache.backends.redis.RedisCache',
        'LOCATION': os.environ.get('REDIS_URL', 'redis://127.0.0.1:6379/1'),
        'OPTIONS': {
            'CLIENT_CLASS': 'django_redis.client.DefaultClient',
        }
    }
}

# Session storage
SESSION_ENGINE = 'django.contrib.sessions.backends.cache'
SESSION_CACHE_ALIAS = 'default'
```

- [ ] Production cache backend configured (Redis/Memcached)
- [ ] Cache keys use version or key prefix
- [ ] Session storage configured
- [ ] Cache invalidation strategy planned

---

## Performance

```python
# Database connection pooling
DATABASES['default']['CONN_MAX_AGE'] = 600

# Template caching
TEMPLATES[0]['OPTIONS']['loaders'] = [
    ('django.template.loaders.cached.Loader', [
        'django.template.loaders.filesystem.Loader',
        'django.template.loaders.app_directories.Loader',
    ]),
]
```

- [ ] Database queries optimized (use select_related/prefetch_related)
- [ ] Database indexes added to frequently queried fields
- [ ] Template caching enabled
- [ ] Expensive operations cached
- [ ] Static files minified and compressed
- [ ] CDN configured for static files (if needed)
- [ ] Database connection pooling enabled

---

## Celery / Background Tasks (If Used)

```python
# Django 6.0 built-in tasks
TASKS = {
    "default": {
        "BACKEND": "django.tasks.backends.database.DatabaseBackend",
    }
}

# Or Celery
CELERY_BROKER_URL = os.environ.get('REDIS_URL')
CELERY_RESULT_BACKEND = os.environ.get('REDIS_URL')
```

- [ ] Task queue configured (Django 6.0 tasks or Celery)
- [ ] Task workers running
- [ ] Failed task monitoring
- [ ] Task results being cleaned up

---

## Management Commands

```bash
# Collect static files
python manage.py collectstatic --noinput

# Migrate database
python manage.py migrate

# Check deployment
python manage.py check --deploy

# Create superuser (if needed)
python manage.py createsuperuser

# Load fixtures (if needed)
python manage.py loaddata initial_data.json
```

- [ ] Migrations run successfully
- [ ] Static files collected
- [ ] `check --deploy` passes without warnings
- [ ] No pending migrations

---

## Web Server

### Gunicorn Configuration

```python
# gunicorn.conf.py
bind = '0.0.0.0:8000'
workers = 3  # (2 x num_cores) + 1
worker_class = 'gthread'
threads = 2
timeout = 30
keepalive = 2
max_requests = 1000
max_requests_jitter = 50
```

- [ ] Gunicorn (or uWSGI) configured
- [ ] Correct number of workers
- [ ] Timeout configured appropriately
- [ ] Worker auto-restart on crashes

### Nginx Configuration

```nginx
upstream django {
    server 127.0.0.1:8000;
}

server {
    listen 80;
    server_name example.com;
    return 301 https://$host$request_uri;
}

server {
    listen 443 ssl http2;
    server_name example.com;

    ssl_certificate /path/to/cert.pem;
    ssl_certificate_key /path/to/key.pem;

    location /static/ {
        alias /path/to/staticfiles/;
        expires 30d;
        add_header Cache-Control "public, immutable";
    }

    location /media/ {
        alias /path/to/media/;
        expires 7d;
    }

    location / {
        proxy_pass http://django;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

- [ ] Nginx (or Apache) configured
- [ ] HTTPS configured with valid certificate
- [ ] HTTP redirects to HTTPS
- [ ] Static files served by nginx
- [ ] Gzip compression enabled
- [ ] Proper headers set (X-Forwarded-For, etc.)

---

## Monitoring & Error Tracking

```python
# Sentry (example)
import sentry_sdk
from sentry_sdk.integrations.django import DjangoIntegration

sentry_sdk.init(
    dsn=os.environ.get('SENTRY_DSN'),
    integrations=[DjangoIntegration()],
    traces_sample_rate=1.0,
    send_default_pii=True,
    environment='production',
)
```

- [ ] Error tracking configured (Sentry, Rollbar, etc.)
- [ ] Application monitoring (New Relic, Datadog, etc.)
- [ ] Uptime monitoring
- [ ] Performance monitoring
- [ ] Database monitoring
- [ ] Disk space monitoring
- [ ] Alerting configured

---

## Backups

- [ ] Database backups automated
- [ ] Media files backups automated
- [ ] Backup restoration tested
- [ ] Offsite backup storage
- [ ] Backup retention policy defined

---

## Testing

```bash
# Run tests
python manage.py test

# Check code coverage
coverage run --source='.' manage.py test
coverage report
```

- [ ] All tests passing
- [ ] Test coverage acceptable (>80%)
- [ ] Integration tests for critical paths
- [ ] Load testing performed
- [ ] Security scanning performed

---

## Documentation

- [ ] README.md with setup instructions
- [ ] Environment variables documented
- [ ] Deployment process documented
- [ ] Rollback procedure documented
- [ ] API documentation (if applicable)

---

## Final Checks

```bash
# Django deployment check
python manage.py check --deploy

# Test production settings locally
python manage.py runserver --settings=myproject.settings.production
```

- [ ] `python manage.py check --deploy` passes
- [ ] All dependencies in requirements.txt
- [ ] Python version specified
- [ ] .gitignore configured (no secrets, db, media)
- [ ] requirements.txt up to date
- [ ] Database migrations tested
- [ ] Admin interface accessible
- [ ] All forms have CSRF tokens
- [ ] User authentication working
- [ ] Password reset working
- [ ] Email notifications working
- [ ] 404/500 error pages work
- [ ] Robots.txt configured
- [ ] Sitemap.xml configured (if needed)
- [ ] Analytics configured
- [ ] GDPR compliance (if applicable)

---

## Post-Deployment

- [ ] Verify site is accessible
- [ ] Test critical user flows
- [ ] Check error logs for issues
- [ ] Monitor performance metrics
- [ ] Verify backups are running
- [ ] Update documentation with any changes
- [ ] Notify team of deployment

---

## Quick Command Reference

```bash
# Check deployment
python manage.py check --deploy

# Collect static files
python manage.py collectstatic --noinput

# Run migrations
python manage.py migrate

# Create superuser
python manage.py createsuperuser

# Test email
python manage.py sendtestemail admin@example.com

# Clear cache
python manage.py clear_cache

# Show migrations
python manage.py showmigrations

# Database shell
python manage.py dbshell

# Python shell
python manage.py shell
```

---

**Run this command before every deployment:**
```bash
python manage.py check --deploy
```

This will catch most common security issues!
