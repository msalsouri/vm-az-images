# Azure Image Capture Template Usage

Publisher/company context: **ALSOURI LLC**

Offer/image candidate: **Private AI Chat Server**

## Purpose

When Azure shows **Download a template for automation** during image creation, the exported ARM/Bicep template should be treated as the repeatable recipe for the image-capture operation.

It is useful for repeatability, evidence, review, automation, and future image versions.

## Current image-capture settings

Observed image creation configuration:

```text
Subscription: MPN VM's
Resource group: rg-alsourillc-images-custom-linux-base-dev
Region: East US
Share image to Azure compute gallery: Yes
Automatically delete this virtual machine after creating the image: Yes
Azure compute gallery: acg_alsourillc_images_custom_linux_base_dev
Operating system state: Generalized
Target VM image definition: ubuntu2404-openwebui-ollama.Def
Version number: 1.0.0
Source virtual machine: ubuntu2204-min-vm
Exclude from latest: No
End of life date: None
Lock deleting Replicated Locations: Yes
Block deletion before End of life: No
Shallow replication: No
Default replica count: 1
Replica region: East US: 1
Tags: ubuntu, 24.04, ollama, webui
```

## What the downloaded template can be used for

### 1. Repeat the same image capture consistently

The template captures the exact Azure resource settings used to create the image version. This avoids relying on portal memory or screenshots.

### 2. Create future image versions

For future updates, keep the same image definition and change only the version number, for example:

```text
1.0.0
1.0.1
1.1.0
2.0.0
```

Recommended versioning:

- Patch version: security updates, package updates, small fixes
- Minor version: new model, improved documentation, service changes
- Major version: different architecture, different app stack, breaking change

### 3. Promote images across environments

A clean workflow can be:

```text
dev gallery -> test gallery -> production gallery -> Partner Center image version
```

The template can be parameterized so that only the gallery name, resource group, region, or version changes.

### 4. Use Infrastructure as Code

The template can be committed into GitHub and used with Azure CLI, Azure DevOps, or GitHub Actions.

Example deployment pattern:

```bash
az deployment group create \
  --resource-group rg-alsourillc-images-custom-linux-base-dev \
  --template-file azure-image-capture-template.json \
  --parameters @azure-image-capture-parameters.json
```

### 5. Create an internal evidence trail

For Marketplace and operational discipline, store:

- Original downloaded template
- Parameters file
- Image version notes
- Build evidence
- Validation logs
- Deprovisioning checklist

### 6. Compare image versions

Template diffs make it easier to answer:

- What changed between 1.0.0 and 1.0.1?
- Did replication change?
- Did source image definition change?
- Was `excludeFromLatest` changed?
- Were tags changed?

### 7. Avoid portal mistakes

The portal is good for first creation, but templates prevent repeated manual mistakes, especially across multiple offers.

## Trade-secret style practices

### Keep image definitions stable

Do not create a new image definition for every small update. Use one image definition and publish new versions under it.

Example:

```text
Image definition: ubuntu2404-openwebui-ollama.Def
Versions: 1.0.0, 1.0.1, 1.1.0
```

### Separate base image and product image

Maintain separate image families:

```text
ubuntu2404-docker-base.Def
ubuntu2404-openwebui-ollama.Def
ubuntu2404-n8n-ai.Def
ubuntu2404-rag-server.Def
```

This makes future marketplace offers faster to build.

### Use tags aggressively

Recommended tags:

```text
publisher=alsourillc
product=private-ai-chat-server
os=ubuntu-24.04
runtime=docker
ai=ollama
ui=open-webui
version=1.0.0
stage=dev
marketplaceCandidate=true
```

### Keep dev/test/prod separation

Recommended pattern:

```text
rg-alsourillc-images-dev
rg-alsourillc-images-test
rg-alsourillc-images-prod
```

or at least:

```text
stage=dev/test/prod
```

### Store the template before clicking Create

The downloaded template represents the intended operation before Azure creates/deletes anything. Keep it as proof of intended configuration.

### Disable auto-delete only if you still need the source VM

If `Automatically delete this virtual machine after creating the image` is enabled, make sure a snapshot exists first.

For this build, the source VM should only be deleted after:

- snapshot checkpoint exists
- image capture succeeds
- Azure Compute Gallery version appears healthy

## Recommended repository structure

```text
offers/private-ai-chat-server/
├── README.md
├── docs/
│   └── azure-image-template-usage.md
├── evidence/
│   ├── 06-end-to-end-chat-validation.md
│   └── 07-pre-capture-snapshot-checkpoint.md
└── infra/
    ├── azure-image-capture-template.json
    └── azure-image-capture-parameters.json
```

## Immediate recommendation for this image

Before clicking **Create**, download the template and store it under:

```text
offers/private-ai-chat-server/infra/azure-image-capture-template.json
```

If Azure also provides a parameters file, store it as:

```text
offers/private-ai-chat-server/infra/azure-image-capture-parameters.json
```

Then proceed with image creation only if the snapshot has already been created.
