# Deployment Documentation Guide

## Goal

An operator should be able to deploy the application to production without access to the original developers. Docs must be complete enough to cover: environment setup, build, deploy, verify, and rollback.

## Sections to include

1. **Prerequisites** — infrastructure access, CLI tools, credentials needed
2. **Environment variables** — full list with descriptions (see detection below)
3. **Build** — how to produce the deployable artifact
4. **Deploy** — the deploy command or procedure
5. **Health check** — how to verify the deployment succeeded
6. **Rollback** — how to revert to the previous version
7. **Monitoring** — what to watch (logs, metrics endpoints)

Omit sections with no content. Never speculate about infrastructure that isn't in the code.

## Detect the deployment model

**Docker / Docker Compose** — `Dockerfile` present:
- Document `docker build` and `docker run` commands
- If `docker-compose.yml` exists, show `docker compose up -d`
- Document exposed ports from `EXPOSE` directives and `ports:` in compose

**Kubernetes** — `k8s/`, `kubernetes/`, `helm/`, or `*.yaml` with `kind: Deployment`:
- Show `kubectl apply -f k8s/` or `helm upgrade --install`
- Document namespace, replica count, resource limits from the manifests
- Note any required secrets or ConfigMaps

**Platform-as-a-Service** — detect by config files:

| File                      | Platform          | Deploy command              |
|---------------------------|-------------------|-----------------------------|
| `Procfile`                | Heroku / Render   | `git push heroku main`      |
| `render.yaml`             | Render            | Managed via dashboard       |
| `railway.toml`            | Railway           | `railway up`                |
| `fly.toml`                | Fly.io            | `fly deploy`                |
| `vercel.json` / `next.config.js` | Vercel     | `vercel --prod`             |
| `netlify.toml`            | Netlify           | Managed via dashboard / CLI |
| `app.yaml`                | Google App Engine | `gcloud app deploy`         |
| `.elasticbeanstalk/`      | AWS Elastic Beanstalk | `eb deploy`             |

**Serverless** — `serverless.yml`, `serverless.ts`, AWS SAM `template.yaml`, or Terraform with Lambda resources:
- Show `serverless deploy` or `sam deploy`
- Document function names and their triggers
- Note cold start implications if relevant

**CI/CD-managed** — if `.github/workflows/` contains a deploy job, document the trigger (branch push, tag, manual dispatch) and what it does. Don't replicate the workflow YAML — describe the process.

**Bare metal / VM** — if there's a `Makefile` with a `deploy` target, `scripts/deploy.sh`, or Ansible playbooks:
- Read the script and document the steps it performs
- Note any SSH access or server inventory required

## Environment variables

Read in this priority order:
1. `.env.example` / `.env.sample` — most explicit
2. `docker-compose.yml` → `environment:` or `env_file:` sections
3. Kubernetes manifests → `env:` and `envFrom:` in container specs
4. `serverless.yml` → `environment:` block
5. Application config loader (`config.py`, `config.ts`, `cmd/root.go`)

Format as a table:

```markdown
| Variable            | Required | Example                         | Description                         |
|---------------------|----------|---------------------------------|-------------------------------------|
| `DATABASE_URL`      | yes      | `postgres://user:pass@host/db`  | Primary database connection string  |
| `REDIS_URL`         | yes      | `redis://host:6379`             | Cache and session store             |
| `SECRET_KEY`        | yes      | 32-byte hex string              | Used to sign sessions and tokens    |
| `SENTRY_DSN`        | no       | `https://...@sentry.io/...`     | Error reporting (omit to disable)   |
| `LOG_LEVEL`         | no       | `info`                          | One of: debug, info, warn, error    |
```

## Build artifact detection

| Stack        | Build command           | Output artifact                  |
|--------------|-------------------------|----------------------------------|
| Node.js      | `npm run build`         | `dist/` or `.next/`             |
| Python       | (often no build step)   | source + `requirements.txt`     |
| Go           | `go build -o ./bin/app` | single binary in `bin/`         |
| Rust         | `cargo build --release` | `target/release/<name>`         |
| Java/Maven   | `./mvnw package`        | `target/*.jar`                  |
| Java/Gradle  | `./gradlew build`       | `build/libs/*.jar`              |
| PHP          | `composer install --no-dev` | source + `vendor/`          |
| C#/.NET      | `dotnet publish -c Release` | `bin/Release/net*/publish/` |

Read the actual `Makefile`, CI workflow, or `Dockerfile` `RUN` commands to confirm — projects often have custom build steps.

## Health check

Look for:
- A `/health`, `/healthz`, `/ping`, or `/ready` endpoint in the routes
- A `HEALTHCHECK` instruction in the Dockerfile
- A `livenessProbe` / `readinessProbe` in Kubernetes manifests

Document the URL, expected status code, and expected response body. If none exists, note its absence — this is useful information for operators.

## Monitoring

Look for these signals before writing this section:

| Signal | Document |
|---|---|
| `GET /health` / `/healthz` / `/metrics` endpoint | URL and expected response |
| `SENTRY_DSN` or similar in env / config | Error tracking is configured |
| Prometheus / Grafana in docker-compose | Metrics stack present |
| Log level env var (`LOG_LEVEL`, `RUST_LOG`, `LOG4J_*`) | How to tune verbosity |
| Structured JSON logging in the application code | Note that logs are machine-parseable |

Minimum to document when nothing else is configured:

```markdown
## Monitoring

**Logs**: `docker compose logs -f <service>` (or `kubectl logs -f <pod>`)

**Health**: `GET /health` → `{"status": "ok"}` (HTTP 200)
```

Only add metrics/tracing entries if they are actually configured in the codebase. Don't mention Datadog, Prometheus, or Sentry unless you find references to them.

## Rollback

Document the actual rollback procedure based on the deployment model:
- **Docker**: tag images; rollback = `docker pull image:<prev-tag> && docker compose up -d`
- **Kubernetes**: `kubectl rollout undo deployment/<name>`
- **Heroku**: `heroku releases:rollback`
- **Fly.io**: `fly deploy --image <prev-image>`
- **CI/CD**: re-run the deploy job against a previous git tag

If no rollback mechanism is evident, document the absence and suggest the minimal git-based pattern:

```markdown
## Rollback

No formal rollback mechanism is configured. To revert to a previous version:

1. Check out the previous git tag or commit: `git checkout <tag>`
2. Rebuild: `<build command>`
3. Restart: `<start command>`

Data (database, volumes) is unaffected by redeployment unless a schema migration ran.
```

Adapt the build/start commands to the actual deployment model. Note any migrations that would need to be reversed manually.
