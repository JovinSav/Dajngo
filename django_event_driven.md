# Event-Driven Logic in Django

## Navigation
- [Overview](#overview)
- [Django Signals](#django-signals)
- [Example: Duplicate-Entry Detection](#example-duplicate-entry-detection)
- [Sending Notifications](#sending-notifications)
- [Async Tasks with Celery](#async-tasks-with-celery)
- [Useful Libraries](#useful-libraries)

## Overview
Implement “reactive” behavior—run code when certain events occur.  
Common use cases:
- Prevent or alert on duplicate entries  
- Log audit trails  
- Send emails or push notifications on state changes  

## Django Signals
Django’s built-in pub/sub:
- **pre_save** / **post_save** / **pre_delete** / **post_delete**  
- **m2m_changed** for many-to-many  
- Custom signals via `django.dispatch.Signal`

Define a receiver:
```python
from django.db.models.signals import pre_save
from django.dispatch import receiver
from .models import Order

@receiver(pre_save, sender=Order)
def check_duplicate_order(sender, instance, **kwargs):
    if Order.objects.filter(user=instance.user, item=instance.item).exists():
        # raise, log or mark instance.duplicate = True
        instance.duplicate = True
```

## Example: Duplicate-Entry Detection
1. Add a boolean field to flag duplicates:
   ```python
   class Order(models.Model):
       user  = models.ForeignKey(User, on_delete=models.CASCADE)
       item  = models.CharField(max_length=100)
       duplicate = models.BooleanField(default=False)
   ```
2. Use `pre_save` to detect duplicates (see above).  
3. In `post_save`, notify if `instance.duplicate` is True.

## Sending Notifications
Inside a signal receiver:
```python
from django.core.mail import send_mail

@receiver(post_save, sender=Order)
def notify_manager(sender, instance, created, **kwargs):
    if created and instance.duplicate:
        send_mail(
          subject="Duplicate Order Alert",
          message=f"User {instance.user} placed duplicate {instance.item}.",
          from_email="noreply@example.com",
          recipient_list=["manager@example.com"],
        )
```

## Async Tasks with Celery
Offload heavy work:
```python
# tasks.py
from celery import shared_task
from django.core.mail import send_mail

@shared_task
def notify_manager_task(user, item):
    send_mail(...)

# in signal
@receiver(post_save, sender=Order)
def notify_async(sender, instance, created, **kwargs):
    if created and instance.duplicate:
        notify_manager_task.delay(instance.user.id, instance.item)
```

## Real-time Notifications to React Frontend

### Django Side

1. Add a `Notification` model:
```python
# filepath: c:\Users\nivoj\Desktop\learining_2025\Dajngo\models.py
class Notification(models.Model):
    user      = models.ForeignKey(User, on_delete=models.CASCADE)
    message   = models.CharField(max_length=255)
    created   = models.DateTimeField(auto_now_add=True)
    read      = models.BooleanField(default=False)
```

2. Create notifications in your signal:
```python
# filepath: c:\Users\nivoj\Desktop\learining_2025\Dajngo\signals.py
from .models import Notification

@receiver(post_save, sender=Order)
def notify_manager(sender, instance, created, **kwargs):
    if created and instance.duplicate:
        Notification.objects.create(
            user=instance.user.manager,  # assign to manager
            message=f"Duplicate order for {instance.item}"
        )
```

3. Expose an API endpoint for unread notifications:
```python
# filepath: c:\Users\nivoj\Desktop\learining_2025\Dajngo\views.py
from rest_framework import viewsets, permissions
from .models import Notification
from .serializers import NotificationSerializer

class NotificationViewSet(viewsets.ReadOnlyModelViewSet):
    serializer_class = NotificationSerializer
    permission_classes = [permissions.IsAuthenticated]

    def get_queryset(self):
        return Notification.objects.filter(
            user=self.request.user, read=False
        ).order_by('-created')
```
```python
# filepath: c:\Users\nivoj\Desktop\learining_2025\Dajngo\urls.py
from rest_framework.routers import DefaultRouter
from .views import NotificationViewSet

router = DefaultRouter()
router.register(r'notifications', NotificationViewSet, basename='notification')
urlpatterns += router.urls
```

### React Side

```jsx
// filepath: src/components/NotificationsTab.jsx
import React, {useState, useEffect} from 'react';

export default function NotificationsTab() {
  const [notes, setNotes] = useState([]);

  useEffect(() => {
    const fetchNotes = () => {
      fetch('/api/notifications/', {
        credentials: 'include', // or Authorization header
      })
      .then(res => res.json())
      .then(data => setNotes(data));
    };
    fetchNotes();
    const iv = setInterval(fetchNotes, 10000); // poll every 10s
    return () => clearInterval(iv);
  }, []);

  return (
    <div>
      <h2>Notifications ({notes.length})</h2>
      <ul>
        {notes.map(n => (
          <li key={n.id}>{n.message} — {new Date(n.created).toLocaleTimeString()}</li>
        ))}
      </ul>
    </div>
  );
}
```

## Alternative Notification Delivery

### 1. Polling (simple, low-freq)
React: already shown using `setInterval`  
Django: standard DRF endpoint

### 2. WebSockets (high-freq, real-time UI)
```python
# Django: consumers.py
from channels.generic.websocket import AsyncJsonWebsocketConsumer

class NotificationConsumer(AsyncJsonWebsocketConsumer):
    async def connect(self):
        await self.channel_layer.group_add("notif_group", self.channel_name)
        await self.accept()

    async def disconnect(self, code):
        await self.channel_layer.group_discard("notif_group", self.channel_name)

    async def notify(self, event):
        await self.send_json(event["data"])
```
```javascript
// React:
useEffect(() => {
  const ws = new WebSocket("ws://localhost:8000/ws/notifications/");
  ws.onmessage = e => setNotes(prev => [JSON.parse(e.data), ...prev]);
  return () => ws.close();
}, []);
```

### 3. Server-Sent Events (uni-directional, lightweight)
```python
# Django: views.py
from django.http import StreamingHttpResponse
import time, json

def sse_notifications(request):
    def stream():
        while True:
            # fetch new notifications
            yield f"data: {json.dumps(get_new())}\n\n"
            time.sleep(5)
    return StreamingHttpResponse(stream(), content_type='text/event-stream')
```
```javascript
// React:
useEffect(() => {
  const es = new EventSource("/api/notifications/sse/");
  es.onmessage = e => setNotes(prev => [JSON.parse(e.data), ...prev]);
  return () => es.close();
}, []);
```

### 4. Web Push (OS/browser alerts)
```python
# Django: utils.py
from pywebpush import webpush

def send_push(sub, payload):
    webpush(sub, payload, vapid_private_key=KEY, vapid_claims={"sub": "mailto:you@example.com"})
```
```javascript
// React (service worker registration)
navigator.serviceWorker.register('/sw.js').then(reg =>
  reg.pushManager.subscribe({
    userVisibleOnly: true,
    applicationServerKey: VAPID_KEY
  })
).then(sub => fetch('/api/save-sub/', {method:'POST', body:JSON.stringify(sub)}));
```

**When to use what**  
- Polling: easiest, OK for low update rates.  
- WebSockets: best for bidirectional, real-time dashboards.  
- SSE: simple server→client stream, no WS overhead.  
- Web Push: out-of-browser notifications, requires HTTPS and user opt-in.

## Useful Libraries
- **django-notifications**: in-app notifications  
- **django-model-utils**: `FieldTracker` for change detection  
- **django-events** / **django-dispatcher**: advanced event frameworks  

---  
Explore more:  
https://docs.djangoproject.com/en/stable/topics/signals/
