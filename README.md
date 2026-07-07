# Docker Sandboxes

## Questions

- [ ] What are microVM's to Docker Sandboxes?

## Default agents

- claude
- copilot
- docker-agent
- gemini
- opencode
- codex
- cursor
- droid
- kiro
- shell

## Getting Started

- Installing
- Logging In
  - Set Network Policy

## Secrets

Mostly API keys for AI providers. They are proxy managed which means that tools inside the vm cannot read the secret directly. But when using it in a request, they will be replaced by sbx automatically.

Secrets set are immediately available in all running sandboxes.

### API Keys

- global
- per sandbox

## Run

After running with a secret set, we shouldn't need to /connect to an api source, it should just work.

## Network Policy


