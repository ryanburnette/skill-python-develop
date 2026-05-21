---
name: python-develop
description: Python development via Docker. Use when writing Python scripts, tools, or apps. Never install Python or packages on the host — always run in Docker to keep the local environment clean.
---

## Core Rule

Never install Python, pip, or any Python packages on the host machine. All Python
runs inside Docker. No `brew install python`, no `pip install`, no virtualenvs on
the host.

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

- NEVER run `brew install python` or any host-level Python install
- NEVER run `pip install` outside a container
- NEVER create virtualenvs on the host
- ALWAYS use `python:3.13-slim` or a project-specified image
- ALWAYS pin dependency versions in `requirements.txt`
- ALWAYS use `--rm` for one-off containers to avoid accumulation
