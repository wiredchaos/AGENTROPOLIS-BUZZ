# Hermes Session Relay Protocol

> Status: AGENTROPOLIS integration specification.

## Purpose

Define a shared relay contract for moving bounded, structured task context between independent Hermes sessions and external applications without copying full chat history, synchronizing application state, or collapsing distinct app boundaries.

The core rule is:

**Hermes sessions may exchange signed context capsules. Block Buzz and Slack remain separate applications with separate adapters, permissions, identities, storage, and receipts.**

This protocol is transport-neutral. Buzz is not Slack, Slack is not Buzz, and neither is the system of record for Hermes memory.

## Architecture

```text
Hermes Session A
      |
      | create context capsule
      v
Relay Protocol
      |
      +-------------------+
      |                   |
      v                   v
Block Buzz Adapter     Slack Adapter
      |                   |
      v                   v
Block Buzz App          Slack App
      |                   |
      +---------+---------+
                |
                | normalized inbound event
                v
          AGENTROPOLIS Ingest
                |
          policy + provenance
                |
                v
         Hermes Session B
```

A direct Hermes-to-Hermes route is also valid:

```text
Hermes Session A -> Relay Protocol -> Hermes Session B
```

## Application boundary rule

### Block Buzz

Block Buzz remains its own application and retains responsibility for:

- communities
- channels
- threads
- Nostr identities and signed events
- workflow state
- Buzz-native attachments
- ACP managed agents
- Buzz search and audit surfaces

The existing Hermes/Buzz bridge remains valid:

```text
Buzz managed agent
  -> buzz-acp
  -> hermes -p <profile> acp
  -> existing Hermes profile state
```

The Hermes profile remains the sole writable owner of its identity, memory, skills, sessions, `SOUL.md`, and `state.db`.

### Slack

Slack is a distinct application surface. A future Slack adapter may support:

- channels
- DMs
- threads
- mentions
- message events
- reactions
- files
- explicit workflow or approval events

Slack state must not be written directly into Hermes long-term memory or RAG simply because it was received. All inbound Slack content is untrusted sensor input and must pass through the AGENTROPOLIS ingest and policy path first.

## Relay commands

The user-facing command vocabulary should be consistent even when destination adapters differ.

```text
/relay session:<session-id>
/relay buzz:<community>/<channel>
/relay slack:<workspace>/<channel>
/relay-status <relay-id>
/relay-cancel <relay-id>
```

Adapter-specific syntax may evolve, but the relay envelope must remain stable.

## Context capsule

A relay MUST send a bounded context capsule rather than raw conversation history.

```json
{
  "relay_version": "1.0",
  "relay_id": "uuid",
  "source": {
    "type": "hermes_session",
    "session_id": "session-a",
    "agent_id": "hermes-builder"
  },
  "destination": {
    "type": "buzz|slack|hermes_session",
    "community_or_workspace": "optional",
    "channel_or_session": "target-id",
    "thread_id": "optional"
  },
  "objective": "Implement the runtime router",
  "decisions": [
    "Device Passport is canonical",
    "routing is capability-based"
  ],
  "changes": [
    "added Android Edge Worker"
  ],
  "blockers": [],
  "next_action": "Implement profiler and runtime router",
  "artifacts": [
    {
      "path": "PLANS/runtime-router.md",
      "sha256": "optional"
    }
  ],
  "constraints": [
    "no production deployment"
  ],
  "provenance": {
    "created_at": "RFC3339 timestamp",
    "source_event_id": "optional",
    "source_thread_id": "optional"
  },
  "context_policy": {
    "raw_history": false,
    "raw_files": false,
    "secrets": false
  }
}
```

## Mandatory relay pipeline

Every cross-application relay must pass through the same invariant:

```text
SOURCE
  -> SUMMARIZE
  -> SANITIZE
  -> PROVENANCE
  -> POLICY CHECK
  -> DESTINATION ADAPTER
  -> DELIVERY ACK
  -> RECEIPT
```

Inbound external messages use the reverse path:

```text
APP EVENT
  -> APP ADAPTER
  -> INGEST MEMBRANE
  -> NORMALIZE
  -> POLICY / RISK
  -> ROUTE
  -> HERMES SESSION
  -> RECEIPT
```

## Capability namespace

### Block Buzz adapter

