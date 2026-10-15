# SearXNG with Redis Stack

<a href="https://www.aihonestybadge.com" target="_blank" rel="noopener"><img src="https://www.aihonestybadge.com/badges/ai-assisted-large.svg" alt="AI Assisted" height="28"></a>

A privacy-respecting, metasearch engine stack aggregating results from more than 70 search services without tracking or profiling users. This deployment is pre-configured with Redis for caching, enabled API endpoints (`json`, `csv`, `rss`), and bot detection settings.

---

## Features

- **Privacy-First**: No user tracking, no profiling, and no query logging.
- **Aggregated Search**: Fetches results from major providers (Google, Bing, DuckDuckGo, Wikipedia, etc.).
- **Multi-Format Support**: Pre-configured to serve HTML, JSON, CSV, and RSS feeds.
- **Embedded Image Proxy**: Shields user IP addresses when viewing external image results.
- **Integrated Redis Cache**: Accelerates frequent requests and rate-limiting lookups.
- **Coolify & Standalone Docker Ready**: Configured for 1-click PaaS deployments (Coolify) or vanilla Docker Compose.

---

## Directory Structure

```text
.
├── docker-compose.yml    # Service definitions (SearXNG + Redis)
├── .env.example          # Sample environment variables
├── .env                  # Your active environment configuration (git-ignored)
└── README.md             # Project documentation
```

---

## Environment Variables

Copy `.env.example` to `.env` before running the services:

```bash
cp .env.example .env
```

| Variable | Required | Default | Description |
| :--- | :---: | :--- | :--- |
| `SERVICE_URL_SEARXNG_8080` | **Yes** | — | Fully qualified URL for the instance (e.g., `https://search.example.com` or `http://localhost:8080`). Mapped to both `BASE_URL` and `SEARXNG_URL`. |
| `SERVICE_PASSWORD_SEARXNGSECRET` | **Yes** | — | Encryption key for session management and token generation. |
| `INSTANCE_NAME` | No | `coolify` | Name displayed in the UI banner and page titles. |
| `SEARXNG_BIND_ADDRESS` | No | `0.0.0.0` | Container internal listening interface. |

### Generating a Secret Key

Generate a secure 32-byte secret key with `openssl`:

```bash
openssl rand -hex 32
```

Paste the output as the value for `SERVICE_PASSWORD_SEARXNGSECRET` in your `.env` file.

---

## Deployment & Setup

### Option 1: Standalone Docker Compose

1. **Verify or create the environment file:**
   ```bash
   cp .env.example .env
   # Edit .env with your preferred text editor
   nano .env
   ```

2. **Start the containers in detached mode:**
   ```bash
   docker compose up -d
   ```

3. **Check container health status:**
   ```bash
   docker compose ps
   ```
   Both `searxng` and `redis` should display `healthy`.

4. **Access the application:**
   - Web UI: `http://localhost:8080` (or the URL defined in `SERVICE_URL_SEARXNG_8080`).

---

### Option 2: Coolify Deployment

1. Create a new service in Coolify and choose **Docker Compose**.
2. Paste the contents of your `docker-compose.yml`.
3. In the service's **Environment Variables** tab, define:
   - `INSTANCE_NAME`: Name for your instance.
   - `SERVICE_URL_SEARXNG_8080`: Assigned automatically or manually set to your domain.
   - `SERVICE_PASSWORD_SEARXNGSECRET`: A generated random hex string.
4. Click **Deploy**.

---

## API Usage Examples

SearXNG allows programmatically querying search results across different output formats:

### 1. JSON Search Request

```bash
curl -G "http://localhost:8080/search" \
  --data-urlencode "q=docker compose" \
  --data-urlencode "format=json"
```

Sample JSON response structure:
```json
{
  "query": "docker compose",
  "number_of_results": 42,
  "results": [
    {
      "url": "https://docs.docker.com/compose/",
      "title": "Docker Compose overview",
      "content": "Docker Compose is a tool for defining and running multi-container Docker applications...",
      "engine": "google",
      "category": "general"
    }
  ]
}
```

### 2. OpenSearch / RSS Feed

Fetch queries directly via RSS readers:
```bash
curl -G "http://localhost:8080/search" \
  --data-urlencode "q=linux kernel" \
  --data-urlencode "format=rss"
```

---

## Configuration Reference

The compose file embeds configuration overrides into `/etc/searxng/`:

- **`settings.yml`**:
  - `use_default_settings: true`: Inherits defaults from upstream SearXNG while overriding specific sections.
  - `server.limiter: false`: Disables default basic limiter in favor of the custom `limiter.toml` bot-detection rules.
  - `server.image_proxy: true`: Proxies images through your server so client IP addresses are not leaked to external media hosts.
  - `search.formats`: Enables output in `html`, `csv`, `json`, and `rss`.
- **`limiter.toml`**:
  - `[botdetection.ip_limit] link_token = true`: Enforces token verification to mitigate bot scrapes while keeping human searches seamless.

---

## Operations & Maintenance

### Viewing Logs

```bash
# SearXNG container logs
docker compose logs -f searxng

# Redis cache logs
docker compose logs -f redis
```

### Clearing the Redis Cache

If you need to flush cached search metadata or limiter tracking:

```bash
docker compose exec redis redis-cli flushall
```

### Stopping the Services

```bash
# Stop containers without removing persistent data
docker compose down

# Stop containers and erase Redis persistent volume
docker compose down -v
```
