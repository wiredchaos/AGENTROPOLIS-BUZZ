# Buzz Skills and ACP Preset Integration

## Status

This document records two upstream developments that extend the Hermes + Buzz integration:

1. [`tonbistudio/buzz-skills`](https://github.com/tonbistudio/buzz-skills) provides portable Hermes skills for a native remote Hermes gateway and verified native Buzz media attachments.
2. [`block/buzz#3225`](https://github.com/block/buzz/pull/3225) is merged and adds Devin to Buzz's built-in ACP preset catalog through the official `devin acp` invocation.

These are implementation inputs, not permission grants. AGENTROPOLIS policy, approvals, isolation, and receipts remain mandatory.

## Supported Hermes connection patterns

### Native remote gateway

```text
Buzz Desktop or web client
          |
          v
      Buzz relay
          ^
          | Nostr WebSocket + Buzz CLI
          |
Remote Hermes gateway
```

Use this path when Hermes should retain its full profile, memory, skills, scheduled tasks, messaging behavior, and long-running gateway state on a remote host.

The client and Hermes host connect independently to the relay. The client does not need an SSH connection to the Hermes machine.

### Shared-profile ACP runtime

```text
Buzz managed agent
  -> buzz-acp
  -> hermes -p <profile> acp
  -> existing Hermes profile state
```

Use this path when Buzz should own the interactive agent session while Hermes preserves the selected profile's memory, personality, and skills.

### Local Buzz Desktop runtime

Use this path when Hermes and Buzz Desktop intentionally run on the same operator machine.

## Portable skills

### `hermes-in-buzz`

The upstream skill provides an end-to-end native-gateway workflow covering:

- relay connectivity
- dedicated Nostr agent identity
- owner-only invocation defaults
- Buzz CLI installation
- NIP-OA owner attestation
- gateway configuration
- live inbound and outbound verification
- Linux, macOS, Windows, and WSL operating paths

The skill requires real end-to-end verification. A running process is not enough. Success requires a received Buzz message, a Hermes response, relay acceptance, and visible delivery in Buzz.

### `buzz-media-attachments`

The upstream media skill uses Buzz's native attachment command:

```bash
buzz messages send \
  --channel <CHANNEL_UUID> \
  --content - \
  --file /absolute/path/to/file
```

It verifies delivery only when Buzz returns an accepted event and event ID. For rejected MP4 files, it supports fast-start remuxing, canonical metadata-free H.264/AAC re-encoding, and GIF fallback.

AGENTROPOLIS receipts should retain:

- source artifact path and hash
- transformed artifact path and hash, when applicable
- target community and channel
- accepted event ID
- media transformation performed
- requesting and approving identities

## Mandatory security rules

- Use a dedicated Buzz agent identity. Never reuse a human owner's private key.
- Never place private keys, auth tags, or API tokens in chat or command-line arguments.
- Enter secrets locally through hidden input and store them only in the active Hermes profile's secret environment.
- Default to `allow_all_users: false`, an explicit owner allowlist, and `require_mention: true`.
- Parse `.env` files with a dotenv parser. Do not shell-source `BUZZ_AUTH_TAG` JSON.
- Use an absolute Buzz CLI path so service environments do not silently lose access.
- Treat ACP-hosted Hermes as a stronger host-security boundary because ACP clients may auto-answer tool permissions.
- Do not broaden agent access merely to diagnose delivery.

## Devin ACP preset

Buzz now includes a built-in Devin preset with this contract:

```text
preset id: devin
executable: devin
arguments: acp
source: built-in Buzz ACP runtime catalog
```

The merged upstream change adds discovery metadata, setup guidance, tests, and a bundled runtime icon. It does not add Devin-specific authentication probing, permission bypasses, model switching, cloud handoff, or expanded capability claims.

Within AGENTROPOLIS, Devin is classified as a BYOH ACP execution lane:

```text
Buzz thread
  -> Mission Control mandate
  -> ACP runtime selection
  -> Devin preset when approved and available
  -> bounded repository or workspace scope
  -> artifact and receipt
  -> human review
```

Devin availability does not authorize repository writes, merges, deployments, external publication, payments, or access to unrelated workspaces.

## Runtime selection rule

```text
Need persistent remote Hermes memory and gateway behavior?
  -> native Hermes gateway

Need Buzz-owned interaction with an existing Hermes profile?
  -> shared-profile ACP runtime

Need another approved coding harness?
  -> Buzz preset catalog, including Devin
```

Every route must preserve the same governance invariant:

```text
IDENTITY -> MANDATE -> CAPABILITY POLICY -> EXECUTION -> ARTIFACT -> RECEIPT -> REVIEW
```

## Adoption checklist

- Pin and review the upstream skill revision before installation.
- Install skills into the active Hermes profile, not an assumed global home.
- Verify a dedicated agent identity and owner-only defaults.
- Verify relay admission and channel discovery.
- Test one inbound and one outbound text exchange.
- Test one native attachment and record the accepted event ID.
- Confirm the Buzz version includes the merged Devin preset before advertising it.
- Keep deploy, publish, merge, and destructive capabilities behind explicit signed approvals.