```text
app.blockbuzz.observe
app.blockbuzz.analyze
app.blockbuzz.receive_relay
app.blockbuzz.dispatch
app.blockbuzz.publish
app.blockbuzz.attach
```

### Slack adapter

```text
app.slack.observe
app.slack.analyze
app.slack.receive_relay
app.slack.draft
app.slack.publish
app.slack.send_message
app.slack.attach
```

Observe/analyze permissions must be separable from execute/publish permissions.

## Receipt

Every relay must leave a permanent receipt.

```json
{
  "relay_id": "uuid",
  "source_session": "session-a",
  "source_agent": "hermes-builder",
  "destination_type": "buzz|slack|hermes_session",
  "destination_id": "target",
  "summary_hash": "sha256",
  "capabilities_used": ["app.blockbuzz.publish"],
  "policy_result": "allow|deny|approval_required",
  "delivery_status": "queued|delivered|failed|rejected",
  "remote_event_id": "optional",
  "acknowledged_at": "RFC3339 timestamp"
}
```

The receipt must never contain raw secrets.

## Security requirements

- Never copy raw Hermes chat history into another session by default.
- Never copy profile secrets, API keys, private keys, auth tags, or credential files into relay payloads.
- Never treat a signed Buzz event or authenticated Slack event as sufficient authorization for shell, repository, payment, deploy, or publish authority.
- Require capability-scoped policy enforcement outside model context.
- Preserve community/workspace isolation.
- Require explicit approval for destructive or externally consequential actions.
- Deduplicate relays by `relay_id` and source event identity.
- Do not let external app messages write directly into durable RAG or memory without validation and promotion.

## Hermes-to-Hermes semantics

Hermes sessions remain independent. The relay transfers only the capsule needed to continue work.

```text
Session A
  -> relay capsule
  -> Session B inbox/queue
  -> Session B acknowledges
  -> Session B continues task
```

This avoids context-window duplication and prevents unrelated source-session content from contaminating the target session.

## Block Buzz integration

The existing native remote gateway and shared-profile ACP routes continue to operate unchanged. Relay Protocol adds a new logical message type above those transports rather than replacing them.

Recommended first implementation:

```text
Hermes session
  -> relay capsule
  -> Hermes/Buzz adapter
  -> buzz-cli message send
  -> Buzz signed event
  -> destination channel/thread
  -> accepted event id
  -> relay receipt
```

For Hermes agents already hosted inside Buzz through ACP, relay capsules can be delivered as bounded thread messages or as structured workflow payloads while preserving the originating thread and relay ID.

## Slack integration

Slack support must be implemented as a separate adapter package or service. Do not place Slack-specific state, tokens, event parsing, or permission logic inside Buzz crates.

Recommended boundary:

```text
integrations/hermes-slack/
  README.md
  config.example.yaml
  schemas/
    slack-event.schema.json
  src/
    intake
    auth
    normalize
    relay
    publisher
  tests/
    workspace-isolation
    duplicate-event-idempotency
    permission-denial
    thread-routing
    secret-redaction
```

The Slack adapter should emit the same canonical relay envelope used by Buzz and direct session-to-session routing.

## AGENTROPOLIS placement

```text
HERMES                    Operator surface + execution runtime
Relay Protocol            Cross-session/app context contract
Block Buzz                Independent civic collaboration application
Slack                     Independent external communications application
Ingest Membrane           External-input normalization and quarantine
Policy / Risk Layer       Capability and approval authority
Audit Ledger              Relay and action receipts
Memory Layer              Governed durable continuity after validation
```

## Acceptance criteria

A first working implementation is complete when:

- Session A can create a bounded relay capsule for Session B without transmitting full history.
- A Hermes session can publish that same capsule into an authorized Buzz channel or thread and receive the accepted Buzz event ID.
- A separate Slack adapter can receive or publish the same canonical capsule without importing Slack code into Buzz.
- App permissions are independently scoped for Buzz and Slack.
- Cross-community or cross-workspace relay attempts fail closed unless explicitly approved.
- Duplicate relay IDs do not execute twice.
- Secrets are excluded from the envelope and receipts.
- Every relay produces a policy result and delivery receipt.
- External app content cannot silently promote itself into Hermes long-term memory or RAG.

## Canonical design rule

**Do not merge applications. Standardize the relay contract between them.**
