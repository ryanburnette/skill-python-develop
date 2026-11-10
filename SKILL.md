---
name: python-develop
description: Python development via Docker. Use when writing project Python scripts, tools, or apps. Run project runtimes, dependencies, tests, and builds in Docker; installed agent tooling may run on the host.
---

## Core Rule

Run project Python scripts, dependencies, tests, and builds inside Docker.
Do not install project runtimes or packages, or create project virtualenvs, on
the host.

Existing agent and host tools, including bundled Python helpers, may run on the
host. Installing or updating such tooling is allowed only when requested or
required by an authorized setup task, following that tool's instructions. This
exception does not permit installing project dependencies on the host.

## Running Python

One-off script or command:

```sh
docker run --rm -v "$PWD":/app -w /app python:3.13-slim python script.py
```

With dependencies, use a `Dockerfile`:

```dockerfile
FROM python:3.13-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY . .
CMD ["python", "main.py"]
```

Build and run:

```sh
docker build -t my-app .
docker run --rm my-app
```

With environment variables:

```sh
docker run --rm -v "$PWD":/app -w /app -e API_KEY=... python:3.13-slim python script.py
# or from a file:
docker run --rm -v "$PWD":/app -w /app --env-file .env python:3.13-slim python script.py
```

Interactive REPL:

```sh
docker run --rm -it python:3.13-slim python
```

## Python Version

Use `python:3.13-slim` unless the project specifies otherwise. Prefer `-slim`
over full images. Never use `-alpine` for Python — C extensions often fail.

## Dependencies

List all dependencies in `requirements.txt` with pinned versions:

```
requests==2.32.3
httpx==0.27.2
```

Generate a pinned lockfile inside the container:

```sh
docker run --rm -v "$PWD":/app -w /app python:3.13-slim \
  sh -c "pip install -r requirements.txt && pip freeze > requirements.lock.txt"
```

## docker-compose

For multi-service projects or persistent dev workflows:

```yaml
services:
  app:
    build: .
    volumes:
      - .:/app
    working_dir: /app
    command: python main.py
```

Run with: `docker compose run --rm app`

## Guardrails

- Do not install project Python runtimes, packages, or virtualenvs on the host; the host-tooling exception above applies only to authorized agent/host tool maintenance
- ALWAYS use `python:3.13-slim` or a project-specified image
- ALWAYS pin dependency versions in `requirements.txt`
- ALWAYS use `--rm` for one-off containers to avoid accumulation
