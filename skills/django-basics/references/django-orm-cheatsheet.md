# Django ORM Quick Reference

**Quick lookup for common Django ORM queries and patterns**

---

## Basic Queries

```python
# Get all objects
Model.objects.all()

# Get single object (raises DoesNotExist if not found)
Model.objects.get(id=1)
Model.objects.get(name='example')

# Get or 404 (in views)
from django.shortcuts import get_object_or_404
obj = get_object_or_404(Model, id=1)

# Get first/last
Model.objects.first()
Model.objects.last()

# Count
Model.objects.count()

# Check existence
Model.objects.filter(name='example').exists()
```

---

## Filtering

```python
# Basic filter
Model.objects.filter(is_active=True)
Model.objects.filter(name='example', status='published')

# Exclude
Model.objects.exclude(is_deleted=True)

# Chaining
Model.objects.filter(is_active=True).exclude(status='draft')

# Field lookups
Model.objects.filter(name__exact='example')      # Exact match
Model.objects.filter(name__iexact='example')     # Case-insensitive exact
Model.objects.filter(name__contains='exam')      # Contains
Model.objects.filter(name__icontains='exam')     # Case-insensitive contains
Model.objects.filter(name__startswith='ex')      # Starts with
Model.objects.filter(name__endswith='le')        # Ends with
Model.objects.filter(age__gt=18)                 # Greater than
Model.objects.filter(age__gte=18)                # Greater than or equal
Model.objects.filter(age__lt=65)                 # Less than
Model.objects.filter(age__lte=65)                # Less than or equal
Model.objects.filter(date__year=2024)            # Year
Model.objects.filter(date__month=1)              # Month
Model.objects.filter(tags__in=['python', 'django'])  # In list
Model.objects.filter(name__isnull=True)          # Is NULL
```

---

## Complex Queries (Q Objects)

```python
from django.db.models import Q

# OR condition
Model.objects.filter(Q(name='example') | Q(status='active'))

# AND condition (explicit)
Model.objects.filter(Q(name='example') & Q(status='active'))

# NOT condition
Model.objects.filter(~Q(status='deleted'))

# Complex combinations
Model.objects.filter(
    (Q(name__icontains='django') | Q(description__icontains='django')) &
    Q(is_published=True)
)
```

---

## Ordering

```python
# Ascending
Model.objects.order_by('name')
Model.objects.order_by('created_at')

# Descending
Model.objects.order_by('-created_at')

# Multiple fields
Model.objects.order_by('-is_featured', '-created_at')

# Random order
Model.objects.order_by('?')
```

---

## Slicing & Limiting

```python
# First 5 objects
Model.objects.all()[:5]

# Objects 5-10
Model.objects.all()[5:10]

# Limit to 10
Model.objects.all()[:10]

# Get one (returns object, not queryset)
Model.objects.all()[0]  # Can raise IndexError
Model.objects.first()    # Returns None if empty (safer)
```

---

## Aggregation

```python
from django.db.models import Count, Sum, Avg, Max, Min

# Count all
Model.objects.count()

# Aggregate
Model.objects.aggregate(
    total=Count('id'),
    average_price=Avg('price'),
    max_price=Max('price'),
    min_price=Min('price'),
    total_sales=Sum('sales')
)
# Returns: {'total': 100, 'average_price': 50.0, ...}

# Annotate (add computed field to each object)
Model.objects.annotate(
    num_comments=Count('comments')
)

# Combined
Model.objects.annotate(
    num_comments=Count('comments')
).filter(num_comments__gt=10)
```

---

## Related Objects (Relationships)

```python
# ForeignKey - Access related object
article.author  # Get the User object
article.author.username

# Reverse ForeignKey - Access related set
author.article_set.all()  # All articles by this author
author.articles.all()      # If related_name='articles'

# ManyToMany
article.tags.all()         # All tags for this article
tag.articles.all()         # All articles with this tag (if related_name='articles')

# Add/Remove M2M
article.tags.add(tag1, tag2)
article.tags.remove(tag1)
article.tags.clear()  # Remove all
article.tags.set([tag1, tag2])  # Replace all
```

---

## Query Optimization

```python
# select_related (for ForeignKey, OneToOne)
# Performs SQL JOIN - single query
Article.objects.select_related('author').all()

# prefetch_related (for ManyToMany, reverse ForeignKey)
# Performs separate query and joins in Python
Article.objects.prefetch_related('tags').all()

# Combine both
Article.objects.select_related('author').prefetch_related('tags').all()

# only() - Load only specific fields
Article.objects.only('title', 'created_at')

# defer() - Load all fields except specified
Article.objects.defer('content')

# values() - Return dictionaries instead of model instances
Article.objects.values('title', 'author__username')

# values_list() - Return tuples
Article.objects.values_list('title', 'created_at')
Article.objects.values_list('title', flat=True)  # Flat list of titles
```

