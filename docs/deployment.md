# Deployment

This page covers how to run Skyvern in three environments: local development, Docker Compose, and Kubernetes. It also covers the environment variables and database setup that every deployment needs.

## Requirements

All deployments share these dependencies:

- PostgreSQL 14+ — the primary database
- Python 3.11 or 3.12 — the backend runtime
- Chromium/Chrome — the browser (installed by the Docker images, or via `playwright install`)
- An LLM API key (OpenAI, Anthropic, Gemini, etc.)

Redis is listed in `docker-compose.yml` but is optional for basic operation. It's used for rate limiting and caching.

## Local development

The quickest local setup uses the `skyvern` Python package:

```bash
pip install skyvern
skyvern quickstart
```

The wizard asks for your LLM provider and API key, writes a `.env` file, runs database migrations, and starts the backend and UI. When it's done:
- Backend API: `http://localhost:8000`
- Web UI: `http://localhost:8080`

To run components separately:

```bash
# 1. Start PostgreSQL (however you prefer — Docker, Homebrew, native)
#    Database URL: postgresql://skyvern@localhost/skyvern

# 2. Run database migrations
python -m alembic upgrade head

# 3. Start the backend
python -m skyvern.forge

# 4. Start the frontend (optional)
cd skyvern-frontend && npm install && npm run dev
```

The backend listens on port `8000` by default (configurable with `PORT`).

## Docker Compose

Docker Compose is the recommended setup for local development with a clean environment, or for small production deployments.

**1. Clone the repository:**

```bash
git clone https://github.com/skyvern-ai/skyvern.git
cd skyvern
```

**2. Configure your LLM:**

Copy the example and fill in your API key. At minimum, enable one provider and set `LLM_KEY`:

```bash
cp .env.example .env
```

For OpenAI:
```
ENABLE_OPENAI=true
LLM_KEY=OPENAI_GPT4O
OPENAI_API_KEY=sk-...
```

For Anthropic:
```
ENABLE_ANTHROPIC=true
LLM_KEY=ANTHROPIC_CLAUDE3.5_SONNET
ANTHROPIC_API_KEY=sk-ant-...
```

**3. Start services:**

```bash
docker-compose up
```

This starts:
- PostgreSQL 14 on an internal network (not exposed)
- Skyvern backend on `http://localhost:8000` with VNC streaming on port 6080
- Skyvern UI on `http://localhost:8080`

Migrations run automatically on startup. An API key is created and written to `.streamlit/secrets.toml` on first run.

**Image references:**
- Backend: `public.ecr.aws/skyvern/skyvern:latest`
- Frontend: `public.ecr.aws/skyvern/skyvern-ui:latest`

**Artifact volumes:**

The compose file mounts local directories for persistent storage:
```yaml
volumes:
  - ./artifacts:/data/artifacts
  - ./videos:/data/videos
  - ./har:/data/har
  - ./log:/data/log
```

## Building from source

To build your own Docker image instead of using the public ECR image:

```bash
# Backend
docker build -t skyvern-backend .

# Frontend
docker build -f Dockerfile.ui -t skyvern-frontend .
```

The backend `Dockerfile` is a two-stage build: it compiles Python dependencies with `uv`, then installs Playwright and Chromium in a slim Python 3.11 image. The entrypoint script (`entrypoint-skyvern.sh`) runs migrations, starts Xvfb (virtual display), launches x11vnc for browser streaming, and starts the FastAPI server.

## Kubernetes

The `kubernetes-deployment/` directory contains production-ready manifests. This is a basic setup suitable for internal/private network deployments.

**Prerequisite:** Run `skyvern quickstart` locally first to generate your `.env` file. Then copy those values into `kubernetes-deployment/backend/backend-secrets.yaml`.

**1. Apply the manifests:**

```bash
cd kubernetes-deployment
./k8s-deploy.sh
```

Or manually:

```bash
kubectl apply -f namespace.yaml
kubectl apply -f postgres/
kubectl apply -f backend/
kubectl apply -f frontend/
kubectl apply -f ingress.yaml
```

**2. Get your API key:**

On first deployment, the backend creates an organization and API key. Get it from the UI (Settings → Copy API Key), then add it to `frontend/frontend-secrets.yaml`:

```yaml
VITE_SKYVERN_API_KEY: "your-api-key"
```

Re-apply and restart frontend pods:

