# Django 6.0 Correction Rules

**Purpose**: Bridge training cutoff gaps for Django 6.0 breaking changes and best practices
**Last Updated**: 2026-01-24
**Django Version**: 6.0.1

---

## Python Version Requirement

### ❌ Outdated (Django 5.x and earlier)
```python
# Python 3.10 or 3.11
```

### ✅ Current (Django 6.0)
```python
# Python 3.12+ required
# Django 6.0 dropped support for Python 3.10 and 3.11
```

**Breaking Change**: Django 6.0 requires Python 3.12 or later. Attempting to run with earlier Python versions will fail.

---

## Default Auto Field

### ❌ Outdated (Django < 3.2)
```python
# settings.py
# No DEFAULT_AUTO_FIELD setting
# Uses AutoField by default (32-bit integer)
```

### ✅ Current (Django 6.0)
```python
# settings.py
DEFAULT_AUTO_FIELD = 'django.db.models.BigAutoField'
# BigAutoField is now the default (64-bit integer)
# Most projects won't need to change this
```

**Change**: Django 6.0 defaults to `BigAutoField` instead of `AutoField`. If you need the old behavior, explicitly set `DEFAULT_AUTO_FIELD = 'django.db.models.AutoField'` in settings.

---

## Content Security Policy (New in 6.0)

### ❌ Outdated (Django < 6.0)
```python
# No built-in CSP support
# Need third-party package: django-csp
```

### ✅ Current (Django 6.0)
```python
# settings.py
from django.utils.csp import CSP

MIDDLEWARE = [
    # ...
    'django.middleware.csp.ContentSecurityPolicyMiddleware',  # NEW
]

SECURE_CSP = {
    "default-src": [CSP.SELF],
    "script-src": [CSP.SELF, CSP.NONCE],
    "style-src": [CSP.SELF, CSP.NONCE],
    "img-src": [CSP.SELF, "https:"],
}

# In templates
{% load csp %}
<script nonce="{{ request.csp_nonce }}">
    // Inline script with nonce
</script>
```

**New Feature**: Django 6.0 has built-in Content Security Policy support. Remove third-party CSP packages.

---

## Background Tasks (New in 6.0)

### ❌ Outdated (Django < 6.0)
```python
# No built-in background tasks
# Need third-party package: Celery, Django-Q, or django-rq
```

### ✅ Current (Django 6.0)
```python
# settings.py
TASKS = {
    "default": {
        "BACKEND": "django.tasks.backends.database.DatabaseBackend",
    }
}

# myapp/tasks.py
from django.tasks import task

@task
def send_email(user_id):
    # Task implementation
    pass

# Enqueue task
send_email.enqueue(user_id=123)

# Check status
result = send_email.enqueue(user_id=123)
print(result.status)  # 'pending', 'running', 'complete', or 'failed'
```

**New Feature**: Django 6.0 includes a built-in background tasks framework. Consider migrating from Celery for simple use cases.

---

## Template Partials (New in 6.0)

### ❌ Outdated (Django < 6.0)
```django
{# Need separate template file for reusable components #}
{% include "components/card.html" %}

{# components/card.html #}
<div class="card">
    {{ content }}
</div>
```

### ✅ Current (Django 6.0)
```django
{# Define partial inline #}
{% partialdef card %}
<div class="card">
    {{ content }}
</div>
{% endpartialdef %}

{# Use the partial #}
{% partial card %}

{# Or include from another template #}
{% include "myapp/list.html#card" %}
```

**New Feature**: Django 6.0 supports template partials for inline reusable components without separate files.

---

## Email API Changes

### ❌ Outdated (Django < 6.0)
```python
from django.core.mail import EmailMessage
from email.mime.base import MIMEBase

# Old MIME-based API
msg = EmailMessage(subject, body, from_email, to)
attachment = MIMEBase('application', 'pdf')  # Deprecated
msg.attach(attachment)
```

### ✅ Current (Django 6.0)
```python
from django.core.mail import EmailMessage
from email.message import MIMEPart

# Modern EmailMessage API (uses email.message.EmailMessage)
msg = EmailMessage(subject, body, from_email, to)

# Use MIMEPart instead of MIMEBase
attachment = MIMEPart()
msg.attach(attachment)

# Or use simpler attach methods
msg.attach_file('/path/to/file.pdf')
```

