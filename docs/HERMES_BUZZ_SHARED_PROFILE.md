# Hermes Buzz Shared Profile Reference

## Working bridge

[`r0b0tlab/hermes-buzz-shared-profile`](https://github.com/r0b0tlab/hermes-buzz-shared-profile) provides a working cross-platform bridge for adding an existing Hermes profile to Buzz Desktop as a native managed agent.

```text
Hermes profile
  -> hermes -p <profile> acp
  -> buzz-acp child process
  -> Buzz managed agent
  -> Buzz channel messages
```

The skill writes an entry into Buzz Desktop's `managed-agents.json` and points it at the local Hermes executable. Buzz then launches Hermes through ACP when the agent is started.

## Install

```bash
hermes skills inspect amanning3390/hermes-buzz-shared-profile/hermes-buzz-shared-profile
hermes skills install amanning3390/hermes-buzz-shared-profile/hermes-buzz-shared-profile

hermes profile create hermes-buzz --description "Shared profile for Buzz"
python3 ${HERMES_SKILL_DIR}/scripts/shared_profile.py buzz-add --profile hermes-buzz
```

Restart Buzz Desktop after registration.

## Managed-agent shape

```json
{
  "agent_command": "/resolved/path/to/hermes",
  "agent_args": ["-p", "<profile>", "acp"],
  "acp_command": "buzz-acp",
  "slug": "hermes:<profile>",
  "is_builtin": false,
  "is_active": true,
  "respond_to": "owner-only"
}
```

## Shared-state contract

The Hermes profile directory remains the sole writable owner of identity, configuration, memory, skills, sessions, `SOUL.md`, and `state.db`. The Buzz entry is a reference and launch configuration. The bridge does not clone or synchronize profile state.

## What the bridge handles

- macOS, Linux, and Windows Buzz data-directory discovery
- Hermes executable resolution
- profile validation
- atomic `managed-agents.json` updates
- add, update, remove, and list lifecycle
- optional bounded prompt import from `SOUL.md`

## What it does not handle

This bridge does not implement the full governed AGENTROPOLIS execution loop. It does not itself provide:

- capability allowlists
- scoped filesystem roots
- signed job-envelope normalization
- external-action approval gates
- artifact manifests and receipts
- secrets brokering
- deployment, publishing, payment, or wallet authority
- cross-community memory transfer

Those controls remain the responsibility of Mission Control, AEGIS, the Hermes execution adapter, and the receipt layer.

## Security note

`respond_to: owner-only` is useful, but it is a host-security boundary rather than a complete authorization system. Because ACP can reach local Hermes tools, compromise of the authorized owner identity may become a path to local command execution. Sensitive tools require separate runtime enforcement, approval binding, credential scoping, and revocation.

## Reviewed upstream status

- Skill version: `0.3.0`
- Runtime: Python 3.11+, standard library
- Platforms: macOS, Linux, Windows 10/11
- License: MIT
