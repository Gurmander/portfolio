---
title: "Adding Practical CI Checks with GitHub Actions"
date: 2026-07-25
description: "How I used automated testing, linting, formatting checks, and builds to catch problems before merging code."
categories:
  - DevOps
tags:
  - GitHub Actions
  - CI/CD
  - Pytest
  - Ruff
  - Python
draft: false
---

As my projects became larger and involved more contributors, manually checking everything before every merge became increasingly unreliable.

This is where I started using **GitHub Actions** as part of the development workflow.

The goal was not to build a complicated DevOps platform.

It was simply to answer one question automatically:

> Does this version of the project still build and pass its basic quality checks?

## The basic workflow

A typical CI pipeline looked roughly like this:

```text
Push / Pull Request
        │
        ▼
GitHub Actions
        │
        ├── Install dependencies
        ├── Run linter
        ├── Check formatting
        ├── Run tests
        └── Build application
                │
                ▼
             Pass / Fail
```

Instead of discovering basic errors after merging code, they can be detected as soon as changes are pushed.

## Testing the backend

For Python backends, I used **Pytest** for automated tests.

The CI runner installs the project's dependencies and then executes the test suite:

```yaml
- name: Run tests
  run: pytest -v
```

This means the tests run in a clean environment rather than relying only on my local machine.

That is useful because something that works locally may fail when dependencies or configuration differ.

## Linting with Ruff

I also used **Ruff** for Python linting and formatting checks.

For example:

```yaml
- name: Run Ruff
  run: ruff check .

- name: Check formatting
  run: ruff format --check .
```

These checks help catch issues such as:

* unused imports
* formatting inconsistencies
* common Python mistakes
* style problems

The important part is that these rules are applied consistently to every contribution.

## Frontend checks

For projects with a separate frontend, the workflow can use another job:

```text
Backend CI
   │
   ├── Python dependencies
   ├── Ruff
   └── Pytest

Frontend CI
   │
   ├── Node dependencies
   └── Build
```

Running them independently also makes failures easier to identify.

If the frontend build fails, I immediately know which part of the pipeline needs attention.

## CI and deployment

For the Syngenta project, GitHub Actions was also part of the deployment workflow.

After changes reached the appropriate branch and passed the required checks, the application could be deployed to its hosting environments.

This introduced a useful distinction:

```text
Continuous Integration
        │
        ▼
Is the code valid?

Continuous Deployment
        │
        ▼
Should the validated code be deployed?
```

The two ideas are related, but they solve different problems.

## Why CI became useful in team projects

CI becomes especially valuable when multiple people are contributing.

Without it:

```text
Developer A → works locally
Developer B → works locally
Merge
   ↓
Something breaks
```

With CI:

```text
Developer A
    │
    ▼
Pull Request
    │
    ▼
Automated Checks
    │
    ├── Tests
    ├── Lint
    └── Build
    │
    ▼
Merge
```

It does not guarantee that the software is bug-free, but it creates a consistent minimum quality gate.

## What I learned

My main takeaway was that CI does not need to be complicated to be useful.

Even a small workflow that runs:

```text
lint
test
build
```

can prevent a surprising number of problems.

It also encourages better engineering habits because the repository itself defines what must succeed before code is considered ready.

For my projects, GitHub Actions became less about "DevOps tooling" and more about making everyday development more predictable.
