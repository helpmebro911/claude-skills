---
name: django-basics
description: |
  Build web applications with Django 6.0 - Python's batteries-included framework. Covers MVT architecture, ORM with migrations, URL routing, forms with validation, built-in admin interface, and background tasks. Includes CSP security, template partials, and async support.

  Use when: starting Django projects, configuring settings, designing models and migrations, building views and templates, troubleshooting "No module named", CSRF errors, migration conflicts, or static file issues.
---

# Django Basics

**Status**: Production Ready ✅
**Last Updated**: 2026-01-24
**Dependencies**: Python 3.12+
**Latest Version**: Django 6.0.1 (released 2025-12-03)

---

## Quick Start (10 Minutes)

### 1. Install Django and Create Project

```bash
# Create virtual environment (recommended)
python3 -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate

# Install Django 6.0
pip install Django==6.0.1

# Create project
django-admin startproject myproject
cd myproject

# Run development server
python manage.py runserver
```

**Why this matters:**
- Virtual environments isolate project dependencies
- Django 6.0 requires Python 3.12+ (breaking change from 5.x)
- Development server auto-reloads on code changes
- Project structure separates configuration from apps

### 2. Create Your First App

```bash
# Create an app
python manage.py startapp myapp

# Add to INSTALLED_APPS in settings.py
```

```python
# myproject/settings.py
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'myapp',  # Add your app here
]
```

**CRITICAL:**
- Apps must be added to `INSTALLED_APPS` before models are recognized
- Use `python manage.py startapp` (not manual folder creation) for proper structure
- App names should be lowercase, plural, and descriptive (e.g., `polls`, `users`, `products`)

### 3. Define Models and Migrate

```python
# myapp/models.py
from django.db import models

class Article(models.Model):
    title = models.CharField(max_length=200)
    content = models.TextField()
    published_date = models.DateTimeField(auto_now_add=True)
    updated_date = models.DateTimeField(auto_now=True)
    is_published = models.BooleanField(default=False)

    class Meta:
        ordering = ['-published_date']
        verbose_name_plural = "Articles"

    def __str__(self):
        return self.title
```

```bash
# Create and apply migrations
python manage.py makemigrations
python manage.py migrate

# Create superuser for admin
python manage.py createsuperuser
```

**Key Points:**
- `makemigrations` creates migration files (version control these!)
- `migrate` applies migrations to database
- Always run `makemigrations` after model changes
- Use `__str__()` for readable model representations in admin

---

## The 7-Step Django Setup Process

### Step 1: Project Structure Setup

```bash
# Recommended project structure
myproject/
├── manage.py
├── myproject/              # Project configuration
│   ├── __init__.py
│   ├── settings.py         # Main settings
│   ├── urls.py             # Root URL configuration
│   ├── asgi.py             # ASGI deployment
│   └── wsgi.py             # WSGI deployment
├── myapp/                  # Your application
│   ├── migrations/
│   ├── templates/myapp/    # App-specific templates
│   ├── static/myapp/       # App-specific static files
│   ├── __init__.py
│   ├── admin.py
│   ├── apps.py
│   ├── models.py
│   ├── tests.py
│   ├── urls.py
│   └── views.py
├── templates/              # Project-wide templates
├── static/                 # Project-wide static files
├── media/                  # User-uploaded files
└── requirements.txt
```

**Key Points:**
- Keep apps self-contained with their own templates/static directories
- Use `templates/myapp/` not just `templates/` to avoid naming conflicts
- Never commit `db.sqlite3` or `media/` to version control

### Step 2: Configure Settings

