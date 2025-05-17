# Django Testing Guide

## Navigation
- [Overview](#overview)
- [Creating Test Cases](#creating-test-cases)
- [Running Tests](#running-tests)
- [Using the Test Client](#using-the-test-client)
- [Common Assertions](#common-assertions)
- [Testing Models](#testing-models)
- [Testing Views & URLs](#testing-views--urls)
- [Testing Forms](#testing-forms)
- [Fixtures & Test Data](#fixtures--test-data)
- [Measuring Coverage](#measuring-coverage)

## Overview
Django tests live in `tests.py` (or `tests/` package). Tests help catch regressions and ensure your app behaves as expected.

## Creating Test Cases
```python
from django.test import TestCase
from .models import Post

class PostModelTest(TestCase):
    @classmethod
    def setUpTestData(cls):
        cls.post = Post.objects.create(title="T", content="C")

    def test_title_max_length(self):
        self.assertEqual(self.post._meta.get_field('title').max_length, 200)
```

## Running Tests
```bash
python manage.py test            # run all tests
python manage.py test app_label  # tests for a specific app
```

## Using the Test Client
Simulate HTTP requests in tests:
```python
response = self.client.get('/posts/')
self.assertEqual(response.status_code, 200)
```
Post data:
```python
response = self.client.post('/posts/create/', {'title':'A','content':'B'})
```

## Common Assertions
- `assertEqual(a, b)`  
- `assertTrue(x)` / `assertFalse(x)`  
- `assertContains(resp, text)` / `assertNotContains(resp, text)`  
- `assertRedirects(resp, url)`  

## Testing Models
Validate model behaviors and custom methods:
```python
self.assertTrue(Post.objects.filter(title="T").exists())
self.assertEqual(self.post.get_summary(), "T…")
```

## Testing Views & URLs
```python
from django.urls import reverse

url = reverse('post-detail', kwargs={'pk': self.post.pk})
resp = self.client.get(url)
self.assertEqual(resp.context['post'], self.post)
```

## Testing Forms
```python
from .forms import PostForm
form = PostForm(data={'title':'','content':'x'})
self.assertFalse(form.is_valid())
self.assertIn('title', form.errors)
```

## Fixtures & Test Data
- Use `fixtures = ['initial_data.json']` in TestCase  
- Or override `setUp()` / `setUpTestData()` to create objects  

## Measuring Coverage
Install `coverage`:
```bash
coverage run --source='.' manage.py test
coverage report
coverage html
```
