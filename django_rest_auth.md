# DRF Authentication & Permissions

## Navigation
- [Overview](#overview)
- [Default Classes](#default-classes)
- [Authentication Schemes](#authentication-schemes)
  - [SessionAuthentication](#sessionauthentication)
  - [BasicAuthentication](#basicauthentication)
  - [TokenAuthentication](#tokenauthentication)
  - [JWT Authentication](#jwt-authentication)
- [Permission Classes](#permission-classes)
  - [AllowAny](#allowany)
  - [IsAuthenticated](#isauthenticated)
  - [IsAdminUser](#isadminuser)
  - [IsAuthenticatedOrReadOnly](#isauthenticatedorreadonly)
- [Customizing Defaults](#customizing-defaults)
- [Dynamic Authentication per User/Client](#dynamic-authentication-per-userclient)

## Overview
DRF separates **authentication** (who you are) from **permissions** (what you can do).  
Configure defaults in your `REST_FRAMEWORK` setting.

## Default Classes
```python
REST_FRAMEWORK = {
    'DEFAULT_AUTHENTICATION_CLASSES': [
        'rest_framework.authentication.SessionAuthentication',
        'rest_framework.authentication.BasicAuthentication',
    ],
    'DEFAULT_PERMISSION_CLASSES': [
        'rest_framework.permissions.AllowAny',
    ],
}
```

## Authentication Schemes

### SessionAuthentication  
Uses Django’s session framework (cookies + CSRF). Good for web-browsable API.

### BasicAuthentication  
HTTP Basic auth; credentials sent with each request (over HTTPS).

### TokenAuthentication  
```bash
pip install djangorestframework-authtoken
```
```python
INSTALLED_APPS += ['rest_framework.authtoken']
# settings.py
REST_FRAMEWORK['DEFAULT_AUTHENTICATION_CLASSES'] += [
    'rest_framework.authentication.TokenAuthentication',
]
```
Obtain token via `rest_framework.authtoken.views.obtain_auth_token`.

### JWT Authentication  
Third-party (e.g., djangorestframework-simplejwt):
```bash
pip install djangorestframework-simplejwt
```
```python
REST_FRAMEWORK['DEFAULT_AUTHENTICATION_CLASSES'] = [
    'rest_framework_simplejwt.authentication.JWTAuthentication',
]
```

## Permission Classes

### AllowAny  
(No restrictions — open to all)

### IsAuthenticated  
(Any logged-in user)

### IsAdminUser  
(`is_staff`==True only)

### IsAuthenticatedOrReadOnly  
(Read-only for unauthenticated; full for authenticated)

## Customizing Defaults
Override in `settings.py`:
```python
REST_FRAMEWORK['DEFAULT_PERMISSION_CLASSES'] = [
    'rest_framework.permissions.IsAuthenticated',
]
REST_FRAMEWORK['DEFAULT_AUTHENTICATION_CLASSES'] = [
    'myapp.auth.CustomHeaderAuthentication',
]
```
Define a custom permission:
```python
from rest_framework.permissions import BasePermission

class IsOwner(BasePermission):
    def has_object_permission(self, request, view, obj):
        return obj.owner == request.user
```
Apply per-view:
```python
from rest_framework.decorators import permission_classes

@permission_classes([IsOwner])
class MyView(...): ...
```

## Dynamic Authentication per User/Client
You can override `get_authenticators()` in a view to return different auth classes based on the request (e.g. staff users vs. API tokens):

```python
from rest_framework.views import APIView
from rest_framework.authentication import SessionAuthentication, BasicAuthentication, TokenAuthentication

class MixedAuthView(APIView):
    def get_authenticators(self):
        # if human user (staff), allow session or basic auth
        if self.request.user and self.request.user.is_staff:
            return [SessionAuthentication(), BasicAuthentication()]
        # otherwise require token for machine clients
        return [TokenAuthentication()]
```

Or simply set per-view:

```python
class MachineOnlyView(APIView):
    authentication_classes = [TokenAuthentication]
```
