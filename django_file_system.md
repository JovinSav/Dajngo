# Django Project File System Overview

## Navigation
- [Default Project Structure](#default-project-structure)
- [Top-Level Files](#top-level-files)
- [Project Package (`mysite/`)](#project-package-mysite)
- [Apps](#apps)
- [Other Common Folders](#other-common-folders)

## Default Project Structure

When you run `django-admin startproject mysite`, Django creates the following structure:

```
mysite/
├── manage.py          # Command-line utility for project management
└── mysite/            # Project package
    ├── __init__.py    # Marks this directory as a Python package
    ├── settings.py    # Project settings and configurations
    ├── urls.py        # URL-to-view mappings
    ├── wsgi.py        # Entry point for WSGI-compatible web servers
    └── asgi.py        # Entry point for ASGI-compatible web servers
```

## Top-Level Files

• manage.py  
  A CLI utility to interact with your project (runserver, migrations, etc.).

## Project Package (`mysite/`)

• __init__.py  
  Marks this directory as a Python package.

• settings.py  
  Global configuration (databases, installed apps, middleware, templates, static files).

• urls.py  
  URL-to-view mappings for your project.

• wsgi.py & asgi.py  
  Entry points for WSGI/ASGI-compatible web servers.

## Apps

When you run `python manage.py startapp blog`, you get:

```
blog/
├── migrations/
│   └── __init__.py  # Marks migrations as a Python package
├── __init__.py      # Marks this directory as a Python package
├── admin.py         # Register models with Django’s admin site
├── apps.py          # App configuration
├── models.py        # Data models (DB tables)
├── tests.py         # Unit tests for this app
└── views.py         # Request handlers (logic)
```

• migrations/  
  Auto-generated files to track model/schema changes.

• admin.py  
  Register models with Django’s admin site.

• apps.py  
  App configuration.

• models.py  
  Data models (DB tables).

• views.py  
  Request handlers (logic).

• tests.py  
  Unit tests for this app.  
  - Create test cases by subclassing `django.test.TestCase`  
  - Run `python manage.py test` to discover and execute tests  
  - Use `self.client` to simulate HTTP requests and assertion methods (`assertEqual`, `assertContains`, etc.) to verify behavior  
  - Validate models, views, forms, URLs and catch regressions early

## Other Common Folders

• templates/  
  HTML templates (configure `TEMPLATESDIRS`).

• static/  
  CSS, JS, images (configure `STATICFILES_DIRS`).

• media/  
  Uploaded files (configure `MEDIA_ROOT` & `MEDIA_URL`).

---

This layout helps you organize code by functionality and keep settings centralized. Happy coding!
