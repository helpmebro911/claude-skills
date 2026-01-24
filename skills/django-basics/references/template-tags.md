# Django Template Tags & Filters Quick Reference

**Quick lookup for common Django template syntax**

---

## Template Basics

```django
{# This is a comment #}
{# Comments are not rendered in HTML #}

{# Multi-line comment #}
{% comment %}
This is a
multi-line comment
{% endcomment %}

{# Variables #}
{{ variable }}
{{ user.username }}
{{ article.author.email }}

{# Template tags #}
{% tag_name %}
```

---

## Built-in Tags

### for Loop

```django
{% for item in items %}
    <p>{{ item.name }}</p>
{% endfor %}

{# Empty case #}
{% for item in items %}
    <p>{{ item.name }}</p>
{% empty %}
    <p>No items found.</p>
{% endfor %}

{# Loop variables #}
{% for item in items %}
    {{ forloop.counter }}      {# 1-indexed counter #}
    {{ forloop.counter0 }}     {# 0-indexed counter #}
    {{ forloop.revcounter }}   {# Reverse counter #}
    {{ forloop.first }}        {# True on first iteration #}
    {{ forloop.last }}         {# True on last iteration #}
    {{ forloop.parentloop }}   {# Parent loop in nested loops #}
    {{ forloop.length }}       {# NEW in Django 6.0 #}
{% endfor %}

{# Nested loops #}
{% for category in categories %}
    {% for item in category.items %}
        {{ forloop.parentloop.counter }}.{{ forloop.counter }}
    {% endfor %}
{% endfor %}
```

### if Conditions

```django
{% if user.is_authenticated %}
    <p>Welcome, {{ user.username }}!</p>
{% endif %}

{% if condition %}
    ...
{% else %}
    ...
{% endif %}

{% if condition %}
    ...
{% elif other_condition %}
    ...
{% else %}
    ...
{% endif %}

{# Operators #}
{% if age >= 18 %}
{% if name == "John" %}
{% if age != 30 %}
{% if age > 18 and age < 65 %}
{% if user.is_staff or user.is_superuser %}
{% if not user.is_anonymous %}
{% if var in list %}
{% if var not in list %}

{# Comparisons #}
{% if somevar == "x" %}
{% if somevar != "x" %}
{% if somevar < 100 %}
{% if somevar > 0 %}
{% if somevar <= 100 %}
{% if somevar >= 1 %}
```

### url - URL Reversing

```django
{# Basic URL #}
<a href="{% url 'home' %}">Home</a>

{# With positional argument #}
<a href="{% url 'article_detail' article.id %}">View</a>

{# With keyword arguments #}
<a href="{% url 'article_detail' pk=article.id %}">View</a>

{# With namespace #}
<a href="{% url 'blog:article_detail' article.id %}">View</a>

{# Assign to variable #}
{% url 'home' as home_url %}
<a href="{{ home_url }}">Home</a>

{# With query parameters (use template filter) #}
<a href="{% url 'search' %}?q={{ query }}">Search</a>
```

### block & extends - Template Inheritance

```django
{# base.html #}
<!DOCTYPE html>
<html>
<head>
    <title>{% block title %}My Site{% endblock %}</title>
</head>
<body>
    {% block content %}{% endblock %}
</body>
</html>

{# child.html #}
{% extends "base.html" %}

{% block title %}Page Title{% endblock %}

{% block content %}
    <h1>Content here</h1>
{% endblock %}

{# Get parent content #}
{% block content %}
    {{ block.super }}  {# Include parent block content #}
    <p>Additional content</p>
{% endblock %}
```

### include - Include Templates

```django
{# Include another template #}
{% include "components/header.html" %}

{# With context variables #}
{% include "components/card.html" with title="My Title" %}

{# Only with specified variables (no access to parent context) #}
{% include "components/card.html" with title="My Title" only %}

{# Django 6.0: Include partial #}
{% include "myapp/template.html#partial-name" %}
```

### load - Load Template Tags

```django
{# Load static files tags #}
{% load static %}
<link rel="stylesheet" href="{% static 'css/style.css' %}">

{# Load custom template tags #}
{% load my_custom_tags %}

{# Load multiple #}
{% load static i18n %}

{# Django 6.0: Load CSP #}
{% load csp %}
```

### static - Static Files

```django
{% load static %}

{# CSS #}
<link rel="stylesheet" href="{% static 'css/style.css' %}">

{# JavaScript #}
<script src="{% static 'js/main.js' %}"></script>

{# Images #}
<img src="{% static 'images/logo.png' %}" alt="Logo">

{# Assign to variable #}
{% static 'images/logo.png' as logo_url %}
<img src="{{ logo_url }}" alt="Logo">
```

### csrf_token - CSRF Protection