```python
# myproject/settings.py

# Security (CHANGE IN PRODUCTION!)
SECRET_KEY = 'django-insecure-CHANGE-THIS-IN-PRODUCTION'
DEBUG = True  # Set to False in production
ALLOWED_HOSTS = []  # Add your domain in production

# Database (default SQLite for development)
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.sqlite3',
        'NAME': BASE_DIR / 'db.sqlite3',
    }
}

# Django 6.0: BigAutoField is now default
DEFAULT_AUTO_FIELD = 'django.db.models.BigAutoField'

# Django 6.0: Content Security Policy (NEW)
MIDDLEWARE = [
    'django.middleware.security.SecurityMiddleware',
    'django.contrib.sessions.middleware.SessionMiddleware',
    'django.middleware.common.CommonMiddleware',
    'django.middleware.csrf.CsrfViewMiddleware',
    'django.middleware.csp.ContentSecurityPolicyMiddleware',  # NEW in 6.0
    'django.contrib.auth.middleware.AuthenticationMiddleware',
    'django.contrib.messages.middleware.MessageMiddleware',
    'django.middleware.clickjacking.XFrameOptionsMiddleware',
]

# Django 6.0: CSP configuration
from django.utils.csp import CSP

SECURE_CSP = {
    "default-src": [CSP.SELF],
    "script-src": [CSP.SELF, CSP.NONCE],
    "style-src": [CSP.SELF, CSP.NONCE],
    "img-src": [CSP.SELF, "https:"],
}

# Static files
STATIC_URL = '/static/'
STATIC_ROOT = BASE_DIR / 'staticfiles'

# Media files
MEDIA_URL = '/media/'
MEDIA_ROOT = BASE_DIR / 'media'

# Timezone and internationalization
TIME_ZONE = 'UTC'
USE_TZ = True
```

**Why these settings:**
- `SECRET_KEY` must be random and secret (use environment variables in production)
- `DEBUG = False` in production prevents sensitive error pages
- `ALLOWED_HOSTS` prevents HTTP Host header attacks
- CSP protects against XSS attacks (new in Django 6.0)
- `USE_TZ = True` enables timezone-aware datetimes

### Step 3: URL Configuration

```python
# myproject/urls.py (root URLconf)
from django.contrib import admin
from django.urls import path, include
from django.conf import settings
from django.conf.urls.static import static

urlpatterns = [
    path('admin/', admin.site.urls),
    path('', include('myapp.urls')),  # Include app URLs
]

# Serve media files in development
if settings.DEBUG:
    urlpatterns += static(settings.MEDIA_URL, document_root=settings.MEDIA_ROOT)
```

```python
# myapp/urls.py
from django.urls import path
from . import views

app_name = 'myapp'  # Namespace for URL reversing

urlpatterns = [
    path('', views.index, name='index'),
    path('article/<int:pk>/', views.article_detail, name='article_detail'),
    path('article/create/', views.article_create, name='article_create'),
]
```

**Key Points:**
- Use `include()` to organize URLs by app
- Add `app_name` for namespaced URL reversing (`{% url 'myapp:index' %}`)
- Use path converters: `<int:pk>`, `<slug:slug>`, `<str:username>`
- Never serve media files via Django in production (use nginx/whitenoise)

### Step 4: Create Views

```python
# myapp/views.py
from django.shortcuts import render, get_object_or_404, redirect
from django.views.generic import ListView, DetailView, CreateView
from django.contrib.auth.decorators import login_required
from django.contrib.auth.mixins import LoginRequiredMixin
from .models import Article
from .forms import ArticleForm

# Function-based view
def index(request):
    articles = Article.objects.filter(is_published=True)
    context = {'articles': articles}
    return render(request, 'myapp/index.html', context)

# Class-based view
class ArticleListView(ListView):
    model = Article
    template_name = 'myapp/article_list.html'
    context_object_name = 'articles'
    paginate_by = 10

    def get_queryset(self):
        return Article.objects.filter(is_published=True)

# Detail view
def article_detail(request, pk):
    article = get_object_or_404(Article, pk=pk, is_published=True)
    return render(request, 'myapp/article_detail.html', {'article': article})

# Protected view (login required)
@login_required
def article_create(request):
    if request.method == 'POST':
        form = ArticleForm(request.POST)
        if form.is_valid():
            article = form.save()
            return redirect('myapp:article_detail', pk=article.pk)
    else:
        form = ArticleForm()
    return render(request, 'myapp/article_form.html', {'form': form})

# Class-based protected view
class ArticleCreateView(LoginRequiredMixin, CreateView):
    model = Article
    form_class = ArticleForm
    template_name = 'myapp/article_form.html'
```

