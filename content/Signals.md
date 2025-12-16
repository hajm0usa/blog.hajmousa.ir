+++
title = "A complete guide on django signals"
date = 2025-12-17
+++
# Django Signals: Complete Guide

Signals are Django's way of allowing decoupled applications to get notified when certain actions occur elsewhere in the framework. Think of them as **event listeners** or **hooks**.

## Core Concept

```
Action happens → Signal sent → Receivers execute
```

It's like a notification system: "Hey, something just happened, anyone who cares should do something about it!"

## Built-in Django Signals

### Model Signals (Most Common)

```python
from django.db.models.signals import (
    pre_save,      # Before model.save()
    post_save,     # After model.save()
    pre_delete,    # Before model.delete()
    post_delete,   # After model.delete()
    m2m_changed,   # When ManyToMany relationship changes
)

# Other signals
from django.db.models.signals import (
    pre_init,      # Before model.__init__()
    post_init,     # After model.__init__()
)
```

### Request/Response Signals

```python
from django.core.signals import (
    request_started,   # When HTTP request starts
    request_finished,  # When HTTP request finishes
)
```

### Database Signals

```python
from django.db.backends.signals import connection_created
```

## How to Use Signals

### Method 1: Using Decorators (Recommended)

```python
from django.db.models.signals import post_save
from django.dispatch import receiver
from django.contrib.auth.models import User

@receiver(post_save, sender=User)
def create_user_profile(sender, instance, created, **kwargs):
    """
    Automatically create a UserProfile when a User is created
    """
    if created:  # Only on creation, not updates
        UserProfile.objects.create(user=instance)
```

### Method 2: Manual Connection

```python
from django.db.models.signals import post_save

def create_user_profile(sender, instance, created, **kwargs):
    if created:
        UserProfile.objects.create(user=instance)

# Connect the signal
post_save.connect(create_user_profile, sender=User)
```

## Signal Parameters Explained

### `post_save` Signal

```python
@receiver(post_save, sender=Book)
def book_saved(sender, instance, created, raw, using, update_fields, **kwargs):
    """
    sender: The model class (Book)
    instance: The actual instance being saved
    created: Boolean - True if new object, False if update
    raw: Boolean - True if loading fixtures (skip processing)
    using: Database alias being used
    update_fields: Set of fields being updated (or None)
    **kwargs: Future compatibility
    """
    pass
```

### `pre_save` Signal

```python
@receiver(pre_save, sender=Book)
def book_pre_save(sender, instance, raw, using, update_fields, **kwargs):
    """
    Same params as post_save, but NO 'created' parameter
    (object hasn't been saved yet, so we don't know if it's new)
    """
    # You can modify instance here before it's saved
    instance.slug = slugify(instance.title)
```

### `post_delete` Signal

```python
@receiver(post_delete, sender=Book)
def book_deleted(sender, instance, using, **kwargs):
    """
    sender: The model class
    instance: The instance that was deleted (still in memory)
    using: Database alias
    """
    # Clean up related files
    if instance.cover_image:
        instance.cover_image.delete(save=False)
```

### `m2m_changed` Signal

```python
@receiver(m2m_changed, sender=Book.authors.through)
def authors_changed(sender, instance, action, reverse, model, pk_set, **kwargs):
    """
    action: 'pre_add', 'post_add', 'pre_remove', 'post_remove', 'pre_clear', 'post_clear'
    instance: The instance being modified
    reverse: Direction of the relation
    model: The related model class
    pk_set: Set of primary keys being added/removed
    """
    if action == "post_add":
        print(f"Added authors {pk_set} to book {instance.title}")
```

## Real-World Examples for Your Bookstore

### 1. Send Email When Review is Posted

```python
from django.db.models.signals import post_save
from django.dispatch import receiver
from django.core.mail import send_mail
from .models import Review

@receiver(post_save, sender=Review)
def notify_author_of_review(sender, instance, created, **kwargs):
    if created and instance.is_approved:
        book = instance.book
        send_mail(
            subject=f'New review for "{book.title}"',
            message=f'{instance.user.username} rated your book {instance.rating}/5',
            from_email='noreply@bookstore.com',
            recipient_list=['author@example.com'],
            fail_silently=True,
        )
```