```django
<form method="post">
    {% csrf_token %}  {# Always include in POST forms! #}
    ...
</form>
```

### with - Create Local Variables

```django
{% with total=business.employees.count %}
    {{ total }} employee{{ total|pluralize }}
{% endwith %}

{# Multiple variables #}
{% with alpha=1 beta=2 %}
    {{ alpha }} and {{ beta }}
{% endwith %}
```

### now - Current Date/Time

```django
{% now "Y-m-d H:i:s" %}
{# Output: 2024-01-15 14:30:00 #}

{% now "jS F Y" %}
{# Output: 15th January 2024 #}
```

### Django 6.0: Template Partials

```django
{# Define partial #}
{% partialdef card-component %}
    <div class="card">
        <h3>{{ title }}</h3>
        <p>{{ content }}</p>
    </div>
{% endpartialdef %}

{# Use partial #}
{% partial card-component with title="My Title" content="Content here" %}

{# Include partial from another template #}
{% include "myapp/components.html#card-component" with title="Title" %}
```

---

## Built-in Filters

### String Filters

```django
{# Length #}
{{ value|length }}

{# Lower/Upper case #}
{{ name|lower }}
{{ name|upper }}
{{ name|title }}       {# Title Case #}
{{ name|capfirst }}    {# Capitalize first letter #}

{# Truncate #}
{{ text|truncatewords:30 }}      {# Truncate to 30 words #}
{{ text|truncatechars:100 }}     {# Truncate to 100 chars #}

{# Default if empty #}
{{ value|default:"Nothing" }}

{# Default if false (includes 0, False, None, "") #}
{{ value|default_if_none:"N/A" }}

{# Safe (mark as HTML-safe) #}
{{ html_content|safe }}          {# Use carefully! XSS risk #}

{# Escape HTML #}
{{ user_input|escape }}

{# Line breaks to <br> #}
{{ text|linebreaks }}
{{ text|linebreaksbr }}

{# Slugify #}
{{ title|slugify }}

{# Add slashes (escape quotes) #}
{{ text|addslashes }}

{# Join list #}
{{ list|join:", " }}

{# First/Last #}
{{ list|first }}
{{ list|last }}

{# Random from list #}
{{ list|random }}

{# Slice #}
{{ list|slice:":5" }}  {# First 5 items #}
```

### Number Filters

```django
{# Add #}
{{ value|add:5 }}

{# Floatformat #}
{{ value|floatformat }}      {# Round to 1 decimal #}
{{ value|floatformat:2 }}    {# Round to 2 decimals #}

{# Pluralize #}
{{ count }} item{{ count|pluralize }}
{{ count }} categor{{ count|pluralize:"y,ies" }}

{# Get digit #}
{{ value|get_digit:1 }}  {# Get specific digit #}
```

### Date/Time Filters

```django
{# Format date #}
{{ date|date:"Y-m-d" }}
{{ date|date:"F j, Y" }}           {# January 15, 2024 #}
{{ date|date:"l, F jS, Y" }}       {# Monday, January 15th, 2024 #}

{# Format time #}
{{ time|time:"H:i:s" }}

{# Time since #}
{{ date|timesince }}               {# "2 days, 3 hours" #}
{{ date|timeuntil }}               {# "in 2 days, 3 hours" #}

{# Common date formats #}
{{ date|date:"SHORT_DATE_FORMAT" }}
{{ date|date:"SHORT_DATETIME_FORMAT" }}
```

### List/Dict Filters

```django
{# Dictionary item #}
{{ dict|get_item:key }}

{# Length #}
{{ list|length }}

{# Dictionary keys/values #}
{{ dict.items }}
{{ dict.keys }}
{{ dict.values }}
```

### URL Filters

```django
{# URL encode #}
{{ value|urlencode }}

{# Linkify URLs in text #}
{{ text|urlize }}
{{ text|urlizetrunc:30 }}  {# Truncate URLs to 30 chars #}
```

### Chaining Filters

```django
{# Filters can be chained #}
{{ text|lower|truncatewords:10 }}
{{ name|default:"Anonymous"|title }}
{{ list|join:", "|lower }}
```

---

## Forms in Templates