**When to use:**
- **Function-based views**: Simple logic, custom workflows
- **Class-based views**: CRUD operations, reusable patterns
- Always use `get_object_or_404()` instead of `Model.objects.get()` for better UX

### Step 5: Create Templates

```django
{# myapp/templates/myapp/base.html #}
{% load static %}
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>{% block title %}My Site{% endblock %}</title>
    <link rel="stylesheet" href="{% static 'myapp/css/style.css' %}">
</head>
<body>
    {% block content %}{% endblock %}
    <script src="{% static 'myapp/js/main.js' %}"></script>
</body>
</html>
```

```django
{# myapp/templates/myapp/article_list.html #}
{% extends "myapp/base.html" %}

{% block title %}Articles{% endblock %}

{% block content %}
<h1>Articles</h1>
{% for article in articles %}
    <article>
        <h2><a href="{% url 'myapp:article_detail' article.pk %}">{{ article.title }}</a></h2>
        <p>{{ article.content|truncatewords:50 }}</p>
        <time>{{ article.published_date|date:"F j, Y" }}</time>
    </article>
{% empty %}
    <p>No articles yet.</p>
{% endfor %}

{# Pagination #}
{% if is_paginated %}
<nav>
    {% if page_obj.has_previous %}
        <a href="?page=1">First</a>
        <a href="?page={{ page_obj.previous_page_number }}">Previous</a>
    {% endif %}

    Page {{ page_obj.number }} of {{ page_obj.paginator.num_pages }}

    {% if page_obj.has_next %}
        <a href="?page={{ page_obj.next_page_number }}">Next</a>
        <a href="?page={{ page_obj.paginator.num_pages }}">Last</a>
    {% endif %}
</nav>
{% endif %}
{% endblock %}
```

**Django 6.0: Template Partials (NEW)**

```django
{# Define reusable partial #}
{% partialdef article-card %}
<article class="card">
    <h3>{{ article.title }}</h3>
    <p>{{ article.content|truncatewords:30 }}</p>
    <a href="{% url 'myapp:article_detail' article.pk %}">Read more</a>
</article>
{% endpartialdef %}

{# Use the partial #}
{% for article in articles %}
    {% partial article-card %}
{% endfor %}

{# Include partial from another template #}
{% include "myapp/article_list.html#article-card" %}
```

### Step 6: Django ORM Queries

```python
# Basic queries
articles = Article.objects.all()
published = Article.objects.filter(is_published=True)
article = Article.objects.get(pk=1)  # Raises DoesNotExist if not found
first = Article.objects.first()
count = Article.objects.count()

# Advanced filters
from django.utils import timezone

recent = Article.objects.filter(
    published_date__gte=timezone.now() - timezone.timedelta(days=7)
)
search = Article.objects.filter(title__icontains='django')
excluded = Article.objects.exclude(is_published=False)

# Chaining and Q objects
from django.db.models import Q

complex = Article.objects.filter(
    Q(title__icontains='django') | Q(content__icontains='python')
).exclude(is_published=False).order_by('-published_date')[:10]

# Aggregation
from django.db.models import Count, Avg

stats = Article.objects.aggregate(
    total=Count('id'),
    avg_length=Avg('content__len')
)

# Select related (optimize queries)
# Use select_related for ForeignKey and OneToOne
articles = Article.objects.select_related('author').all()

# Use prefetch_related for ManyToMany and reverse ForeignKey
articles = Article.objects.prefetch_related('tags').all()

# Create objects
article = Article.objects.create(
    title='New Article',
    content='Content here',
    is_published=True
)

# Update objects
Article.objects.filter(pk=1).update(is_published=True)
article.title = 'Updated Title'
article.save()

# Delete objects
Article.objects.filter(pk=1).delete()
article.delete()
```

**Query optimization tips:**
- Use `select_related()` and `prefetch_related()` to reduce database queries
- Use `only()` and `defer()` to load only needed fields
- Use `iterator()` for large querysets to save memory
- Check query count with `django-debug-toolbar` in development

### Step 7: Forms and Validation

