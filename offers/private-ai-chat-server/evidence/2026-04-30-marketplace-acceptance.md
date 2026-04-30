# Marketplace Acceptance Evidence

Offer: Private AI Chat Server on Ubuntu 24.04 LTS
Publisher: ALSOURI LLC
Date: 2026-04-30
Status: Accepted / Live

## Summary

The Azure Marketplace VM offer **Private AI Chat Server on Ubuntu 24.04 LTS** has been accepted and is now live.

## Final accepted build approach

The accepted image used the thin-image pattern:

- Fresh Ubuntu 24.04 LTS Gen2 VM
- Docker and Docker Compose installed
- Open WebUI and Ollama Docker Compose configuration present
- Application systemd service present but disabled before capture
- No Docker containers running before capture
- No Docker images pulled before capture
- No preloaded LLM model
- No Microsoft Defender for Endpoint / `mdatp`
- No MDE waagent extension artifacts
- No EICAR traces
- Shell history cleared immediately before `waagent -deprovision+user`

## Key certification lesson

The previous failures were caused by Microsoft Defender for Endpoint / `mdatp` being injected by Defender for Servers/MDE extension behavior. The successful approach required confirming no MDE extension or `mdatp` artifacts existed before capture.

## Final customer first-use instruction

Customers must allow inbound TCP port 3000 in the Azure Network Security Group and start the service after deployment:

```bash
sudo systemctl enable --now alsourillc-private-ai-chat.service
```

The service then pulls and runs Open WebUI and Ollama containers on first use.
