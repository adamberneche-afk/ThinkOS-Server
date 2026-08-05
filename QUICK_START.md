# RCRT Quick Start Guide

## One-Click Setup 🚀

1. **Clone the repository**:
   ```bash
   git clone <repository-url>
   cd breadcrums
   ```

2. **Run the setup script**:
   ```bash
   ./setup.sh
   ```
   
   The script automatically detects your platform:
   - 🍎 **Mac**: Native ARM64 build on Apple Silicon, x64 on Intel
   - 🐧 **Linux**: Optimized for your architecture
   - 🪟 **Windows**: Full support via WSL or Git Bash

That's it! The script will:
- ✅ Check all prerequisites
- ✅ Build the browser extension
- ✅ Generate missing files (Cargo.lock)
- ✅ Start all Docker services
- ✅ Validate everything is working
- ✅ Guide you to install the browser extension

## Manual Setup (if needed)

If the automated script doesn't work, follow these steps:

### Prerequisites
- Docker Desktop (running)
- Git
- Rust/Cargo toolchain
- curl

### Steps
1. Generate Cargo.lock: `cargo generate-lockfile`
2. Start services: `docker compose up -d`
3. Wait for services to be healthy: `docker compose ps`

## Service URLs

Once running, access these URLs:

| Service | URL | Purpose |
|---------|-----|---------|
| **Dashboard** | http://localhost:8082 | RCRT Dashboard v2 |
| **RCRT Server** | http://localhost:8081 | Main API server |
| **Builder UI** | http://localhost:3000 | Visual workflow builder |
| **Browser Extension** | `extension/dist/` | Chrome/Edge extension |
| **Database** | localhost:5432 | PostgreSQL + pgvector |
| **NATS** | localhost:4222 | Message broker |

## 🧩 Browser Extension

After setup, install the browser extension for chat access:

1. **Quick install**: Run `./install-extension.sh`
2. **Manual install**:
   - Open Chrome/Edge extensions page
   - Enable Developer mode
   - Load unpacked → select `extension/dist/`
   - Click the RCRT icon to chat!

## Environment Variables (Optional)

Copy `env.example` to `.env` and configure:

```bash
cp env.example .env
# Edit .env with your values
```

Key variables:
- `LOCAL_KEK_BASE64` - Encryption key for secrets
- `OPENROUTER_API_KEY` - For LLM features

## Troubleshooting

### Services not starting?
```bash
# Check service status
docker compose ps

# View logs
docker compose logs -f

# Restart a specific service
docker compose restart <service-name>
```

### Health checks failing?
```bash
# Test endpoints manually
curl http://localhost:3000/api/health  # Builder
curl http://localhost:8081/health      # RCRT Server
curl -I http://localhost:8082          # Dashboard
```

### Complete reset
```bash
# Stop and remove everything
docker compose down -v

# Rebuild and start
docker compose up -d --build
```

## Development

### View logs
```bash
docker compose logs -f [service-name]
```

### Stop services
```bash
docker compose down
```

### Rebuild after changes
```bash
docker compose up -d --build
```

## Need Help?

See [docs/README.md](./docs/README.md) for the full documentation index, or [docs/DEPLOYMENT.md](./docs/DEPLOYMENT.md) for deployment troubleshooting.