**Breaking Change**: Django 6.0 uses Python's modern `email.message.EmailMessage` class. Update custom email code accordingly.

---

## ORM Expression as_sql() Return Type

### ❌ Outdated (Django < 6.0)
```python
from django.db.models import Func

class CustomExpression(Func):
    def as_sql(self, compiler, connection):
        sql_string = "CUSTOM SQL"
        params = ['value1', 'value2']  # Returns list
        return sql_string, params
```

### ✅ Current (Django 6.0)
```python
from django.db.models import Func

class CustomExpression(Func):
    def as_sql(self, compiler, connection):
        sql_string = "CUSTOM SQL"
        params = ('value1', 'value2')  # Must return tuple
        return sql_string, params
```

**Breaking Change**: `as_sql()` methods in custom ORM expressions must return params as a tuple, not a list.

---

## URL Filter HTTPS Default

### ❌ Outdated (Django < 6.0)
```django
{# urlize assumes HTTP by default #}
{{ user_input|urlize }}
{# Output: <a href="http://example.com">example.com</a> #}
```

### ✅ Current (Django 6.0)
```python
# settings.py
URLIZE_ASSUME_HTTPS = True  # Will be default in Django 7.0
```

```django
{# Now assumes HTTPS #}
{{ user_input|urlize }}
{# Output: <a href="https://example.com">example.com</a> #}
```

**Deprecation Warning**: Add `URLIZE_ASSUME_HTTPS = True` to prepare for Django 7.0 where it will be the default.

---

## Admin Settings Format

### ❌ Outdated (Django < 6.0)
```python
# settings.py
ADMINS = [
    ('John Doe', 'john@example.com'),  # Tuple format deprecated
]

MANAGERS = [
    ('Jane Smith', 'jane@example.com'),  # Tuple format deprecated
]
```

### ✅ Current (Django 6.0)
```python
# settings.py
ADMINS = [
    'John Doe <john@example.com>',  # String format
]

MANAGERS = [
    'Jane Smith <jane@example.com>',  # String format
]
```

**Deprecation Warning**: Tuple format for `ADMINS` and `MANAGERS` is deprecated. Use email string format.

---

## Model __str__() Best Practice

### ❌ Outdated Pattern
```python
class Article(models.Model):
    title = models.CharField(max_length=200)

    # No __str__() method
```

### ✅ Current Best Practice
```python
class Article(models.Model):
    title = models.CharField(max_length=200)

    def __str__(self):
        return self.title  # Always define __str__()
```

**Best Practice**: Always define `__str__()` for readable representations in admin and debugging.

---

## Migration File Version Control

### ❌ Bad Practice
```bash
# .gitignore
*/migrations/*.py  # Never ignore migrations!
```

### ✅ Best Practice
```bash
# .gitignore
*/migrations/__pycache__/
db.sqlite3

# migrations/ folder and .py files should be committed
```

**Best Practice**: Always version control migration files. They are essential for database schema history.

---

## Static Files in Production

### ❌ Outdated Pattern
```python
# settings.py
DEBUG = True  # Never in production!

# urls.py - serving static files with Django
from django.conf.urls.static import static
urlpatterns += static(settings.STATIC_URL, document_root=settings.STATIC_ROOT)
```

### ✅ Current Best Practice
```python
# settings.py
DEBUG = False
STATIC_ROOT = BASE_DIR / 'staticfiles'

# Use whitenoise for static files
MIDDLEWARE = [
    'django.middleware.security.SecurityMiddleware',
    'whitenoise.middleware.WhiteNoiseMiddleware',  # Add this
    # ...
]

# Never serve static files with Django in production
# Use nginx, whitenoise, or CDN
```

**Best Practice**: Never serve static files with Django in production. Use WhiteNoise, nginx, or a CDN.

---

## Database for Production

### ❌ Bad Practice
```python
# settings.py
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.sqlite3',  # Don't use in production!
        'NAME': BASE_DIR / 'db.sqlite3',
    }
}
```

### ✅ Best Practice
```python
# settings.py
import os

DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql',  # Use PostgreSQL
        'NAME': os.environ.get('DB_NAME'),
        'USER': os.environ.get('DB_USER'),
        'PASSWORD': os.environ.get('DB_PASSWORD'),
        'HOST': os.environ.get('DB_HOST', 'localhost'),
        'PORT': os.environ.get('DB_PORT', '5432'),
    }
}
```

