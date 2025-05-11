# Introduction to Django

Django is a high-level Python web framework that encourages rapid development and clean, pragmatic design. It handles much of the hassle of web development so you can focus on writing your app without needing to reinvent the wheel.

## Prerequisites
- Python 3.8+
- pip (comes with Python)
- (Optional) virtualenv or venv for isolated environments

## 1. Create a Virtual Environment
```bash
# Windows
python -m venv env
env\Scripts\activate

# macOS/Linux
python3 -m venv env
source env/bin/activate
```

## 2. Install Django
```bash
pip install django
```

## 3. Start a New Project
```bash
django-admin startproject mysite
cd mysite
```

## 4. Run the Development Server
```bash
python manage.py runserver
```
Visit http://127.0.0.1:8000/ to see the welcome page.

## 5. Create a New App
```bash
python manage.py startapp blog
```
- Add `'blog'` to `INSTALLED_APPS` in settings.py.

## 6. Apply Migrations
```bash
python manage.py makemigrations
python manage.py migrate
```

## What are Migrations?
Migrations in Django are its built-in way to track and apply changes you make to your models (your Python classes) into your database schema. Think of them as version control for your database:
1. You change or create a model (e.g. add a field).  
2. You run `python manage.py makemigrations`.  
   • Django generates a new migration file describing the SQL changes.  
3. You run `python manage.py migrate`.  
   • Django reads and applies these migrations to the database in the correct order.

Benefits:
- Keeps your schema in sync with your code.  
- Lets you roll forward and backwards through schema changes.  
- Enables teams to collaborate without manually writing SQL.

## 7. Use the Admin Site
1. Create a superuser:  
   `python manage.py createsuperuser`  
2. Run server and go to `/admin/`.

## 8. Types of Views

Django offers several ways to write views:

### Function-Based Views (FBV)
```python
from django.shortcuts import render

def my_view(request):
    # handle request
    context = {'key': 'value'}
    return render(request, 'template.html', context)
```

### Class-Based Views (CBV)
```python
from django.views import View
from django.shortcuts import render

class MyView(View):
    def get(self, request):
        return render(request, 'template.html', {})
```

### Generic Class-Based Views
Built-in views for common patterns:
```python
from django.views.generic import ListView, DetailView
from .models import Post

class PostListView(ListView):
    model = Post
    template_name = 'post_list.html'

class PostDetailView(DetailView):
    model = Post
    template_name = 'post_detail.html'
```
Other generics include CreateView, UpdateView, DeleteView, FormView.

## 9. Serving Static Files

### Development
- Ensure `django.contrib.staticfiles` is in `INSTALLED_APPS` (default).
- Define `STATIC_URL = '/static/'` in `settings.py`.
- Place your files under an app’s `static/` folder.
- During `runserver`, Django auto-serves static assets.

In templates:
```django
{% load static %}
<img src="{% static 'app_name/images/logo.png' %}" alt="Logo">
```

### Production
1. Configure in `settings.py`:
   ```python
   STATIC_ROOT = os.path.join(BASE_DIR, 'staticfiles')
   ```
2. Run `python manage.py collectstatic` to gather assets into `STATIC_ROOT`.
3. Configure your web server (e.g. Nginx) to serve `/static/` from `staticfiles/`:
   ```
   location /static/ {
       alias /path/to/your/project/staticfiles/;
   }
   ```

## Next Steps
- Learn about URLs, views, and templates  
- Work with models and the ORM  
- Explore Django’s built-in authentication  
- Check the official docs: https://docs.djangoproject.com/

Happy coding with Django!
