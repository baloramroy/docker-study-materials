# Docker Compose: environment Variable
Docker Compose does **not** set environment variables automatically. You must **explicitly** define them in your `compose.yaml`.

There are two main ways:

---

## 1. Using the `environment:` attribute
The `environment:` section in `docker-compose.yml` is used to pass environment variables inside a container.
You can write them in 2 ways:
- Map Format (`key: value`)
- Array Format (`KEY=value`)

Both produce the same result.

---

### 1.1 Map Format (Recommended)

**1.1.1 NGINX with a Custom Port Example**\
Your app needs to pass the port number to NGINX.
```yaml
environment:
  SERVER_NAME: "example.com"
  APP_PORT: "8080"
  ENABLE_CACHE: "true"
```
**Inside container:**
- `SERVER_NAME=example.com` → set inside container
- `APP_PORT=8080` → set inside container
- `ENABLE_CACHE=true` → quoted so YAML does **not** treat it as boolean

#

**1.1.2 Host-Provided Variable Example**

You want the container to read the environment variable from your host machine, such as an API key or a token.

**Host machine:**
```bash
export API_TOKEN="12345-XYZ-SECRET"
```

**docker-compose.yml:**
```yaml
environment:
  API_TOKEN:        # no value → try to get from host
  APP_MODE: "production"
```
**Inside container:**
- `APP_MODE=production`
- `API_TOKEN=12345-XYZ-SECRET` (if exported on host)
If not exported → `API_TOKEN` will **not** exist inside the container.

---

### 1.2 Array Format

**1.2.1 NGINX Example**

Your app needs to pass the port number to NGINX.
```yaml
environment:
  - SERVER_NAME=example.com
  - APP_PORT=8080
  - ENABLE_CACHE=true
```
**Inside container:**
- `SERVER_NAME=example.com` → set inside container
- `APP_PORT=8080` → set inside container
- `ENABLE_CACHE=true` → quoted so YAML does **not** treat it as boolean

#

**1.2.2 Host Variable Example**

**Host machine:**
```bash
export APP_SECRET="superkey"
```
**`docker-compose.yml`:**
```yaml
environment:
  - APP_ENV=production
  - APP_SECRET
```
**Inside container:**
- `APP_ENV=production`
- `APP_SECRET=superkey`
**Note:** If the host doesn’t have `APP_SECRET` → it does **NOT** exist inside the container.

---

## 2. Using `env_file:`
You can keep environment variables in a separate `.env` file.

### 2.1 Single Env File Example

**webapp.env**
```env
DEBUG=true
MYSQL_HOST=localhost
```
**compose.yaml**
```yaml
services:
  webapp:
    env_file: "webapp.env"
```
**Benefits:**
- Cleaner compose file
- Reuse same `.env` file for multiple services
- Works with `docker run --env-file`
- Keeps sensitive variables outside YAML (better organization)

### 2.2 Multiple env files
```yaml
env_file:
  - default.env
  - override.env
```
>Later files override earlier ones.

### 2.3 Optional env files (Compose ≥ 2.24.0)
```yaml
env_file:
  - path: ./default.env
    required: true
  - path: ./override.env
    required: false
```
>If `override.env` is missing, no error.

---

## 3. Set environment variables at runtime
You can pass them only when running a service:
```bash
docker compose run -e DEBUG=1 web python console.py
```

Or pass from your shell:
```bash
docker compose run -e DEBUG web python console.py
```

---