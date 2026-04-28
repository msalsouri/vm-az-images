# Security Type Recommendation

Offer: Private AI Chat Server on Ubuntu 24.04 LTS
Plan: Private AI Chat Server - Ubuntu 24.04 LTS
Publisher/company context: ALSOURI LLC

## Recommended selection

```text
Trusted launch (Recommended)
```

## Do not select for this first version

```text
Trusted launch and confidential
```

unless the image has been specifically built and tested for Confidential VM scenarios.

## Rationale

Trusted launch is the best default for Gen 2 Marketplace VM images. It supports a stronger security posture while keeping deployment compatibility broader than Confidential VM-only images.

Confidential VM support should be introduced later as a separate plan or image variant after validation on supported VM sizes and regions.

## Recommended future plan structure

Current plan:

```text
private-ai-chat-ubuntu-2404
Security type: Trusted launch
```

Future optional plan:

```text
private-ai-chat-ubuntu-2404-confidential
Security type: Trusted launch and confidential
```
