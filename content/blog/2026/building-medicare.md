---
title: "Building MediCare: A Full-Stack Hospital Management System"
date: 2026-05-11
description: "What I learned while building a role-based hospital management system with Flask, Vue.js, Celery, Redis, and Cloudflare R2."
categories:
  - Software Engineering
tags:
  - Flask
  - Vue.js
  - Redis
  - Celery
  - SQLAlchemy
  - REST API
draft: false
---

**MediCare** is a full-stack hospital management system I built to explore how a larger web application can handle multiple user roles, background tasks, document storage, and different workflows through a common backend.

The application uses **Flask** for the backend and **Vue.js** for the frontend, with SQLAlchemy for database access and JWT-based authentication.

<a href="https://www.youtube.com/watch?v=dEuNWN0dwdU&t=5s" target="_blank" rel="noopener noreferrer">Demo Video</a>

## The problem

A hospital management application involves more than simply storing patients and appointments.

Different users need access to different parts of the system, and many actions depend on the role of the person using the application.

MediCare supports four main roles:

- Admin
- Doctor
- Patient
- Associate

Each role has its own permissions and workflows.

For example, a patient should not have access to administrative functionality, while a doctor needs access to information and actions related to patient care.

The application therefore needed both **authentication** and **authorization**.

## Application architecture

I separated the application into a Vue frontend and a Flask backend.

The overall architecture looks roughly like this:

```text
Vue.js Frontend
       │
       │ HTTP / REST
       ▼
Flask API
       │
       ├── SQLAlchemy
       │       │
       │       ▼
       │    Database
       │
       ├── Redis
       │
       ├── Celery Workers
       │
       └── Cloudflare R2
               │
               ▼
          Documents / Files
```

The frontend communicates with the backend through REST APIs rather than directly accessing the database.

This separation made it easier to keep the responsibilities of the frontend and backend clear.

## Role-based access control

One of the more important parts of the project was designing the application around different user roles.

Authentication answers:

> Who is making this request?

Authorization answers:

> Is this user allowed to perform this action?

After authentication, the backend uses the user's identity and role to determine which operations are allowed.

Conceptually:

```text
User logs in
     │
     ▼
Credentials verified
     │
     ▼
JWT issued
     │
     ▼
Authenticated API request
     │
     ▼
Role / permission check
     │
     ├── Allowed → Continue
     │
     └── Denied  → Reject request
```

This was more useful than relying only on frontend restrictions.

Even if a page or button is hidden in the browser, the backend still needs to verify that the user is authorized to call the corresponding API.

## Building the REST API

The Flask backend exposes REST endpoints that the Vue frontend can consume.

A simplified request flow looks like:

```text
Vue Component
     │
     ▼
Axios Request
     │
     ▼
Flask Route
     │
     ▼
Authentication / Validation
     │
     ▼
Application Logic
     │
     ▼
SQLAlchemy
     │
     ▼
Database
```

I also maintained an **OpenAPI specification** for the API.

This helped document the available endpoints and made the frontend/backend contract easier to understand.

## SQLAlchemy and database access

I used **SQLAlchemy** as the ORM layer rather than writing SQL directly throughout the application.

This allowed the application code to work with Python objects while keeping database operations organized.

For example, relationships between users and application entities could be represented through models instead of manually reconstructing them after every query.

Using an ORM also made it easier to keep database-related logic separate from the route handlers.

## Why I added Redis and Celery

Not every task should happen inside the HTTP request that triggered it.

Some operations can take longer or do not need to finish before the user receives a response.

Examples include:

* scheduled notifications
* sending emails
* generating exports
* background processing
* recurring jobs

Running all of these directly inside Flask request handlers would make requests slower and tightly couple background work to the web server.

Instead, I used **Celery** with **Redis**.

The idea is:

```text
User Request
     │
     ▼
Flask API
     │
     ├──────────────► Immediate Response
     │
     ▼
Celery Task Queue
     │
     ▼
Redis
     │
     ▼
Celery Worker
     │
     ▼
Background Task
```

The Flask application can queue work and return a response without waiting for the entire background operation to finish.

## Scheduled tasks

Some tasks also need to run automatically rather than being triggered directly by a user.

For example, the application can perform scheduled operations related to notifications or reports.

This introduced an important distinction between:

```text
Web requests
```

and:

```text
Background / scheduled work
```

Keeping these separate made the architecture much easier to reason about.

## Using Redis for caching

Redis was also useful for caching data that does not need to be repeatedly recomputed or fetched from the database.

The basic idea is:

```text
Request
   │
   ▼
Check Redis
   │
   ├── Cache hit ─────► Return cached data
   │
   └── Cache miss
           │
           ▼
        Database
           │
           ▼
       Store in Redis
           │
           ▼
         Return
```

Caching is useful, but it also introduced another problem: **cache invalidation**.

If the underlying database changes, cached information may become outdated.

That made me think more carefully about which data should actually be cached and when the cache should be refreshed or removed.

## Handling document storage

The application also supports document-related workflows.

Instead of storing uploaded files directly inside the application database, I used **Cloudflare R2** for object storage.

The architecture becomes:

```text
Application
     │
     ├── Metadata ─────► Database
     │
     └── File ─────────► Cloudflare R2
```

The database can store information about a document while the actual file lives in object storage.

This keeps large binary files separate from relational application data.

## Frontend with Vue.js

The frontend is built with **Vue.js** and communicates with the Flask backend using Axios.

Vue Router handles navigation between pages, while the application's authentication state determines which areas of the interface should be available to each role.

The frontend therefore acts mainly as the user interface, while authorization and important business rules remain enforced by the backend.

This separation was important because frontend checks alone should never be treated as a security boundary.

## What I learned

MediCare was useful because it moved beyond a simple CRUD application and forced me to think about how different parts of a full-stack system interact.

A few things stood out:

* Authentication and authorization are different problems and both need to be enforced on the backend.
* Long-running work should not necessarily execute inside an HTTP request.
* Celery and Redis provide a clean way to move work into background workers.
* Caching can improve performance, but cached data also needs a clear invalidation strategy.
* Object storage is a better fit for files than storing large binary data directly in a relational database.
* Separating the frontend and backend creates clearer boundaries between presentation, API logic, and persistence.
* API documentation becomes increasingly valuable as the number of endpoints grows.

The biggest lesson was that building a larger web application is less about any individual framework and more about deciding **which part of the system should be responsible for each piece of work**.

MediCare gave me practical experience connecting a Vue frontend, Flask REST API, relational database, Redis, Celery workers, and cloud object storage into a single application.
