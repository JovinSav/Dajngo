# Django REST Framework Guide

## Navigation
- [Installation](#installation)
- [Adding to INSTALLED_APPS & Settings](#adding-to-installed_apps--settings)
- [Serializers](#serializers)
- [Views](#views)
- [URL Routing & Routers](#url-routing--routers)
- [Authentication & Permissions](#authentication--permissions)
- [Pagination & Filtering](#pagination--filtering)
- [Testing APIs](#testing-apis)
- [Resources](#resources)

## Installation
```bash
pip install djangorestframework
```

## Adding to INSTALLED_APPS & Settings
```python
# settings.py
INSTALLED_APPS += [
    'rest_framework',
    'yourapp',
]

REST_FRAMEWORK = {
    'DEFAULT_AUTHENTICATION_CLASSES': [
        'rest_framework.authentication.SessionAuthentication',
        'rest_framework.authentication.TokenAuthentication',
    ],
    'DEFAULT_PERMISSION_CLASSES': [
        'rest_framework.permissions.IsAuthenticatedOrReadOnly',
    ],
}
```

## Serializers
```python
# yourapp/serializers.py
from rest_framework import serializers
from .models import Post

class PostSerializer(serializers.ModelSerializer):
    class Meta:
        model = Post
        fields = ['id', 'title', 'content', 'author', 'published']
```

## Views
```python
# yourapp/views.py
from rest_framework import viewsets
from .models import Post
from .serializers import PostSerializer

class PostViewSet(viewsets.ModelViewSet):
    queryset = Post.objects.all()
    serializer_class = PostSerializer
```

## URL Routing & Routers
```python
# yourapp/urls.py
from rest_framework.routers import DefaultRouter
from .views import PostViewSet

router = DefaultRouter()
router.register(r'posts', PostViewSet)

urlpatterns = router.urls
```

Include in project URLs:
```python
# mysite/urls.py
from django.urls import path, include

urlpatterns = [
    # ... existing patterns ...
    path('api/', include('yourapp.urls')),
]
```

## Authentication & Permissions
- Obtain a token:
  ```bash
  POST /api-token-auth/  # returns {"token": "..."}
  ```
- Protect an endpoint:
  ```python
  from rest_framework.permissions import IsAdminUser

  class AdminPostViewSet(viewsets.ModelViewSet):
      permission_classes = [IsAdminUser]
      ...
  ```

## Pagination & Filtering
```python
# settings.py
REST_FRAMEWORK['DEFAULT_PAGINATION_CLASS'] = \
    'rest_framework.pagination.PageNumberPagination'
REST_FRAMEWORK['PAGE_SIZE'] = 10
```
Use filters:
```python
# views.py
from rest_framework import filters

class PostViewSet(...):
    filter_backends = [filters.SearchFilter, filters.OrderingFilter]
    search_fields = ['title', 'content']
    ordering_fields = ['published']
```

## Testing APIs
```python
from rest_framework.test import APITestCase
from django.urls import reverse

class PostAPITest(APITestCase):
    def test_list_posts(self):
        url = reverse('post-list')
        response = self.client.get(url)
        self.assertEqual(response.status_code, 200)
```

## Resources
- Official docs: https://www.django-rest-framework.org/  
- Tutorial: https://www.django-rest-framework.org/tutorial/  