### 2. Update Book Stats When Rating Changes

```python
from django.db.models.signals import post_save, post_delete
from django.dispatch import receiver
from django.db.models import Avg, Count
from .models import Rating

@receiver(post_save, sender=Rating)
@receiver(post_delete, sender=Rating)
def update_book_rating_stats(sender, instance, **kwargs):
    """
    Recalculate book's average rating and count
    Works for both new ratings and deleted ratings
    """
    book = instance.book
    stats = Rating.objects.filter(book=book).aggregate(
        avg_rating=Avg('rating_value'),
        total_ratings=Count('id')
    )
    
    book.average_rating = stats['avg_rating'] or 0
    book.total_ratings = stats['total_ratings']
    book.save(update_fields=['average_rating', 'total_ratings'])
```

### 3. Create Activity Log

```python
from django.db.models.signals import post_save, post_delete
from django.dispatch import receiver
from .models import Book, ActivityLog

@receiver(post_save, sender=Book)
def log_book_activity(sender, instance, created, **kwargs):
    action = 'created' if created else 'updated'
    ActivityLog.objects.create(
        action=action,
        model_name='Book',
        object_id=instance.id,
        description=f'Book "{instance.title}" was {action}'
    )

@receiver(post_delete, sender=Book)
def log_book_deletion(sender, instance, **kwargs):
    ActivityLog.objects.create(
        action='deleted',
        model_name='Book',
        object_id=instance.id,
        description=f'Book "{instance.title}" was deleted'
    )
```

### 4. Inventory Management

```python
from django.db.models.signals import post_save
from django.dispatch import receiver
from .models import Purchase, Book

@receiver(post_save, sender=Purchase)
def update_inventory(sender, instance, created, **kwargs):
    if created:
        book = instance.book
        book.stock_quantity = F('stock_quantity') - instance.quantity
        book.save(update_fields=['stock_quantity'])
        
        # Send alert if stock is low
        book.refresh_from_db()
        if book.stock_quantity < 5:
            # Send notification to staff
            send_low_stock_alert(book)
```

### 5. Generate Slug Automatically

```python
from django.db.models.signals import pre_save
from django.dispatch import receiver
from django.utils.text import slugify
from .models import Book

@receiver(pre_save, sender=Book)
def generate_slug(sender, instance, **kwargs):
    if not instance.slug:
        instance.slug = slugify(instance.title)
        
        # Ensure uniqueness
        original_slug = instance.slug
        counter = 1
        while Book.objects.filter(slug=instance.slug).exists():
            instance.slug = f'{original_slug}-{counter}'
            counter += 1
```

## Where to Put Signal Code?

### Option 1: In `signals.py` (Recommended for large projects)

```
bookstore/
├── models.py
├── signals.py  ← Create this
├── apps.py
└── views.py
```

**signals.py:**

```python
from django.db.models.signals import post_save
from django.dispatch import receiver
from .models import Review

@receiver(post_save, sender=Review)
def handle_review_save(sender, instance, created, **kwargs):
    # Your logic here
    pass
```

**apps.py:**

```python
from django.apps import AppConfig

class BookstoreConfig(AppConfig):
    default_auto_field = 'django.db.models.BigAutoField'
    name = 'bookstore'

    def ready(self):
        import bookstore.signals  # Import signals here
```

### Option 2: At bottom of `models.py` (OK for small projects)

```python
# models.py

class Review(models.Model):
    # ... fields ...
    pass

# Signals at the bottom
@receiver(post_save, sender=Review)
def handle_review(sender, instance, created, **kwargs):
    pass
```

## Common Pitfalls & Best Practices

### ❌ Pitfall 1: Infinite Loops

```python
# BAD - Creates infinite loop!
@receiver(post_save, sender=Book)
def update_book(sender, instance, **kwargs):
    instance.updated_at = timezone.now()
    instance.save()  # This triggers post_save again! 💥
```

**Solution:**