**Best Practice**: SQLite is fine for development, but use PostgreSQL in production. SQLite doesn't handle concurrent writes well.

---

## Security Settings

### ❌ Outdated Pattern
```python
# settings.py
SECRET_KEY = 'django-insecure-hardcoded-key'  # Never hardcode!
DEBUG = True  # Never in production!
ALLOWED_HOSTS = []  # Won't work in production!
```

### ✅ Current Best Practice
```python
# settings.py
import os

SECRET_KEY = os.environ.get('DJANGO_SECRET_KEY')  # From environment
DEBUG = os.environ.get('DJANGO_DEBUG', 'False') == 'True'
ALLOWED_HOSTS = os.environ.get('DJANGO_ALLOWED_HOSTS', '').split(',')

# Security headers (production)
if not DEBUG:
    SECURE_SSL_REDIRECT = True
    SESSION_COOKIE_SECURE = True
    CSRF_COOKIE_SECURE = True
    SECURE_HSTS_SECONDS = 31536000
    SECURE_HSTS_INCLUDE_SUBDOMAINS = True
    SECURE_HSTS_PRELOAD = True
```

**Best Practice**: Use environment variables for secrets, set `DEBUG = False` in production, and enable security headers.

---

## Query Optimization

### ❌ Inefficient Pattern
```python
# N+1 query problem
articles = Article.objects.all()
for article in articles:
    print(article.author.name)  # Hits database for each author!
```

### ✅ Optimized Pattern
```python
# Use select_related for ForeignKey/OneToOne
articles = Article.objects.select_related('author').all()
for article in articles:
    print(article.author.name)  # No extra queries!

# Use prefetch_related for ManyToMany/reverse FK
articles = Article.objects.prefetch_related('tags').all()
for article in articles:
    print(article.tags.all())  # Optimized with prefetch
```

**Best Practice**: Always use `select_related()` and `prefetch_related()` to avoid N+1 query problems.

---

## Form Validation

### ❌ Outdated Pattern
```python
# views.py
def create_article(request):
    if request.method == 'POST':
        title = request.POST.get('title')
        # No validation!
        Article.objects.create(title=title)
```

### ✅ Current Best Practice
```python
# forms.py
from django import forms
from .models import Article

class ArticleForm(forms.ModelForm):
    class Meta:
        model = Article
        fields = ['title', 'content']

    def clean_title(self):
        title = self.cleaned_data.get('title')
        if len(title) < 5:
            raise forms.ValidationError('Title too short')
        return title

# views.py
def create_article(request):
    if request.method == 'POST':
        form = ArticleForm(request.POST)
        if form.is_valid():  # Validates automatically
            form.save()
            return redirect('success')
    else:
        form = ArticleForm()
    return render(request, 'form.html', {'form': form})
```

**Best Practice**: Always use Django forms for validation. Never trust user input directly.

---

## CSRF Protection

### ❌ Missing CSRF Token
```django
<form method="post">
    {# Missing {% csrf_token %} - Will fail! #}
    {{ form.as_p }}
    <button type="submit">Submit</button>
</form>
```

### ✅ With CSRF Token
```django
<form method="post">
    {% csrf_token %}  {# Always include this! #}
    {{ form.as_p }}
    <button type="submit">Submit</button>
</form>
```

**Critical**: Always include `{% csrf_token %}` in POST forms to prevent CSRF attacks.

---

## Template Loading

### ❌ Wrong Directory Structure
```
myapp/
├── templates/
│   └── index.html  # Wrong! Name collision risk
```

### ✅ Correct Directory Structure
```
myapp/
├── templates/
│   └── myapp/  # App name twice prevents collisions
│       └── index.html
```

**Best Practice**: Use `myapp/templates/myapp/` structure to avoid template name collisions between apps.

---

**Summary**: Django 6.0 introduces several breaking changes (Python 3.12+ requirement, BigAutoField default, email API changes, ORM expression params as tuples) and new features (CSP, background tasks, template partials). Always use environment variables for secrets, PostgreSQL for production, and follow Django's security best practices.