```bash
kubectl apply -f frontend/
kubectl delete pod -n skyvern -l app=skyvern-frontend
```

**Architecture:** The Kubernetes deployment uses a single backend replica and a single frontend replica. The backend connects to PostgreSQL via the internal service DNS. Artifacts are stored on `hostPath` volumes (`/data/artifacts`, `/data/videos`, etc.) — for production, replace these with persistent volumes or S3.

**Ingress:** The included `ingress.yaml` uses Traefik. For other ingress controllers, update the annotations. The default routing:
- `/api/` → backend port 8000
- `/artifacts/` → frontend port 9090
- `/` → frontend port 8080

For TLS, uncomment the `tls` section in `ingress.yaml` and set your certificate secret name.

**Warning:** Do not expose Skyvern to the internet without adding authentication. The Kubernetes README explicitly recommends this deployment only for private networks.

## Database setup

Skyvern uses PostgreSQL with Alembic for schema management.

**Create the database:**

```sql
CREATE USER skyvern;
CREATE DATABASE skyvern OWNER skyvern;
```

**Run migrations:**

```bash
python -m alembic upgrade head
```

The Docker entrypoint runs migrations automatically on every start. To skip migrations (e.g., in rolling updates where schema is already current), set `ALLOWED_SKIP_DB_MIGRATION_VERSION` to the current Alembic revision.

## Environment variables

These are the most important settings. All are read from `.env` or the process environment.

**Required:**

| Variable | Description |
|---|---|
| `DATABASE_STRING` | PostgreSQL connection URL (async driver) |
| `LLM_KEY` | Active LLM model key (e.g., `OPENAI_GPT4O`, `ANTHROPIC_CLAUDE3.5_SONNET`) |
| `OPENAI_API_KEY` / `ANTHROPIC_API_KEY` / etc. | API key for your LLM provider |
| `SECRET_KEY` | JWT signing secret — generate a strong random value for production |

**LLM provider toggles** (set the one you're using to `true`):

```
ENABLE_OPENAI=true
ENABLE_ANTHROPIC=true
ENABLE_GEMINI=true
ENABLE_AZURE=true
ENABLE_BEDROCK=true
ENABLE_OLLAMA=true
ENABLE_OPENROUTER=true
```

**Storage:**

| Variable | Default | Description |
|---|---|---|
| `SKYVERN_STORAGE_TYPE` | `local` | `local`, `s3cloud`, or `azureblob` |
| `ARTIFACT_STORAGE_PATH` | `./artifacts` | Local path when `SKYVERN_STORAGE_TYPE=local` |
| `AWS_S3_BUCKET_ARTIFACTS` | `skyvern-artifacts` | S3 bucket name |
| `AWS_REGION` | `us-east-1` | AWS region for S3 |
| `AZURE_STORAGE_ACCOUNT_NAME` | `null` | Azure storage account |
| `AZURE_STORAGE_ACCOUNT_KEY` | `null` | Azure storage key |

**Server:**

| Variable | Default | Description |
|---|---|---|
| `PORT` | `8000` | Backend port |
| `ALLOWED_ORIGINS` | `["*"]` | CORS allowed origins — restrict this in production |
| `LOG_LEVEL` | `INFO` | Log verbosity: `DEBUG`, `INFO`, `WARNING`, `ERROR` |
| `JSON_LOGGING` | `false` | Emit structured JSON logs |
| `REDIS_URL` | `redis://localhost:6379/0` | Redis URL |

**Browser:**

| Variable | Default | Description |
|---|---|---|
| `BROWSER_TYPE` | `chromium-headful` | `chromium-headful` or `cdp-connect` |
| `BROWSER_WIDTH` | `1920` | Viewport width |
| `BROWSER_HEIGHT` | `1080` | Viewport height |
| `MAX_STEPS_PER_RUN` | `10` | Default step limit per task |

## Scaling considerations

Skyvern is stateless at the API layer — the database holds all state. To scale horizontally, run multiple backend instances behind a load balancer. Each instance manages its own browser processes, so browser state is node-local.

Session recordings and artifacts are stored in S3 or Azure Blob (set `SKYVERN_STORAGE_TYPE` accordingly) so all instances can access them.

The database connection pool is configured with `DATABASE_POOL_SIZE=5` and `DATABASE_POOL_MAX_OVERFLOW=10` per instance. Adjust these based on your PostgreSQL connection limits and number of replicas.