```python
# myapp/forms.py
from django import forms
from .models import Article

# ModelForm (recommended for model-based forms)
class ArticleForm(forms.ModelForm):
    class Meta:
        model = Article
        fields = ['title', 'content', 'is_published']
        widgets = {
            'title': forms.TextInput(attrs={'class': 'form-control'}),
            'content': forms.Textarea(attrs={'class': 'form-control', 'rows': 5}),
        }

    def clean_title(self):
        title = self.cleaned_data.get('title')
        if len(title) < 5:
            raise forms.ValidationError('Title must be at least 5 characters.')
        return title

# Regular Form (for non-model forms)
class ContactForm(forms.Form):
    name = forms.CharField(max_length=100, required=True)
    email = forms.EmailField(required=True)
    message = forms.CharField(widget=forms.Textarea, required=True)

    def clean_email(self):
        email = self.cleaned_data.get('email')
        if not email.endswith('@example.com'):
            raise forms.ValidationError('Please use your company email.')
        return email
```

```python
# Using forms in views
from django.views.decorators.http import require_http_methods

@require_http_methods(["GET", "POST"])
def contact_view(request):
    if request.method == 'POST':
        form = ContactForm(request.POST)
        if form.is_valid():
            # Process form data
            name = form.cleaned_data['name']
            email = form.cleaned_data['email']
            message = form.cleaned_data['message']
            # Send email, save to database, etc.
            return redirect('success')
    else:
        form = ContactForm()

    return render(request, 'myapp/contact.html', {'form': form})
```

```django
{# Template with form #}
<form method="post">
    {% csrf_token %}
    {{ form.as_p }}
    <button type="submit">Submit</button>
</form>

{# Or manually render fields #}
<form method="post">
    {% csrf_token %}
    {% for field in form %}
        <div class="field">
            {{ field.label_tag }}
            {{ field }}
            {% if field.errors %}
                <div class="error">{{ field.errors }}</div>
            {% endif %}
        </div>
    {% endfor %}
    <button type="submit">Submit</button>
</form>
```

---

## Critical Rules

### Always Do

✅ **Use migrations for all database changes** - Never modify database manually
✅ **Add `{% csrf_token %}` to all POST forms** - Prevents CSRF attacks
✅ **Use `get_object_or_404()` in views** - Better UX than 500 errors
✅ **Set `DEBUG = False` in production** - Prevents information disclosure
✅ **Use environment variables for secrets** - Never hardcode credentials
✅ **Run `python manage.py check` before deployment** - Catches configuration errors
✅ **Use `{% static %}` and `{% url %}` template tags** - Avoid hardcoded paths
✅ **Enable HTTPS in production** - Set `SECURE_SSL_REDIRECT = True`

### Never Do

❌ **Never commit `SECRET_KEY` to version control** - Use environment variables
❌ **Never use `Model.objects.get()` without exception handling** - Use `get_object_or_404()`
❌ **Never trust user input** - Always validate and sanitize
❌ **Never use `safe` filter unless absolutely necessary** - XSS risk
❌ **Never delete migration files** - Breaks database history
❌ **Never run development server in production** - Use gunicorn/uwsgi
❌ **Never use `SELECT *`-equivalent queries** - Use `only()` to select specific fields
❌ **Never forget `related_name` on reverse relations** - Makes queries readable

---

## Django 6.0 New Features

### 1. Content Security Policy (CSP)

```python
# settings.py
from django.utils.csp import CSP

MIDDLEWARE = [
    # ...
    'django.middleware.csp.ContentSecurityPolicyMiddleware',
]

SECURE_CSP = {
    "default-src": [CSP.SELF],
    "script-src": [CSP.SELF, CSP.NONCE],
    "style-src": [CSP.SELF, CSP.NONCE, "https://cdn.example.com"],
    "img-src": [CSP.SELF, "https:", "data:"],
    "font-src": [CSP.SELF, "https://fonts.gstatic.com"],
}

# Use nonce in templates
{% load csp %}
<script nonce="{{ request.csp_nonce }}">
    // Your inline script
</script>
```

### 2. Background Tasks

