# Introduction to Django Models

## Navigation
- [1. Defining a Model](#1-defining-a-model)
- [2. Common Field Types](#2-common-field-types)
- [3. Applying Model Changes](#3-applying-model-changes)
- [4. Registering in Admin](#4-registering-in-admin)
- [5. Querying the Database](#5-querying-the-database)
- [6. Model Methods & Meta](#6-model-methods--meta)
- [7. Custom User Model & Manager](#7-custom-user-model--manager)

## 1. Defining a Model

```python
from django.db import models
from django.contrib.auth.models import User

class Post(models.Model):
    title      = models.CharField(max_length=200)
    content    = models.TextField()
    published  = models.DateTimeField(auto_now_add=True)
    author     = models.ForeignKey(User, on_delete=models.CASCADE)

    def __str__(self):
        return self.title
```

## 2. Common Field Types
Below are examples of how each field is declared in code:

```python
# CharField: fixed-length string
name = models.CharField(max_length=200, default='')

# TextField: large text
description = models.TextField(null=True, blank=True)

# IntegerField: whole numbers
quantity = models.IntegerField(default=0)

# FloatField: decimal numbers
price = models.FloatField(default=0.0)

# BooleanField: True/False
is_active = models.BooleanField(default=True)

# DateField & DateTimeField
created_date   = models.DateField(auto_now_add=True)
modified_date  = models.DateTimeField(auto_now=True)

# ForeignKey: many-to-one relationship
author = models.ForeignKey(User, on_delete=models.CASCADE)

# OneToOneField: one-to-one relationship
profile = models.OneToOneField(Profile, on_delete=models.CASCADE, null=True, blank=True)

# ManyToManyField: many-to-many relationship
tags = models.ManyToManyField(Tag, blank=True)

# FileField: upload any file
document = models.FileField(upload_to='documents/%Y/%m/%d/', null=True, blank=True)

# ImageField: upload images (requires Pillow)
photo = models.ImageField(upload_to='photos/%Y/%m/%d/', null=True, blank=True)
```

Each field takes optional arguments (e.g. `max_length`, `default`, `null`, `blank`).

## 3. Applying Model Changes
1. Add your app to `INSTALLED_APPS` in `settings.py`.  
2. Run:
   ```bash
   python manage.py makemigrations
   python manage.py migrate
   ```

## 4. Registering in Admin
In `yourapp/admin.py`:
```python
from django.contrib import admin
from .models import Post

admin.site.register(Post)
```

## 5. Querying the Database
```python
# import model
from yourapp.models import Post

# create
Post.objects.create(title="Hello", content="World", author=user)

# retrieve
posts = Post.objects.all()
recent = Post.objects.filter(published__year=2023).order_by('-published')
one = Post.objects.get(id=1)

# update
post = Post.objects.first()
post.title = "New Title"
post.save()

# delete
post.delete()

# Additional QuerySet Examples
# filter with multiple conditions
drafts = Post.objects.filter(is_active=True, published__year=2023)

# case-insensitive contains lookup
qs = Post.objects.filter(title__icontains='hello')

# exclude records
old_posts = Post.objects.exclude(published__lt='2022-01-01')

# get or create
post, created = Post.objects.get_or_create(
    title="Hello",
    defaults={'content': "World", 'author': user}
)

# bulk create
new_posts = [
    Post(title="Title1", content="C1", author=user),
    Post(title="Title2", content="C2", author=user)
]
Post.objects.bulk_create(new_posts)

# check existence
has_posts = Post.objects.filter(author=user).exists()

# count total records
total_posts = Post.objects.count()
```

### Lookup Expressions
Django uses `field__lookup=value` syntax to filter by operators or transformations.  
Common lookups include:

- exact / iexact        → field__exact="abc", field__iexact="ABC"  
- contains / icontains  → field__contains="foo", field__icontains="Foo"  
- startswith / istartswith  
- endswith / iendswith  
- in                    → field__in=[1,2,3]  
- gt / gte / lt / lte   → published__lt='2022-01-01', price__gte=10  
- range                 → date__range=['2023-01-01','2023-12-31']  
- isnull                → field__isnull=True  
- year / month / day    → published__year=2023, date__month=12  
- week_day / hour / minute / second  

Examples:  
```python
Post.objects.filter(published__year=2023)  
Post.objects.filter(title__startswith='Hello')  
Post.objects.filter(id__in=[1,2,3])  
```

**Reference:**  
Full list of field lookups → https://docs.djangoproject.com/en/stable/ref/models/querysets/#field-lookups

## 6. Model Methods & Meta
- Define custom methods inside your model.  
- Use `class Meta` to set ordering, verbose names, unique constraints, etc.
  ```python
  class Meta:
      ordering = ['-published']
      verbose_name = "Blog Post"
  ```

## 7. Custom User Model & Manager

Below are the roles of the imported base classes:
- **AbstractBaseUser**: Provides core user functionalities (password hashing, authentication) without defining fields like username.
- **BaseUserManager**: Offers helper methods to create and manage user instances, handling email normalization and saving logic.
- **PermissionsMixin**: Adds permission-related fields and methods (is_superuser, groups, user_permissions) to support Django’s auth system.

```python
from django.db import models
from django.contrib.auth.models import (
    AbstractBaseUser, BaseUserManager, PermissionsMixin
)

class CustomUserManager(BaseUserManager):
    """
    Manager for CustomUser, providing methods to create regular users and superusers.
    """

    def create_user(self, email, password=None, **extra_fields):
        """
        Create and save a regular user with the given email and password.
        """
        if not email:
            raise ValueError('An email address is required')
        # Normalize and set email
        email = self.normalize_email(email)
        user = self.model(email=email, **extra_fields)
        user.set_password(password)      # Hashes the password
        user.save(using=self._db)        # Save to the configured database
        return user

    def create_superuser(self, email, password, **extra_fields):
        """
        Create and save a superuser with staff and superuser permissions.
        """
        extra_fields.setdefault('is_staff', True)
        extra_fields.setdefault('is_superuser', True)

        if extra_fields.get('is_staff') is not True or extra_fields.get('is_superuser') is not True:
            raise ValueError('Superuser must have is_staff=True and is_superuser=True')
        return self.create_user(email, password, **extra_fields)


class CustomUser(AbstractBaseUser, PermissionsMixin):
    """
    Custom user model that replaces username with email as the unique identifier.
    """
    email      = models.EmailField('email address', unique=True)
    first_name = models.CharField('first name', max_length=30)
    last_name  = models.CharField('last name', max_length=30)
    is_staff   = models.BooleanField(default=False, help_text='Access to admin site')
    is_active  = models.BooleanField(default=True, help_text='Active user account')

    objects = CustomUserManager()      # Link to the custom manager

    USERNAME_FIELD = 'email'           # Use email for authentication
    REQUIRED_FIELDS = ['first_name', 'last_name']

    def __str__(self):
        return self.email
```

In `settings.py`, add:

```python
# Tell Django to use this custom user model
AUTH_USER_MODEL = 'yourapp.CustomUser'
```

---

Explore more at https://docs.djangoproject.com/en/stable/topics/db/models/
