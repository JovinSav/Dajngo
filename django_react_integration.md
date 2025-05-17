# Integrating React with Django

## Navigation
- [Overview](#overview)
- [Integration Approaches](#integration-approaches)
- [Authentication Strategies](#authentication-strategies)
- [Deployment Strategies](#deployment-strategies)
- [Resources](#resources)

## Overview
You can use Django as a backend API and React for the frontend. Common patterns:
- Single repo vs. separate repos  
- Bundled build served by Django or served by a dedicated web server  

## Integration Approaches
1. Django-served build  
   • Build React (`npm run build`) → place in Django’s `static/` → serve index.html via a Django view.  
2. Separate apps  
   • React runs on Node (dev: `npm start`), Django on port 8000 → configure CORS.  
3. Django template rendering  
   • Embed React components in Django templates with `<div id="root">` and include built JS.

## Authentication Strategies
- **SessionAuth (with CSRF)**  
  • Best when React is served by Django on the same domain.  
  • Leverages existing Django login/logout and CSRF protections.  
  • Use for full-stack apps without cross‐origin concerns.

- **TokenAuth (DRF TokenAuthentication)**  
  • Simple token per user; stored in localStorage or memory.  
  • Good for mobile/native apps or third‐party clients.  
  • Client includes `Authorization: Token <key>` header.

- **JWT (djangorestframework-simplejwt)**  
  • Stateless, scalable; tokens contain expiry and claims.  
  • Ideal for SPAs with separate frontends, CORS-enabled.  
  • Store in HttpOnly cookie (secure) or memory and refresh with refresh token.

- **OAuth2/OpenID (django-allauth + dj-rest-auth)**  
  • Use for social login, third‐party authentication, or enterprise SSO.  
  • Implements flows (authorization code, PKCE) suited to public clients.

Example (JWT):
```python
# settings.py
INSTALLED_APPS += ['rest_framework_simplejwt']
REST_FRAMEWORK['DEFAULT_AUTHENTICATION_CLASSES'] = [
  'rest_framework_simplejwt.authentication.JWTAuthentication',
]
```
In React, store token in memory or `HttpOnly` cookie and attach to `Authorization: Bearer <token>`.

## Deployment Strategies
1. Monolithic  
   • Build React → collect into Django static → one server (e.g. Gunicorn + WhiteNoise).  
2. Split services  
   • Django API on one domain/subdomain, React SPA on CDN or Node‐based server → set CORS & proxy.  
3. Containerized  
   • Docker Compose: one service for Django, one for Nginx serving React, plus DB.  
   • Use multi‐stage build for React to keep container lean.

## Resources
- Django docs: https://docs.djangoproject.com/  
- DRF JWT: https://github.com/jazzband/djangorestframework-simplejwt  
- CORS: https://github.com/adamchainz/django-cors-headers  
- Deployment (WhiteNoise): http://whitenoise.evans.io/