```django
{# Render entire form #}
<form method="post">
    {% csrf_token %}
    {{ form.as_p }}          {# As paragraphs #}
    {{ form.as_table }}      {# As table rows #}
    {{ form.as_ul }}         {# As list items #}
    <button type="submit">Submit</button>
</form>

{# Render individual fields #}
<form method="post">
    {% csrf_token %}

    {# Field with all elements #}
    {{ form.field_name.label_tag }}
    {{ form.field_name }}
    {{ form.field_name.errors }}
    {{ form.field_name.help_text }}

    <button type="submit">Submit</button>
</form>

{# Manual field rendering #}
<form method="post">
    {% csrf_token %}

    {% for field in form %}
        <div class="form-group">
            {{ field.label_tag }}
            {{ field }}
            {% if field.errors %}
                <div class="error">{{ field.errors }}</div>
            {% endif %}
            {% if field.help_text %}
                <small>{{ field.help_text }}</small>
            {% endif %}
        </div>
    {% endfor %}

    <button type="submit">Submit</button>
</form>

{# Check field type #}
{% if field.field.widget.input_type == 'checkbox' %}
    {# Special handling for checkboxes #}
{% endif %}

{# Non-field errors #}
{% if form.non_field_errors %}
    <div class="alert">{{ form.non_field_errors }}</div>
{% endif %}
```

---

## Pagination

```django
{# Basic pagination #}
{% if is_paginated %}
<nav>
    {% if page_obj.has_previous %}
        <a href="?page=1">First</a>
        <a href="?page={{ page_obj.previous_page_number }}">Previous</a>
    {% endif %}

    <span>
        Page {{ page_obj.number }} of {{ page_obj.paginator.num_pages }}
    </span>

    {% if page_obj.has_next %}
        <a href="?page={{ page_obj.next_page_number }}">Next</a>
        <a href="?page={{ page_obj.paginator.num_pages }}">Last</a>
    {% endif %}
</nav>
{% endif %}

{# Page range #}
{% for num in page_obj.paginator.page_range %}
    {% if page_obj.number == num %}
        <span class="current">{{ num }}</span>
    {% else %}
        <a href="?page={{ num }}">{{ num }}</a>
    {% endif %}
{% endfor %}
```

---

## Internationalization (i18n)

```django
{% load i18n %}

{# Translate text #}
{% trans "Hello" %}
{% trans "Welcome to our site" %}

{# Translate with variable #}
{% blocktrans %}Hello {{ username }}{% endblocktrans %}

{# Plurals #}
{% blocktrans count counter=list|length %}
    There is {{ counter }} item.
{% plural %}
    There are {{ counter }} items.
{% endblocktrans %}

{# Language selection #}
{% get_current_language as LANGUAGE_CODE %}
{% get_available_languages as LANGUAGES %}
```

---

## Autoescape

```django
{# Disable autoescape (use with caution!) #}
{% autoescape off %}
    {{ html_content }}
{% endautoescape %}

{# Enable autoescape #}
{% autoescape on %}
    {{ user_input }}
{% endautoescape %}
```

---

## Custom Template Tags Usage

```django
{# After creating custom tags in templatetags/my_tags.py #}
{% load my_tags %}

{# Simple tag #}
{% my_custom_tag arg1 arg2 %}

{# Inclusion tag #}
{% show_results poll %}

{# Assignment tag #}
{% get_current_time "%Y-%m-%d %I:%M %p" as the_time %}
<p>The time is {{ the_time }}.</p>
```

---

## Debugging

```django
{# Debug info (only works when DEBUG=True) #}
{{ debug }}

{# Print variable type #}
{{ variable|pprint }}

{# Check if variable exists #}
{% if variable %}
    {{ variable }}
{% endif %}

{# Show all context variables (DEBUG=True) #}
{% debug %}
```

---

## Common Patterns

```django
{# Navigation with active state #}
<nav>
    <a href="{% url 'home' %}" {% if request.resolver_match.url_name == 'home' %}class="active"{% endif %}>
        Home
    </a>
</nav>

{# Conditional CSS class #}
<div class="{% if user.is_authenticated %}logged-in{% else %}logged-out{% endif %}">
    ...
</div>

{# Flash messages #}
{% if messages %}
    {% for message in messages %}
        <div class="alert alert-{{ message.tags }}">
            {{ message }}
        </div>
    {% endfor %}
{% endif %}

{# Breadcrumbs #}
<nav aria-label="breadcrumb">
    <ol class="breadcrumb">
        <li class="breadcrumb-item"><a href="{% url 'home' %}">Home</a></li>
        <li class="breadcrumb-item active">{{ article.title }}</li>
    </ol>
</nav>

{# Empty state #}
{% if items %}
    {% for item in items %}
        {{ item.name }}
    {% endfor %}
{% else %}
    <p>No items found.</p>
{% endif %}
```

---

## Best Practices

1. **Always use {% csrf_token %}** in POST forms
2. **Use {% url %}** instead of hardcoded URLs
3. **Use {% static %}** for static files
4. **Escape user input** (enabled by default)
5. **Use |safe only for trusted HTML** (XSS risk)
6. **Keep logic in views**, not templates
7. **Use template inheritance** (DRY principle)
8. **Use {% load %}** at the top of templates
9. **Comment your templates** with {# #}
10. **Cache expensive template fragments** using {% cache %}
