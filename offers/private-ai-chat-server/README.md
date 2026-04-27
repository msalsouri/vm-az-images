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

Result:

```json
{"models":[]}
```

This confirms Ollama is running. No models had been pulled at this checkpoint.

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

## Next recommended steps

1. Restrict Azure NSG inbound rule for port `3000` to a trusted source IP during testing.
2. Pull a small test model, for example `llama3.2:3b`.
3. Test chat through Open WebUI.
4. Decide whether to keep or remove preloaded models before image capture.
5. Prepare Marketplace-safe cleanup and deprovisioning steps before Azure Compute Gallery capture.
