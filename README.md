# FastAPI Beyond CRUD

This is the source code for the [FastAPI Beyond CRUD](https://youtube.com/playlist?list=PLEt8Tae2spYnHy378vMlPH--87cfeh33P&si=rl-08ktaRjcm2aIQ) course, extended with a full CI/CD pipeline using GitHub Actions.

For more details on the original course, visit the project's [website](https://jod35.github.io/fastapi-beyond-crud-docs/site/).

---

## CI/CD Pipeline (Assignment 7 Additions)

### 1. Conventional Commits Enforcement (`.github/workflows/conventional-commits.yml`)

Triggered on every pull request. Checks that **all commits** follow the [Conventional Commits](https://www.conventionalcommits.org/) specification.

**If any commit violates the spec:**
- The PR is automatically **closed** with an explanatory comment.
- An **email notification** is sent via Ethereal.email SMTP.
- The workflow is marked as **failed**.

**Commit format required:**
```
<type>[optional scope]: <description>
```

Valid types: `feat` | `fix` | `docs` | `style` | `refactor` | `test` | `chore` | `perf` | `ci` | `build` | `revert`

### 2. Nightly Build (`.github/workflows/nightly-build.yml`)

Runs every day at **12:00 AM UTC** (configurable via cron). Can also be triggered manually via `workflow_dispatch`.

**Pipeline steps:**
1. **Run all tests** — uses mocked dependencies (no real DB needed in CI).
2. **If tests pass** → build Docker image and push to **GitHub Container Registry** (`ghcr.io`).
3. **If tests fail** → image is NOT pushed, and an **email notification** is sent.

**Image tags pushed to `ghcr.io`:**
- `nightly` — always points to the latest nightly build
- `nightly-YYYYMMDD` — date-stamped tag for each build
- `nightly-<sha>` — commit SHA tag for traceability

### 3. Email Notifications

Both workflows use [Ethereal.email](https://ethereal.email/) as the SMTP provider (free, no real delivery — messages viewable in the Ethereal web inbox).

---

## GitHub Secrets Required

Configure the following secrets in your repository under **Settings → Secrets and variables → Actions**:

| Secret | Description |
|---|---|
| `MAIL_USERNAME` | Ethereal SMTP username (e.g. `user@ethereal.email`) |
| `MAIL_PASSWORD` | Ethereal SMTP password |
| `MAIL_FROM` | Sender address (same as username for Ethereal) |
| `NOTIFICATION_EMAIL` | Email address that will receive failure notifications |

> **Get Ethereal credentials:** Go to [https://ethereal.email/create](https://ethereal.email/create) and click **Create Ethereal Account**. Copy the SMTP credentials shown.

---

## Running Locally with Docker

### Prerequisites
- Docker and Docker Compose installed

### Setup

1. Clone the repository:
    ```bash
    git clone <your-fork-url>
    cd fastapi-beyond-CRUD
    ```

2. Fill in your Ethereal email credentials in `.env`:
    ```bash
    # Edit the MAIL_* fields in .env with your Ethereal credentials
    nano .env
    ```
    All other values (database, Redis, JWT) are pre-configured for Docker.

3. Start all services:
    ```bash
    docker compose up
    ```
    This will:
    - Start PostgreSQL and wait until it is healthy
    - Run `alembic upgrade head` (database migrations)
    - Start the FastAPI app on port `8000`
    - Start the Celery worker
    - Start Redis

4. Access the API docs: [http://localhost:8000/api/v1/docs](http://localhost:8000/api/v1/docs)

---

## Running Tests

```bash
python -m pytest src/tests/ -v
```

Tests use mock objects and do **not** require a running database or Redis instance.

---

## Table of Contents

1. [Getting Started](#getting-started)
2. [CI/CD Pipeline](#cicd-pipeline-assignment-7-additions)
3. [GitHub Secrets Required](#github-secrets-required)
4. [Running Locally with Docker](#running-locally-with-docker)
5. [Running Tests](#running-tests)
6. [Contributing](#contributing)

## Getting Started (Manual Setup without Docker)

### Prerequisites
- Python >= 3.10
- PostgreSQL
- Redis

### Project Setup

1. Clone the project repository:
    ```bash
    git clone <your-fork-url>
    cd fastapi-beyond-CRUD
    ```

2. Create and activate a virtual environment:
    ```bash
    python3 -m venv env
    source env/bin/activate
    ```

3. Install the required dependencies:
    ```bash
    pip install -r requirements.txt
    ```

4. Set up environment variables by editing `.env` with your local settings.

5. Run database migrations:
    ```bash
    alembic upgrade head
    ```

6. Start the Celery worker (in a separate terminal):
    ```bash
    sh runworker.sh
    ```

7. Start the application:
    ```bash
    fastapi dev src/
    ```

## Contributing

I welcome contributions! Please ensure all commits follow the [Conventional Commits](https://www.conventionalcommits.org/) specification — PRs with non-conforming commits will be automatically closed by CI.
