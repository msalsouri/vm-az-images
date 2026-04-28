# Private AI Chat Server

Publisher/company context: **ALSOURI LLC**

This document records the initial build evidence for the Azure VM image candidate: **Private AI Chat Server**.

## Current VM state

Base VM hostname observed:

```text
ubuntu2204-min-vm
```

Application path:

```text
/opt/alsourillc/private-ai-chat
```

## Stack

- Docker Engine: installed and running
- Docker Compose: installed and working
- Ollama container: `alsourillc-ollama`
- Open WebUI container: `alsourillc-open-webui`
- Open WebUI exposed on TCP `3000`
- Ollama bound to localhost only on `127.0.0.1:11434`

## Confirmed service status

```text
NAME                    IMAGE                                COMMAND               SERVICE      CREATED         STATUS                   PORTS
alsourillc-ollama       ollama/ollama:latest                 "/bin/ollama serve"   ollama       3 minutes ago   Up 3 minutes             127.0.0.1:11434->11434/tcp
alsourillc-open-webui   ghcr.io/open-webui/open-webui:main   "bash start.sh"       open-webui   3 minutes ago   Up 3 minutes (healthy)   0.0.0.0:3000->8080/tcp, [::]:3000->8080/tcp
```

## Confirmed local HTTP test

```text
curl -I http://localhost:3000
```

Result:

```text
HTTP/1.1 200 OK
date: Mon, 27 Apr 2026 21:58:21 GMT
server: uvicorn
content-type: text/html; charset=utf-8
accept-ranges: bytes
content-length: 11099
last-modified: Fri, 24 Apr 2026 10:00:00 GMT
etag: "deeeda0e8dc37b3370da3c6be45220e4"
x-process-time: 0
```

## Confirmed Ollama API test

```text
curl http://localhost:11434/api/tags
```

Result at first checkpoint:

```json
{"models":[]}
```

Later checkpoint after pulling a model:

```json
{"models":[{"name":"llama3.2:3b","model":"llama3.2:3b","modified_at":"2026-04-27T22:00:42.134683884Z","size":2019393189,"digest":"a80c4f17acd55265feec403c7aef86be0c25983ab279d83f3bcd3abbcb5b8b72","details":{"parent_model":"","format":"gguf","family":"llama","families":["llama"],"parameter_size":"3.2B","quantization_level":"Q4_K_M"}}]}
```

## Confirmed listening ports

```text
tcp   LISTEN 0      4096       127.0.0.1:11434      0.0.0.0:*    users:(("docker-proxy",pid=4912,fd=8))
tcp   LISTEN 0      4096         0.0.0.0:22         0.0.0.0:*    users:(("sshd",pid=1627,fd=3),("systemd",pid=1,fd=73))
tcp   LISTEN 0      4096         0.0.0.0:3000       0.0.0.0:*    users:(("docker-proxy",pid=5006,fd=8))
tcp   LISTEN 0      4096            [::]:22            [::]:*    users:(("sshd",pid=1627,fd=4),("systemd",pid=1,fd=74))
tcp   LISTEN 0      4096            [::]:3000          [::]:*    users:(("docker-proxy",pid=5012,fd=8))
```

Security interpretation:

- SSH is listening on port `22`
- Open WebUI is listening on port `3000`
- Ollama is listening on `127.0.0.1:11434` only, not publicly exposed

## Confirmed UFW firewall status

```text
Status: active
Logging: on (low)
Default: deny (incoming), allow (outgoing), deny (routed)
New profiles: skip

To                         Action      From
--                         ------      ----
22/tcp (OpenSSH)           ALLOW IN    Anywhere
3000/tcp                   ALLOW IN    Anywhere
11434/tcp                  DENY IN     Anywhere
22/tcp (OpenSSH (v6))      ALLOW IN    Anywhere (v6)
3000/tcp (v6)              ALLOW IN    Anywhere (v6)
11434/tcp (v6)             DENY IN     Anywhere (v6)
```

## Confirmed project directory

```text
total 20
drwxr-xr-x 3 msalsouri msalsouri 4096 Apr 27 21:41 .
drwxr-xr-x 3 root      root      4096 Apr 27 21:28 ..
-rw------- 1 msalsouri docker     328 Apr 27 21:40 .env
-rw-rw-r-- 1 msalsouri docker     628 Apr 27 21:54 docker-compose.yml
drwxrwxr-x 2 msalsouri docker    4096 Apr 27 21:43 scripts
```

## Current Docker Compose file

```yaml
services:
  ollama:
    image: ollama/ollama:latest
    container_name: alsourillc-ollama
    restart: unless-stopped
    ports:
      - "127.0.0.1:${OLLAMA_PORT}:11434"
    volumes:
      - ollama_data:/root/.ollama

  open-webui:
    image: ghcr.io/open-webui/open-webui:main
    container_name: alsourillc-open-webui
    restart: unless-stopped
    depends_on:
      - ollama
    ports:
      - "${OPEN_WEBUI_PORT}:8080"
    environment:
      - OLLAMA_BASE_URL=${OLLAMA_BASE_URL}
      - WEBUI_SECRET_KEY=${WEBUI_SECRET_KEY}
    volumes:
      - open_webui_data:/app/backend/data

volumes:
  ollama_data:
  open_webui_data:
```

## Release decision framework

The aim is to avoid endless feature creep. This image should stop at a clear **Version 1.0** once it meets the minimum release criteria.

### Option A: Continue build

Use this only for features required before a customer can safely use the VM.

Recommended additions before listing:

1. Add a systemd service wrapper for the Compose stack.
2. Add a first-run/readme guide in `/opt/alsourillc/private-ai-chat/README.md`.
3. Add a simple health-check script.
4. Confirm the stack survives reboot.
5. Decide whether `llama3.2:3b` should remain preloaded or be removed before capture.

Do not add advanced features such as multi-node deployment, billing, user analytics, Entra ID, RAG, document upload pipelines, or custom dashboards to Version 1.0. Those should become separate offers or later versions.

### Option B: Marketplace-ready capture

Move to image capture when the following are true:

- Open WebUI returns HTTP `200` locally.
- Open WebUI is reachable through the intended Azure NSG rule.
- Ollama is bound only to `127.0.0.1:11434`.
- UFW is active.
- Docker containers restart after VM reboot.
- No personal Open WebUI admin account remains in the image, unless intentionally documented for a private/test image.
- No real secrets remain in `.env`.
- Build evidence is saved in this repository.
- Azure Linux Agent deprovisioning is performed before image capture.

### Option C: Monetisation/product layer

Do not include this in Version 1.0.

Examples:

- API key billing
- Usage metering
- Customer dashboards
- Stripe integration
- Entra ID integration
- Central licensing
- Support portal integration

These are valuable, but they slow down the first Marketplace listing. They should be planned after the first clean VM offer is live.

## Recommended Version 1.0 stop line

For ALSOURI LLC, the correct stop line for this first image is:

> Ubuntu server + Docker + Ollama + Open WebUI + secure local Ollama binding + firewall + reboot persistence + clear documentation.

Once that is done, stop adding features and prepare the Azure Compute Gallery image. Start the next project as a separate offer rather than turning this one into a large, unclear bundle.

## Next recommended steps

1. Add the systemd service wrapper.
2. Confirm reboot persistence.
3. Browser-test Open WebUI through Azure NSG.
4. Decide whether to include or remove `llama3.2:3b` from the final image.
5. Prepare Marketplace-safe cleanup and deprovisioning steps before Azure Compute Gallery capture.
