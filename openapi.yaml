# GitHub Issues Gateway

This service wraps GitHub Issues for one configured repository. It supports issue create/list/get/update, comments, verified `issues` and `issue_comment` webhooks, durable local event history, OpenAPI 3.1, tests, health checks, rate-limit translation, and Docker.

## Run

```bash
cp .env.example .env
# edit GITHUB_TOKEN, GITHUB_OWNER, GITHUB_REPO, and WEBHOOK_SECRET
make install
make run
```

Docker:

```bash
docker build -t github-issues-gateway .
docker run --rm -p 8000:8000 --env-file .env -v "$PWD/data:/service/data" github-issues-gateway
```

Required variables are `GITHUB_TOKEN`, `GITHUB_OWNER`, `GITHUB_REPO`, `WEBHOOK_SECRET`, and `PORT`. Use a fine-grained PAT limited to the test repository with Issues read/write permission. `EVENT_STORE_PATH` defaults to `./data/events.sqlite3`.

## API examples

```bash
curl -i -X POST localhost:8000/issues -H 'content-type: application/json' -d '{"title":"Demo","body":"Created through gateway","labels":["bug"]}'
curl 'localhost:8000/issues?state=open&per_page=20'
curl localhost:8000/issues/1
curl -X PATCH localhost:8000/issues/1 -H 'content-type: application/json' -d '{"title":"Renamed","state":"closed"}'
curl -X POST localhost:8000/issues/1/comments -H 'content-type: application/json' -d '{"body":"A gateway comment"}'
curl localhost:8000/issues/1/comments
curl localhost:8000/events
curl localhost:8000/healthz
```

`DELETE` is intentionally absent: closing an issue is `PATCH /issues/{number}` with `{"state":"closed"}`. Full request/response models and examples are in [openapi.yaml](openapi.yaml).

## Webhooks

In the repository’s GitHub Settings, add a webhook pointing to `https://your-public-host/webhook`, choose `application/json`, set the same `WEBHOOK_SECRET`, and subscribe to Issues and Issue comments. For local demos, use ngrok, Cloudflared, or smee. GitHub retries failed deliveries; the delivery ID makes reprocessing safe. Inspect accepted summaries with `GET /events`.

## Tests and live integration

Run `make test` for unit/HTTP-mocked tests and coverage. Optional live tests belong in `tests/integration` and should be gated by `RUN_LIVE_TESTS=1`; configure the same environment variables before running them against a dedicated repository. Never commit `.env` or tokens.