```python
# myapp/tasks.py
from django.tasks import task
from django.core.mail import send_mail
from django.contrib.auth import get_user_model

User = get_user_model()

@task
def send_welcome_email(user_id):
    user = User.objects.get(pk=user_id)
    send_mail(
        'Welcome!',
        f'Hello {user.username}',
        'noreply@example.com',
        [user.email],
    )
    return f'Email sent to {user.email}'

# Enqueue task
send_welcome_email.enqueue(user_id=user.id)

# Check task status
result = send_welcome_email.enqueue(user_id=user.id)
print(result.status)  # 'pending', 'running', 'complete', or 'failed'
```

**Configure task backend in settings:**

```python
# settings.py
TASKS = {
    "default": {
        "BACKEND": "django.tasks.backends.database.DatabaseBackend",
    }
}
```

### 3. Template Partials

```django
{# Define partial #}
{% partialdef user-badge %}
<div class="badge">
    <img src="{{ user.avatar }}" alt="{{ user.username }}">
    <span>{{ user.username }}</span>
</div>
{% endpartialdef %}

{# Use partial #}
{% partial user-badge %}

{# Include from another template #}
{% include "users/profile.html#user-badge" %}
```

---

## Known Issues Prevention

This skill prevents **8** documented issues:

### Issue #1: ImproperlyConfigured - "No module named 'myapp'"
**Error**: `django.core.exceptions.ImproperlyConfigured: No module named 'myapp'`
**Source**: [Django Documentation](https://docs.djangoproject.com/en/6.0/ref/exceptions/#improperlyConfigured)
**Why It Happens**: App not added to `INSTALLED_APPS` in settings.py
**Prevention**: Always add app to `INSTALLED_APPS` after running `startapp`

### Issue #2: CSRF Verification Failed
**Error**: `Forbidden (403): CSRF verification failed. Request aborted.`
**Source**: [Django CSRF Documentation](https://docs.djangoproject.com/en/6.0/ref/csrf/)
**Why It Happens**: Missing `{% csrf_token %}` in POST forms
**Prevention**: Always include `{% csrf_token %}` inside `<form method="post">`

### Issue #3: TemplateDoesNotExist
**Error**: `django.template.exceptions.TemplateDoesNotExist: myapp/index.html`
**Source**: [Django Template Loading](https://docs.djangoproject.com/en/6.0/topics/templates/)
**Why It Happens**: Templates not in correct directory or app not in `INSTALLED_APPS`
**Prevention**: Use `myapp/templates/myapp/template.html` structure (app name twice)

### Issue #4: Migration Conflicts
**Error**: `CommandError: Conflicting migrations detected`
**Source**: [Django Migrations](https://docs.djangoproject.com/en/6.0/topics/migrations/)
**Why It Happens**: Multiple developers created migrations for same model
**Prevention**: Run `makemigrations` before creating new migrations; use `--merge` to resolve

### Issue #5: Static Files Not Loading
**Error**: Static files 404 in production
**Source**: [Django Static Files](https://docs.djangoproject.com/en/6.0/howto/static-files/)
**Why It Happens**: `STATIC_ROOT` not configured or `collectstatic` not run
**Prevention**: Set `STATIC_ROOT`, run `python manage.py collectstatic` before deployment

### Issue #6: Database Locked (SQLite)
**Error**: `sqlite3.OperationalError: database is locked`
**Source**: [SQLite Limitations](https://docs.djangoproject.com/en/6.0/ref/databases/#sqlite-notes)
**Why It Happens**: SQLite doesn't handle concurrent writes
**Prevention**: Use PostgreSQL in production; SQLite only for development

### Issue #7: SECRET_KEY in Production
**Error**: Security warning about `SECRET_KEY`
**Source**: [Django Security](https://docs.djangoproject.com/en/6.0/topics/security/)
**Why It Happens**: Using default or committed `SECRET_KEY`
**Prevention**: Use environment variables:

```python
import os
SECRET_KEY = os.environ.get('DJANGO_SECRET_KEY', 'dev-key-change-in-production')
```

### Issue #8: Python Version Incompatibility (Django 6.0)
**Error**: `Python 3.12 or later is required`
**Source**: [Django 6.0 Release Notes](https://docs.djangoproject.com/en/6.0/releases/6.0/)
**Why It Happens**: Django 6.0 dropped support for Python 3.10 and 3.11
**Prevention**: Upgrade to Python 3.12+ before using Django 6.0

---

## Configuration Files Reference

### settings.py (Production-Ready Example)

```python
import os
from pathlib import Path
from django.utils.csp import CSP

BASE_DIR = Path(__file__).resolve().parent.parent

# Security
SECRET_KEY = os.environ.get('DJANGO_SECRET_KEY')
DEBUG = os.environ.get('DJANGO_DEBUG', 'False') == 'True'
ALLOWED_HOSTS = os.environ.get('DJANGO_ALLOWED_HOSTS', '').split(',')

# Application definition
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    # Third-party apps
    # Your apps
    'myapp',
]

MIDDLEWARE = [
    'django.middleware.security.SecurityMiddleware',
    'django.contrib.sessions.middleware.SessionMiddleware',
    'django.middleware.common.CommonMiddleware',
    'django.middleware.csrf.CsrfViewMiddleware',
    'django.middleware.csp.ContentSecurityPolicyMiddleware',
    'django.contrib.auth.middleware.AuthenticationMiddleware',
    'django.contrib.messages.middleware.MessageMiddleware',
    'django.middleware.clickjacking.XFrameOptionsMiddleware',
]

ROOT_URLCONF = 'myproject.urls'

TEMPLATES = [
    {
        'BACKEND': 'django.template.backends.django.DjangoTemplates',
        'DIRS': [BASE_DIR / 'templates'],
        'APP_DIRS': True,
        'OPTIONS': {
            'context_processors': [
                'django.template.context_processors.debug',
                'django.template.context_processors.request',
                'django.contrib.auth.context_processors.auth',
                'django.contrib.messages.context_processors.messages',
                'django.template.context_processors.csp',  # For CSP nonces
            ],
        },
    },
]

WSGI_APPLICATION = 'myproject.wsgi.application'

# Database (use environment variables)
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql',
        'NAME': os.environ.get('DB_NAME'),
        'USER': os.environ.get('DB_USER'),
        'PASSWORD': os.environ.get('DB_PASSWORD'),
        'HOST': os.environ.get('DB_HOST', 'localhost'),
        'PORT': os.environ.get('DB_PORT', '5432'),
    }
}

# Password validation
AUTH_PASSWORD_VALIDATORS = [
    {'NAME': 'django.contrib.auth.password_validation.UserAttributeSimilarityValidator'},
    {'NAME': 'django.contrib.auth.password_validation.MinimumLengthValidator'},
    {'NAME': 'django.contrib.auth.password_validation.CommonPasswordValidator'},
    {'NAME': 'django.contrib.auth.password_validation.NumericPasswordValidator'},
]

# Internationalization
LANGUAGE_CODE = 'en-us'
TIME_ZONE = 'UTC'
USE_I18N = True
USE_TZ = True

# Static files
STATIC_URL = '/static/'
STATIC_ROOT = BASE_DIR / 'staticfiles'
STATICFILES_DIRS = [BASE_DIR / 'static']

# Media files
MEDIA_URL = '/media/'
MEDIA_ROOT = BASE_DIR / 'media'

# Default primary key field type
DEFAULT_AUTO_FIELD = 'django.db.models.BigAutoField'

# Security settings (production)
if not DEBUG:
    SECURE_SSL_REDIRECT = True
    SESSION_COOKIE_SECURE = True
    CSRF_COOKIE_SECURE = True
    SECURE_HSTS_SECONDS = 31536000
    SECURE_HSTS_INCLUDE_SUBDOMAINS = True
    SECURE_HSTS_PRELOAD = True

# Content Security Policy
SECURE_CSP = {
    "default-src": [CSP.SELF],
    "script-src": [CSP.SELF, CSP.NONCE],
    "style-src": [CSP.SELF, CSP.NONCE],
    "img-src": [CSP.SELF, "https:", "data:"],
}

# Background tasks
TASKS = {
    "default": {
        "BACKEND": "django.tasks.backends.database.DatabaseBackend",
    }
}
```

**Why these settings:**
- Environment variables keep secrets out of code
- CSP provides XSS protection
- Security headers protect against common attacks
- `USE_TZ = True` prevents timezone bugs
- PostgreSQL recommended for production

### requirements.txt

```txt
Django==6.0.1
psycopg2-binary==2.9.9  # PostgreSQL adapter
python-decouple==3.8    # Environment variables
gunicorn==21.2.0        # Production server
whitenoise==6.6.0       # Static file serving
```

---

## Common Patterns

### Pattern 1: Custom User Model

```python
# users/models.py
from django.contrib.auth.models import AbstractUser
from django.db import models

class CustomUser(AbstractUser):
    email = models.EmailField(unique=True)
    bio = models.TextField(blank=True)
    avatar = models.ImageField(upload_to='avatars/', blank=True)

    def __str__(self):
        return self.username
```

```python
# settings.py
AUTH_USER_MODEL = 'users.CustomUser'
```

**When to use**: Start of every project (can't easily change later)

### Pattern 2: Model Managers and QuerySets

```python
# models.py
from django.db import models

class PublishedManager(models.Manager):
    def get_queryset(self):
        return super().get_queryset().filter(is_published=True)

class Article(models.Model):
    title = models.CharField(max_length=200)
    is_published = models.BooleanField(default=False)

    objects = models.Manager()  # Default manager
    published = PublishedManager()  # Custom manager

# Usage
all_articles = Article.objects.all()
published_articles = Article.published.all()
```

### Pattern 3: Signals for Side Effects

```python
# models.py or signals.py
from django.db.models.signals import post_save
from django.dispatch import receiver
from django.contrib.auth import get_user_model

User = get_user_model()

@receiver(post_save, sender=User)
def create_user_profile(sender, instance, created, **kwargs):
    if created:
        # Send welcome email, create profile, etc.
        from .tasks import send_welcome_email
        send_welcome_email.enqueue(user_id=instance.id)
```

```python
# apps.py
from django.apps import AppConfig

class MyappConfig(AppConfig):
    default_auto_field = 'django.db.models.BigAutoField'
    name = 'myapp'

    def ready(self):
        import myapp.signals  # Register signals
```

### Pattern 4: Context Processors

```python
# myapp/context_processors.py
def site_settings(request):
    return {
        'SITE_NAME': 'My Site',
        'CONTACT_EMAIL': 'contact@example.com',
    }
```

```python
# settings.py
TEMPLATES = [
    {
        'OPTIONS': {
            'context_processors': [
                # ...
                'myapp.context_processors.site_settings',
            ],
        },
    },
]
```

### Pattern 5: Custom Management Commands

```python
# myapp/management/commands/seed_data.py
from django.core.management.base import BaseCommand
from myapp.models import Article

class Command(BaseCommand):
    help = 'Seeds database with sample data'

    def add_arguments(self, parser):
        parser.add_argument('--count', type=int, default=10)

    def handle(self, *args, **options):
        count = options['count']
        for i in range(count):
            Article.objects.create(
                title=f'Article {i+1}',
                content='Sample content',
                is_published=True
            )
        self.stdout.write(self.style.SUCCESS(f'Created {count} articles'))
```

```bash
# Run command
python manage.py seed_data --count=20
```

---

## Admin Interface Configuration

```python
# myapp/admin.py
from django.contrib import admin
from .models import Article

@admin.register(Article)
class ArticleAdmin(admin.ModelAdmin):
    list_display = ['title', 'is_published', 'published_date']
    list_filter = ['is_published', 'published_date']
    search_fields = ['title', 'content']
    prepopulated_fields = {'slug': ('title',)}
    date_hierarchy = 'published_date'
    ordering = ['-published_date']

    # Customize form layout
    fieldsets = (
        ('Content', {
            'fields': ('title', 'slug', 'content')
        }),
        ('Publishing', {
            'fields': ('is_published', 'published_date'),
            'classes': ('collapse',)
        }),
    )

    # Custom actions
    actions = ['make_published']

    @admin.action(description='Mark selected articles as published')
    def make_published(self, request, queryset):
        queryset.update(is_published=True)
```

**Admin customization features:**
- `list_display` - Columns to show in list view
- `list_filter` - Sidebar filters
- `search_fields` - Enable search
- `prepopulated_fields` - Auto-fill from other fields
- `date_hierarchy` - Date-based navigation
- Custom actions for bulk operations

---

## Using Bundled Resources

### Templates (templates/)

This skill includes ready-to-use templates:

- `templates/model.py.template` - Model with common fields and Meta options
- `templates/view.py.template` - Function and class-based view examples
- `templates/form.py.template` - ModelForm with validation
- `templates/admin.py.template` - Admin configuration with search and filters

**When to use**: Copy templates when creating new models, views, or forms

### References (references/)

- `references/django-orm-cheatsheet.md` - Quick reference for ORM queries
- `references/template-tags.md` - Common template tags and filters
- `references/deployment-checklist.md` - Production deployment checklist

**When Claude should load these**: When you ask about ORM queries, template syntax, or deployment

---

## Troubleshooting

### Problem: Migrations not detected
**Solution**:
1. Ensure app is in `INSTALLED_APPS`
2. Check `migrations/` directory exists
3. Run `python manage.py makemigrations myapp` (specify app name)

### Problem: Static files not loading in development
**Solution**:
1. Ensure `DEBUG = True`
2. Add `django.contrib.staticfiles` to `INSTALLED_APPS`
3. Use `{% load static %}` in templates
4. Check `STATIC_URL` is set

### Problem: Admin login doesn't work
**Solution**:
1. Create superuser: `python manage.py createsuperuser`
2. Ensure `django.contrib.admin` is in `INSTALLED_APPS`
3. Check `path('admin/', admin.site.urls)` in URLs

### Problem: Template syntax errors
**Solution**:
1. Use `{% load static %}` before `{% static %}`
2. Always close template tags: `{% if %}...{% endif %}`
3. Use `{{ variable }}` for variables, `{% tag %}` for logic

---

## Complete Setup Checklist

Use this checklist to verify your Django project:

- [ ] Virtual environment created and activated
- [ ] Django 6.0.1 installed (`pip install Django==6.0.1`)
- [ ] Project created (`django-admin startproject`)
- [ ] Apps created and added to `INSTALLED_APPS`
- [ ] `SECRET_KEY` using environment variable
- [ ] `DEBUG = False` for production
- [ ] `ALLOWED_HOSTS` configured
- [ ] Database configured (PostgreSQL for production)
- [ ] Models defined with `__str__()` methods
- [ ] Migrations created and applied
- [ ] URL routing configured
- [ ] Views created (function or class-based)
- [ ] Templates in correct directory structure
- [ ] Forms with validation
- [ ] Admin interface configured
- [ ] Static files configuration (`STATIC_ROOT`, `STATIC_URL`)
- [ ] `python manage.py check` runs without errors
- [ ] `python manage.py collectstatic` for production
- [ ] Tests written and passing
- [ ] Security settings enabled (`CSRF`, CSP, HTTPS)
- [ ] `requirements.txt` created

---

## Official Documentation

- **Django 6.0**: https://docs.djangoproject.com/en/6.0/
- **Django Tutorial**: https://docs.djangoproject.com/en/6.0/intro/tutorial01/
- **Django Best Practices**: https://docs.djangoproject.com/en/6.0/misc/design-philosophies/
- **Django Deployment Checklist**: https://docs.djangoproject.com/en/6.0/howto/deployment/checklist/
- **Django Release Notes**: https://docs.djangoproject.com/en/6.0/releases/6.0/

---

## Package Versions (Verified 2026-01-24)

```json
{
  "dependencies": {
    "Django": "6.0.1",
    "psycopg2-binary": "2.9.9",
    "python-decouple": "3.8",
    "gunicorn": "21.2.0",
    "whitenoise": "6.6.0"
  }
}
```

**Python Version**: 3.12+ required for Django 6.0

---

**Questions? Issues?**

1. Check [Django Documentation](https://docs.djangoproject.com/en/6.0/)
2. Review this skill's references in `references/`
3. Run `python manage.py check --deploy` for production readiness
4. Verify all environment variables are set correctly
