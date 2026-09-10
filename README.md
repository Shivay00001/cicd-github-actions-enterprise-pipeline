# cicd-github-actions-enterprise-pipeline

A Go-based reference service and pipeline scaffold intended to demonstrate an enterprise-style CI/CD workflow with GitHub Actions, Docker containerization, and pre-commit hygiene hooks.

> **Repository:** `github.com/Shivay00001/cicd-github-actions-enterprise-pipeline`
> **Language:** Go 1.20 · **Runtime:** Docker (Alpine-based) · **Port:** `8080`

---

## 🚀 Overview

This repository contains a minimal, self-contained Go HTTP service packaged as a Docker image, along with the scaffolding (Makefile, git hooks, `.gitignore` secret hygiene) that an enterprise CI/CD pipeline would build upon. The service itself is intentionally simple — a single HTTP endpoint that reports system liveness with a timestamp — making it an ideal "canary" application for validating that a pipeline (build → test → containerize → deploy) is functioning end-to-end.

## ✨ Features

- **Lightweight Go HTTP service** — zero external dependencies (stdlib only: `net/http`, `log`, `fmt`, `time`), so builds are fast and reproducible.
- **Multi-stage-ready Dockerfile** — compiles the binary inside a `golang:1.20-alpine` container; no local Go toolchain required to run.
- **Secret hygiene** — an aggressive `.gitignore` blocks `.env` files, API keys, service-account JSON, and credential files from ever being committed.
- **Git hooks bootstrap** — `scripts/setup-hooks.sh` installs a `pre-commit` hook into `.git/hooks/`.
- **Makefile task runner** — standard `lint`, `test`, and `clean` targets as pipeline entry points.
- **Custom commercial license** — free for non-commercial use; revenue-share and enterprise terms apply (see [License](#-license)).

## 🏗️ Architecture / How It Works

The repository is deliberately flat and consists of the following components:

```
.
├── main.go                  # The HTTP service (entry point)
├── go.mod                   # Go module definition (Go 1.20, no external deps)
├── Dockerfile               # Container build definition
├── Makefile                 # lint / test / clean task runner
├── scripts/
│   └── setup-hooks.sh       # Installs the pre-commit git hook
├── .gitignore               # Secret & build-artifact exclusion rules
└── LICENSE                  # VisionQuantech Custom Commercial License
```

### Request Flow (`main.go`)

1. On startup, `main()` registers a single handler on the root path `/` using Go's default `http.ServeMux`.
2. The handler writes `System Operational: <current timestamp>` to the response — effectively a combined **health check / liveness probe**.
3. The server binds to **`:8080`** via `http.ListenAndServe`, logging startup and any fatal error through the standard `log` package.

Because there are no third-party imports, `go build` requires no module downloads, which keeps CI builds hermetic and fast.

### Container Build (`Dockerfile`)

```dockerfile
FROM golang:1.20-alpine
WORKDIR /app
COPY . .
RUN go build -o app
CMD ["./app"]
```

The image copies the full repository context, compiles a static-ish binary named `app`, and runs it as the container's entrypoint. Note: this is a **single-stage** build — see the [Workability Assessment](#-workability-assessment) for the implications.

### Pipeline Hooks

- **`make lint` / `make test`** — currently echo placeholders; these are the seams where `golangci-lint` and `go test ./...` would be wired in.
- **`scripts/setup-hooks.sh`** — copies `scripts/pre-commit` into `.git/hooks/pre-commit` and marks it executable, so lint/format checks can run before every commit.

## 🐳 Running with Docker (Any Laptop or Server)

This is the recommended way to run the service — no Go installation needed.

### Option A: Standard Dockerfile (works today)

```bash
# 1. Build the image
docker build -t cicd-enterprise-pipeline .

# 2. Run the container, mapping host port 8080 -> container port 8080
docker run -d -p 8080:8080 --name cicd-service cicd-enterprise-pipeline

# 3. Verify the service is live
curl http://localhost:8080
# Expected output: System Operational: 2026-01-01 12:00:00 ...
```

### Option B: Docker Compose

No `docker-compose.yml` is currently committed. If you prefer Compose, create one:

```yaml
services:
  app:
    build: .
    ports:
      - "8080:8080"
    restart: unless-stopped
```

Then run:

```bash
docker-compose up -d --build
```

### Stop / clean up

```bash
docker stop cicd-service && docker rm cicd-service
```

## 🛠️ Local Development

Requires Go 1.20+:

```bash
# Run directly
go run main.go

# Build a binary
go build -o app && ./app

# Pipeline tasks (currently placeholders)
make lint
make test

# Install git hooks
bash scripts/setup-hooks.sh
```

> ⚠️ **Note:** `scripts/setup-hooks.sh` expects a `scripts/pre-commit` file to exist. It is not currently present in the repository, so the script will fail until that hook file is added.

## 🔍 Workability Assessment

An honest evaluation of the current state:

**What works:**
- ✅ The Go service **compiles and runs correctly**. It is valid, dependency-free Go that responds on `:8080` as advertised.
- ✅ The Dockerfile is valid and will produce a working container image.
- ✅ The `.gitignore` secret-exclusion rules are genuinely good practice.

**What is incomplete or misleading:**
- ❌ **No GitHub Actions workflows exist.** Despite the repository name, there is no `.github/workflows/` directory — the actual CI/CD pipeline (the headline feature) is absent and must be authored.
- ❌ **`make lint` and `make test` are stubs** that only `echo` text. There are no real tests (`*_test.go` files) and no linter configuration.
- ❌ **`scripts/pre-commit` is missing**, so `setup-hooks.sh` will fail as shipped.
- ❌ **The Dockerfile is not production-optimized**: it is single-stage (the final image contains the full Go toolchain and source, ~300MB+), runs as root, and has no `HEALTHCHECK`. A multi-stage build with a `scratch`/`distroless` final stage and a non-root user is recommended.
- ❌ **No graceful shutdown** in `main.go` — no signal handling, timeouts, or structured logging.

**Verdict:** The repository is a **working skeleton / starter scaffold**, not a finished enterprise pipeline. The application code is functional and containerizable, but the "enterprise CI/CD" value proposition is aspirational — expect to add workflow YAML, real tests, a hardened Dockerfile, and the missing hook script before this is production-ready.

## 📄 License

**VisionQuantech Custom Commercial License** — Copyright (c) 2026 Shivay00001 / VisionQuantech.

- **Non-commercial / educational use:** free.
- **Individual revenue-generating use:** 15–30% gross revenue share.
- **Business / enterprise use:** requires a separate commercial license — contact **visionquantech@proton.me**.

See [LICENSE](./LICENSE) for full terms. The software is provided "AS IS", without warranty of any kind.