# beacon

Poll and record the health of your internal HTTP services from a single lightweight API.

## Quick start

```bash
cp .env.example .env
docker compose up -d
```

Register a service to monitor:

```bash
curl -s -X POST http://localhost:8080/services \
  -H "Content-Type: application/json" \
  -d '{"name": "payments-api", "url": "http://payments:8000/health", "interval_seconds": 60}'
```

Check its current status:

```bash
curl -s http://localhost:8080/services/1/health | jq .
```

## Configuration

| Variable                 | Required | Default    | Description                                |
|--------------------------|----------|------------|--------------------------------------------|
| `DATABASE_PATH`          | no       | `beacon.db`| Path to the SQLite database file           |
| `CHECK_INTERVAL_SECONDS` | no       | `60`       | Default polling interval for new services  |
| `LOG_LEVEL`              | no       | `info`     | One of: debug, info, warn, error           |
| `PORT`                   | no       | `8080`     | Port the API listens on                    |

## Development setup

```bash
python -m venv .venv && source .venv/bin/activate
pip install -e ".[dev]"
pytest
```

See [ONBOARDING.md](ONBOARDING.md) for a full contributor guide and [docs/api.md](docs/api.md) for the full API reference.

## License

MIT
