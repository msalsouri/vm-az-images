# End-to-End Chat Validation

Publisher/company context: **ALSOURI LLC**

VM image candidate: **Private AI Chat Server**

## Validation result

Open WebUI was accessed successfully through the VM public IP and Azure NSG rule on TCP port `3000`.

The test LLM `llama3.2:3b` responded successfully from the Open WebUI chat interface.

## Network validation

External connectivity test from local machine:

```text
nc -vz 20.83.152.106 3000
Connection to 20.83.152.106 3000 port [tcp/*] succeeded!
```

This confirms the full network path:

```text
Client browser / local machine
→ Azure NSG port 3000
→ Ubuntu UFW port 3000
→ Docker published port 3000
→ Open WebUI container
→ Ollama backend on localhost-only port 11434
```

## Chat validation

Model used:

```text
llama3.2:3b
```

User prompt:

```text
hi, can you help me please?
```

Model response:

```text
I'd be happy to try and assist you. What's on your mind? Do you have a specific question or problem you'd like help with? I'm all ears!
```

## Status

This proves that the VM is not only running services, but is functionally usable as a private AI chat server.

## Notes before image capture

Before capturing a public Marketplace image, decide whether to:

1. Keep `llama3.2:3b` preloaded for immediate demo functionality.
2. Remove the Open WebUI admin/user account and let the customer create the first account on first launch.
3. Reset or rotate any static secret values in `.env`.
4. Keep the Azure NSG documentation clear: expose TCP `3000`, do not expose TCP `11434`.