---

## Creating Objects

```python
# Method 1: Create and save
obj = Model(name='example', status='active')
obj.save()

# Method 2: Create in one step
obj = Model.objects.create(name='example', status='active')

# Method 3: Get or create
obj, created = Model.objects.get_or_create(
    name='example',
    defaults={'status': 'active'}
)
# created = True if new, False if existing

# Method 4: Update or create
obj, created = Model.objects.update_or_create(
    name='example',
    defaults={'status': 'active'}
)

# Bulk create (efficient for many objects)
Model.objects.bulk_create([
    Model(name='obj1'),
    Model(name='obj2'),
    Model(name='obj3'),
])
```

---

## Updating Objects

```python
# Update single object
obj = Model.objects.get(id=1)
obj.name = 'New Name'
obj.save()

# Update queryset (no signals triggered)
Model.objects.filter(status='draft').update(status='published')

# Update or create
obj, created = Model.objects.update_or_create(
    id=1,
    defaults={'name': 'New Name'}
)

# Bulk update (Django 2.2+)
objs = Model.objects.all()
for obj in objs:
    obj.status = 'active'
Model.objects.bulk_update(objs, ['status'])

# F expressions (for atomic updates)
from django.db.models import F
Model.objects.filter(id=1).update(views=F('views') + 1)
```

---

## Deleting Objects

```python
# Delete single object
obj = Model.objects.get(id=1)
obj.delete()

# Delete queryset
Model.objects.filter(status='deleted').delete()

# Soft delete (set flag instead of deleting)
Model.objects.filter(id=1).update(is_deleted=True, deleted_at=timezone.now())
```

---

## Dates & Times

```python
from django.utils import timezone
from datetime import timedelta

# Current time
timezone.now()

# Filter by date
Model.objects.filter(created_at__date=timezone.now().date())

# Date range
start_date = timezone.now() - timedelta(days=7)
Model.objects.filter(created_at__gte=start_date)

# Date components
Model.objects.filter(created_at__year=2024)
Model.objects.filter(created_at__month=1)
Model.objects.filter(created_at__day=15)
Model.objects.filter(created_at__week_day=1)  # 1=Sunday

# Date comparison
Model.objects.filter(created_at__lt=timezone.now())
Model.objects.filter(created_at__range=(start_date, end_date))
```

---

## Raw SQL

```python
# Raw queries (use sparingly)
Model.objects.raw('SELECT * FROM myapp_model WHERE id = %s', [1])

# Execute raw SQL
from django.db import connection
with connection.cursor() as cursor:
    cursor.execute("SELECT * FROM myapp_model WHERE id = %s", [1])
    row = cursor.fetchone()
```

---

## Transactions

```python
from django.db import transaction

# Atomic block
with transaction.atomic():
    obj1 = Model.objects.create(name='obj1')
    obj2 = Model.objects.create(name='obj2')
    # If any error occurs, both are rolled back

# Atomic decorator
@transaction.atomic
def create_objects():
    Model.objects.create(name='obj1')
    Model.objects.create(name='obj2')
```

---

## Common Patterns

```python
# Get or None (instead of try/except)
obj = Model.objects.filter(id=1).first()  # Returns None if not found

# Latest/Earliest
Model.objects.latest('created_at')
Model.objects.earliest('created_at')

# Random object
Model.objects.order_by('?').first()

# Distinct values
Model.objects.values('status').distinct()

# Union, intersection, difference (Django 1.11+)
qs1 = Model.objects.filter(status='active')
qs2 = Model.objects.filter(is_featured=True)
qs1.union(qs2)
qs1.intersection(qs2)
qs1.difference(qs2)

# Check if queryset is empty
if Model.objects.filter(status='active').exists():
    # More efficient than count() > 0
    pass

# Iterator for large querysets (memory efficient)
for obj in Model.objects.all().iterator():
    # Process obj
    pass
```

---

## Performance Tips

1. **Use select_related() and prefetch_related()** to avoid N+1 queries
2. **Use only() and defer()** to load only needed fields
3. **Use exists()** instead of `count() > 0`
4. **Use iterator()** for large querysets
5. **Avoid queries in loops** - use select_related/prefetch_related
6. **Use bulk_create/bulk_update** for multiple objects
7. **Use F() expressions** for atomic database operations
8. **Index frequently queried fields** in model Meta
9. **Use django-debug-toolbar** to monitor queries in development
10. **Cache expensive queries** using Django's cache framework

---

## Debugging

```python
# Print SQL query
queryset = Model.objects.filter(name='example')
print(queryset.query)

# Count queries (use django-debug-toolbar in dev)
from django.db import connection
print(len(connection.queries))
print(connection.queries)
```
