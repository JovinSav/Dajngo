# Event-Driven Programming in Django

A simple way to run code when something happens (an “event”) in your app.

## 1. How It Works
- Django emits **signals** on certain events (save, delete, etc.).  
- You write a **receiver** function that listens and reacts.

## 2. Why Use It?
- Keeps logic separate and reusable.  
- Automatically trigger emails, logs, or other actions.

## 3. Example: Welcome Email on User Signup
```python
from django.db.models.signals import post_save
from django.dispatch import receiver
from django.contrib.auth.models import User
from django.core.mail import send_mail

@receiver(post_save, sender=User)
def send_welcome_email(sender, instance, created, **kwargs):
    if created:
        send_mail(
            "Welcome!",
            f"Hi {instance.username}, thanks for joining.",
            "from@example.com",
            [instance.email],
        )
```

## 4. Example: Log Price Changes
```python
from django.db.models.signals import pre_save
from django.dispatch import receiver
import logging
from .models import Product

logger = logging.getLogger(__name__)

@receiver(pre_save, sender=Product)
def log_price_change(sender, instance, **kwargs):
    if instance.pk:
        old = Product.objects.get(pk=instance.pk)
        if old.price != instance.price:
            logger.info(f"Product {instance.name} price: {old.price} → {instance.price}")
```

## 5. Running Long Tasks
For heavy work (e.g. sending many emails), use **Celery**:
```python
# tasks.py
from celery import shared_task

@shared_task
def send_report(user_id):
    # generate and send report
    ...
```
Call from signal:
```python
send_report.delay(instance.user.id)
```

## 6. Real-Time Updates
Use **WebSockets** with **Django Channels** to push data instantly:
```python
# consumers.py
class Notifier(AsyncJsonWebsocketConsumer):
    async def connect(self):
        await self.accept()
    async def send_update(self, event):
        await self.send_json(event["data"])
```
In React, open a WebSocket and listen for messages.

---

Start by placing your receivers in `signals.py`, importing them in `apps.py`, and testing each event. Happy coding!
