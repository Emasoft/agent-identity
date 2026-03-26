---
name: agent-identity
description: "Agent Identity (AID) authentication with Ed25519 keys and OAuth token exchange. Use when an agent needs API authentication or identity setup. Trigger with /agent-identity or aid-init."
license: MIT
compatibility: Requires curl, jq, openssl (3.x for Ed25519), base64. macOS and Linux.
metadata:
  version: "0.3.0"
  homepage: "https://agentids.org"
  repository: "https://github.com/agentmessaging/agent-identity"
---

# Agent Identity (AID) Protocol

Authenticate AI agents with auth servers using Ed25519 identity documents and proof of possession.

## Overview

AID provides cryptographic agent authentication via Ed25519 keypairs. Agents initialize an identity, register with auth servers, and exchange signed proofs for JWT tokens. Self-contained -- no other protocols required. Shares `~/.agent-messaging/agents/` with AMP if both installed.

## Prerequisites

- [ ] `curl` installed
- [ ] `jq` installed
- [ ] `openssl` 3.x with Ed25519 support (macOS: `brew install openssl@3`)
- [ ] `base64` CLI available
- [ ] Admin JWT token for registration step

## Instructions

1. Initialize agent identity (one-time setup):
   ```bash
   aid-init.sh --auto
   ```
   Or specify a name: `aid-init.sh --name my-agent`

2. Register with an auth server (one-time, requires admin JWT):
   ```bash
   aid-register.sh --auth https://auth.example.com/tenant \
     --token "$ADMIN_JWT" --role-id 2
   ```

3. Get a JWT token for API calls:
   ```bash
   TOKEN=$(aid-token.sh --auth https://auth.example.com/tenant --quiet)
   ```

4. Use the token with any API:
   ```bash
   curl -H "Authorization: Bearer $TOKEN" https://api.example.com/resource
   ```

5. Check identity and registration status:
   ```bash
   aid-status.sh
   ```

## Output

- `aid-init.sh` -- Creates `~/.agent-messaging/agents/NAME/` with keypair and config.json
- `aid-register.sh` -- Saves registration to `api_registrations/HOST.json`, prints unique ID
- `aid-token.sh` -- Returns JWT access token (text, JSON, or quiet mode). Caches in `tokens/`
- `aid-status.sh` -- Displays identity, registrations, and cached tokens

## Error Handling

| Problem | Solution |
|---------|----------|
| Agent identity not initialized | Run `aid-init.sh --auto` |
| Not registered | Run `aid-register.sh` with auth server details |
| Proof expired | Sync system clock (skew must be under 5 min) |
| Invalid signature | Re-init and re-register: `aid-init.sh --force` |
| Agent suspended | Contact admin for reactivation |
| Scope not allowed | Request only scopes granted during registration |

## Examples

Initialize and authenticate:

```bash
# Initialize
aid-init.sh --auto

# Register
aid-register.sh --auth https://auth.23blocks.com/acme \
  --token eyJhbGciOi... --role-id 2

# Get token for API calls
TOKEN=$(aid-token.sh --auth https://auth.23blocks.com/acme --quiet)

# Use token
curl -H "Authorization: Bearer $TOKEN" https://api.example.com/files
```

Scoped tokens:

```bash
aid-token.sh --auth https://auth.23blocks.com/acme --scope "files:read"
```

## Resources

- Specification: https://agentids.org
- Repository: https://github.com/agentmessaging/agent-identity
- AMP interop: Shares `~/.agent-messaging/agents/` directory