```python
# GOOD
@receiver(post_save, sender=Book)
def update_book(sender, instance, **kwargs):
    if not instance.updated_at:
        instance.updated_at = timezone.now()
        instance.save(update_fields=['updated_at'])  # Specific fields only
```

### ❌ Pitfall 2: Ignoring `raw` Parameter

```python
# BAD - Runs during fixture loading
@receiver(post_save, sender=Book)
def send_notification(sender, instance, created, **kwargs):
    send_email(f"New book: {instance.title}")  # Sends during loaddata! 😱
```

**Solution:**

```python
# GOOD
@receiver(post_save, sender=Book)
def send_notification(sender, instance, created, raw, **kwargs):
    if raw:
        return  # Skip during fixture loading
    if created:
        send_email(f"New book: {instance.title}")
```

### ❌ Pitfall 3: Heavy Processing in Signals

```python
# BAD - Slows down every save
@receiver(post_save, sender=Review)
def process_review(sender, instance, created, **kwargs):
    # This blocks the request!
    analyze_sentiment(instance.review_text)  # Takes 5 seconds
    generate_summary(instance.review_text)   # Takes 3 seconds
```

**Solution: Use Celery for async tasks**

```python
# GOOD
from .tasks import process_review_async

@receiver(post_save, sender=Review)
def queue_review_processing(sender, instance, created, **kwargs):
    if created:
        process_review_async.delay(instance.id)  # Runs in background
```

### ✅ Best Practice: Conditional Execution

```python
@receiver(post_save, sender=Review)
def handle_review(sender, instance, created, update_fields, **kwargs):
    # Only run if specific fields changed
    if update_fields and 'is_approved' in update_fields:
        if instance.is_approved:
            notify_user_review_approved(instance)
```

## Disconnecting Signals (For Testing)

```python
from django.test import TestCase
from django.db.models.signals import post_save
from .models import Book
from .signals import update_book_stats

class BookTestCase(TestCase):
    def test_book_creation_without_signals(self):
        # Temporarily disconnect
        post_save.disconnect(update_book_stats, sender=Book)
        
        book = Book.objects.create(title="Test")
        # Signal won't fire
        
        # Reconnect
        post_save.connect(update_book_stats, sender=Book)
```

## Custom Signals

You can create your own signals:

```python
# signals.py
from django.dispatch import Signal

# Define custom signal
book_purchased = Signal()  # Providing args is optional but recommended

# Send the signal
from .signals import book_purchased

def complete_purchase(user, book, quantity):
    # ... purchase logic ...
    book_purchased.send(
        sender=Purchase,
        user=user,
        book=book,
        quantity=quantity
    )

# Receive the signal
@receiver(book_purchased)
def handle_book_purchase(sender, user, book, quantity, **kwargs):
    # Award loyalty points
    user.loyalty_points += quantity * 10
    user.save()
```

## When to Use Signals vs Other Approaches

### Use Signals When:

- ✅ Decoupling apps (one app shouldn't know about another)
- ✅ Multiple actions need to happen after an event
- ✅ Third-party packages need to react to your models
- ✅ Creating audit trails or logging

### Don't Use Signals When:

- ❌ Simple logic that belongs in `save()` method
- ❌ Complex business logic (use services/managers instead)
- ❌ When debugging is important (signals make flow harder to trace)
- ❌ Performance-critical code (signals add overhead)

### Example - When NOT to use signals:

```python
# BAD - Overly complex
@receiver(post_save, sender=Order)
def process_order(sender, instance, created, **kwargs):
    if created:
        validate_payment()
        update_inventory()
        send_confirmation()
        create_shipping_label()

# GOOD - Explicit service
class OrderService:
    def process_order(self, order):
        self.validate_payment(order)
        self.update_inventory(order)
        self.send_confirmation(order)
        self.create_shipping_label(order)

# In your view
service = OrderService()
service.process_order(order)
```

## Summary

**Signals are powerful but use them wisely!** They're great for:

- Keeping code decoupled
- Reacting to model changes across different apps
- Creating plugins or extensible systems

But they can make your code harder to follow and debug. When in doubt, start with simple methods and only reach for signals when you need the decoupling.

Would you like to see how to implement any specific signal pattern for your bookstore rating system?
