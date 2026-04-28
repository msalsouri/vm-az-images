# Trusted Launch Warning Decision

Offer: Private AI Chat Server on Ubuntu 24.04 LTS
Plan: Private AI Chat Server - Ubuntu 24.04 LTS
Publisher/company context: ALSOURI LLC

## Partner Center warning

Partner Center warning:

```text
Are you sure you want to enable this security type option?

The security type will apply to all future Gen 2 images published in this plan. Once enabled, this selection cannot later be downgraded.
```

## Recommended decision

Proceed with:

```text
Trusted launch (Recommended)
```

## Reason

This is acceptable for this plan because the image is Gen 2 and this plan should remain the modern secure baseline for the product. The non-downgrade warning means future Gen 2 image versions under this same plan must continue to support Trusted Launch.

## Important implication

If a future version needs lower compatibility or legacy/non-Trusted-Launch behavior, create a separate plan instead of trying to downgrade this plan.

Example future fallback plan:

```text
Plan ID: private-ai-chat-ubuntu-2404-standard
Security type: None
```

## Do not choose for this first version

Do not choose `Trusted launch and confidential` unless a separate confidential VM image has been built and tested.
