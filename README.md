# SearXNG with Redis Stack

<a href="https://www.aihonestybadge.com" target="_blank" rel="noopener"><img src="https://www.aihonestybadge.com/badges/ai-generated-large.svg" alt="AI Generated" height="28"></a>

A privacy-respecting metasearch engine stack aggregating results from over 70 search services without tracking or profiling users. Configured with a Redis caching layer, JSON/CSV/RSS API output formats, and link token bot protection.

---

## Repository Structure

```text
.
├── openship.json         # OpenShip configuration & edge port routing
├── docker-compose.yml    # SearXNG and Redis service definitions
├── settings.yml          # SearXNG engine and proxy overrides
├── limiter.toml          # Bot detection rules
├── .env.example          # Sample environment configuration
├── .env                  # Active environment file (git-ignored)
└── README.md             # Project documentation

```

---

## Environment Configuration

Generate a secure 32-byte secret key:

```bash
openssl rand -hex 32

```

Copy the template file to `.env`:

```bash
cp .env.example .env

```

| Variable | Required | Default | Description |
| --- | --- | --- | --- |
| `BASE_URL` / `SEARXNG_URL` | **Yes** | `http://localhost:8080` | Public domain of your instance (e.g., `https://search.example.com`). |
| `SEARXNG_SECRET` | **Yes** | — | Cryptographic salt generated via `openssl rand -hex 32`. |
| `INSTANCE_NAME` | No | `SearXNG` | Instance label rendered in UI banners and metadata. |
| `SEARXNG_BIND_ADDRESS` | No | `0.0.0.0` | Container network listening interface. |

---

## Deployment Methods

### 1. OpenShip Deployment (Recommended)

#### Via OpenShip CLI

1. Initialize the repository link:
```bash
openship init

```


2. Set your production secrets:
```bash
openship env set BASE_URL="[https://search.yourdomain.com](https://search.yourdomain.com)"
openship env set SEARXNG_URL="[https://search.yourdomain.com](https://search.yourdomain.com)"
openship env set SEARXNG_SECRET="$(openship crypto rand 32)"
openship env set INSTANCE_NAME="SearXNG"

```


3. Deploy the application:
```bash
openship deploy

```



#### Via OpenShip Desktop / Web Dashboard

1. Connect your server or select OpenShip Cloud.
2. Link your Git repository. OpenShip detects `openship.json` and `docker-compose.yml`.
3. In the project dashboard under **Environment Variables**, add `BASE_URL`, `SEARXNG_URL`, and `SEARXNG_SECRET`.
4. Under **Domains**, add your custom domain. OpenShip provisions the Let's Encrypt certificate and routes traffic to port `8080`.
5. Click **Deploy**.

---

### 2. Standalone Docker Compose

1. Populate your active `.env` file:
```bash
cp .env.example .env

```


2. Start the services:
```bash
docker compose up -d

```


3. Verify status:
```bash
docker compose ps

```


4. Access via `http://localhost:8080`.

---

### 3. Coolify Deployment

1. Create a **Docker Compose** service pointing to this repository.
2. Under the service's **Environment Variables** tab, define:
* `BASE_URL` or `SERVICE_URL_SEARXNG_8080`
* `SEARXNG_SECRET` or `SERVICE_PASSWORD_SEARXNGSECRET`
* `INSTANCE_NAME`


3. Click **Deploy**.

---

## API Querying

Query search results programmatically via JSON:

```bash
curl -G "http://localhost:8080/search" \
  --data-urlencode "q=docker compose" \
  --data-urlencode "format=json"

```

Query search results via RSS feed:

```bash
curl -G "http://localhost:8080/search" \
  --data-urlencode "q=linux kernel" \
  --data-urlencode "format=rss"

```

---

## Operations & Maintenance

### Stream Container Logs

```bash
docker compose logs -f searxng
docker compose logs -f redis

```

### Flush Redis Cache

```bash
docker compose exec redis redis-cli flushall

```

### Teardown

```bash
# Stop containers preserving data
docker compose down

# Stop containers and remove Redis data volume
docker compose down -v
```
