# Django settings.py Deep Dive

The `settings.py` file holds all the configuration for your Django project. Below are its most important sections:

## 1. BASE_DIR
Defines the project’s root directory.
```python
import os
BASE_DIR = os.path.dirname(os.path.dirname(__file__))
```

## 2. SECRET_KEY
A unique key used for cryptographic signing. **Keep it secret** in production.
```python
SECRET_KEY = 'your-secret-key-here'
```

## 3. DEBUG
Toggle debug mode. `True` shows detailed errors; always set to `False` in production.
```python
DEBUG = True
```

## 4. ALLOWED_HOSTS
List of domain names/IPs your site can serve.
```python
ALLOWED_HOSTS = ['localhost', '127.0.0.1']
```

## 5. INSTALLED_APPS
Apps and Django contrib modules enabled in this project.
```python
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    # ... other default apps ...
    'blog',           # your custom app
]
```

## 6. MIDDLEWARE
Middleware classes that process request/response.
```python
MIDDLEWARE = [
    'django.middleware.security.SecurityMiddleware',
    'django.contrib.sessions.middleware.SessionMiddleware',
    # ... more middleware ...
]
```

## 7. ROOT_URLCONF
The module that defines your URL patterns.
```python
ROOT_URLCONF = 'mysite.urls'
```

## 8. TEMPLATES
Configuration for the template engine.
```python
TEMPLATES = [
    {
        'BACKEND': 'django.template.backends.django.DjangoTemplates',
        'DIRS': [os.path.join(BASE_DIR, 'templates')],
        # ... more settings ...
    },
]
```

## 9. WSGI_APPLICATION
Entry point for WSGI servers.
```python
WSGI_APPLICATION = 'mysite.wsgi.application'
```

## 10. DATABASES
Define database connections (default is SQLite).
```python
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.sqlite3',
        'NAME': os.path.join(BASE_DIR, 'db.sqlite3'),
    }
}
```

## 11. AUTH_PASSWORD_VALIDATORS
Password strength validation.
```python
AUTH_PASSWORD_VALIDATORS = [
    {'NAME': 'django.contrib.auth.password_validation.UserAttributeSimilarityValidator'},
    # ... more validators ...
]
```

## 12. INTERNATIONALIZATION
Language, timezone, and formatting.
```python
LANGUAGE_CODE = 'en-us'
TIME_ZONE = 'UTC'
USE_I18N = True
USE_L10N = True
USE_TZ = True
```

## 13. STATIC & MEDIA FILES
Paths for serving static and media files.
```python
STATIC_URL = '/static/'
STATICFILES_DIRS = [os.path.join(BASE_DIR, 'static')]

MEDIA_URL = '/media/'
MEDIA_ROOT = os.path.join(BASE_DIR, 'media')
```

## Static & Media Settings Explained
- **STATIC_URL**: URL prefix used to reference static files in your templates.
- **STATICFILES_DIRS**: Extra folders Django will search for static assets during development.
- **STATIC_ROOT**: Destination folder for all static files when running `collectstatic` in production.
- **MEDIA_URL**: URL prefix for serving user-uploaded media files.
- **MEDIA_ROOT**: Filesystem path where uploaded media is stored.

## 14. Docker Environment
When deploying with Docker, load settings from environment variables and adjust paths:

```python
import os
from pathlib import Path
from dotenv import load_dotenv

load_dotenv()
BASE_DIR = Path(__file__).resolve().parent.parent

SECRET_KEY = os.getenv('DJANGO_SECRET_KEY', 'change-me')
DEBUG = os.getenv('DJANGO_DEBUG', 'False') == 'True'

# Allowed hosts must include the hostname Nginx uses (or your domain)
# In your .env, for example:
# DJANGO_ALLOWED_HOSTS=example.com,nginx,localhost
ALLOWED_HOSTS = os.getenv('DJANGO_ALLOWED_HOSTS', 'localhost').split(',')

DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql',
        'NAME': os.getenv('POSTGRES_DB'),
        'USER': os.getenv('POSTGRES_USER'),
        'PASSWORD': os.getenv('POSTGRES_PASSWORD'),
        'HOST': os.getenv('POSTGRES_HOST', 'db'),
        'PORT': os.getenv('POSTGRES_PORT', '5432'),
    }
}

# Collect static files into this folder for Docker
STATIC_ROOT = BASE_DIR / 'staticfiles'
```

---

Refer to the official docs for more advanced settings:  
https://docs.djangoproject.com/en/stable/ref/settings/

Refer to your Docker-Compose and `.env` for variable definitions.
