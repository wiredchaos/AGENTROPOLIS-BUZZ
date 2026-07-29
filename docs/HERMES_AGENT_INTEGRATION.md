# Hermes Agent + Buzz Integration Blueprint

> Status: design proposal. This document describes a target integration for AGENTROPOLIS. It does not claim that a production Hermes adapter already ships in this repository.

## Objective

Combine Buzz's signed, community-scoped collaboration layer with Hermes Agent's execution runtime so humans and agents can coordinate in persistent channels, perform bounded work, publish durable artifacts, and leave an auditable trail.

The operating principle is simple:

**Buzz coordinates. Hermes executes. The workspace preserves artifacts. Humans govern authority.**

## AGENTROPOLIS placement

```text
Human Mission Control
        |
        | mandate, policy, approvals
        v
Buzz community and channel layer
        |
        | signed messages, threads, mentions, workflow events
        v
Hermes adapter / dispatcher
        |
        | normalized job envelope
        v
Hermes Agent runtime
        |
        | browser, terminal, files, approved tools
        v
Shared workspace
RESEARCH/  PLANS/  GUIDES/  OUTBOX/  RECEIPTS/
        |
        | artifact paths, hashes, summaries, status
        v
Buzz thread + audit trail + human review
```

## Component responsibilities

### Buzz

Buzz is the civic communications grid for the integration.

It should provide:

- community-local identity, channels, threads, mentions, reactions, and search
- signed events for human and agent activity
- stable task threads that survive agent restarts
- workflow triggers and approval signals
- links between conversations, patches, CI results, artifacts, and decisions
- an auditable record of who requested, approved, executed, and reviewed work

### Hermes Agent

Hermes is the operator and execution runtime.

It should provide:

- planning and task decomposition
- browser and terminal execution
- tool and MCP invocation
- local-first file creation and transformation
- scheduled or long-running workflows where explicitly authorized
- concise status updates and final artifact publication

### Shared workspace

The workspace is durable operational memory, not a replacement for Buzz's signed event log.

Recommended layout:

```text
RESEARCH/   source notes and evidence
PLANS/      execution plans and checklists
GUIDES/     reusable operating procedures
OUTBOX/     completed deliverables ready for review or release
RECEIPTS/   manifests, hashes, logs, and execution summaries
```

Agents should hand work off by verified path and manifest, not by pasting large deliverables into chat.

### Human Mission Control

Humans remain the authority boundary.

Mission Control should:

- issue mandates and define scope
- approve sensitive capabilities
- review proposed destructive or external actions
- accept, reject, or revise artifacts
- revoke agent access and credentials
- preserve final accountability

## Proposed interaction model

### 1. Dispatch

A human or authorized workflow mentions a Hermes-backed agent in a Buzz thread.

Example:

```text
@hermes-researcher investigate the reported relay failure.
Write findings to RESEARCH/relay-failure.md.
Do not modify production systems.
Return the path, evidence links, and open questions.
```

### 2. Job normalization

The adapter converts the signed Buzz event into a bounded job envelope.

```json
{
  "community": "derived-from-relay-url",
  "channel_id": "channel-event-id",
  "thread_root": "root-event-id",
  "request_event": "signed-request-event-id",
  "agent": "hermes-researcher",
  "mandate": "Investigate the reported relay failure",
  "allowed_capabilities": ["read_files", "web_research"],
  "denied_capabilities": ["write_production", "publish_external"],
  "output_path": "RESEARCH/relay-failure.md",
  "approval_required": false
}
```

The envelope should be treated as a runtime constraint, not merely prompt text.

### 3. Execution

Hermes performs the work using only approved tools and paths. Progress updates should be sparse and operationally useful.

Recommended states:

```text
QUEUED -> CLAIMED -> PLANNING -> EXECUTING -> BLOCKED | REVIEW_READY -> ACCEPTED | REJECTED
```

### 4. Artifact publication

Hermes writes the deliverable and a receipt manifest.

```json
{
  "status": "review_ready",
  "artifact": "RESEARCH/relay-failure.md",
  "receipt": "RECEIPTS/relay-failure.json",
  "sha256": "<artifact-hash>",
  "request_event": "<signed-request-event-id>",
  "tools_used": ["browser", "filesystem"],
  "external_side_effects": [],
  "open_questions": []
}
```

The agent posts the verified paths, hash, and a concise summary back to the originating Buzz thread.

### 5. Review and handoff

A human or specialist agent reviews the artifact. Approval may be represented by an explicit workflow action or a configured signed reaction.

A downstream agent should receive:

- the original thread root
- the current mandate
- verified artifact paths
- the receipt manifest
- unresolved questions
- the next permitted action

## Agent roles

A practical first deployment can use four narrowly scoped Hermes identities:

| Agent | Purpose | Default authority |
|---|---|---|
| `hermes-researcher` | Research, evidence gathering, summaries | Read-only external access, workspace write |
| `hermes-builder` | Code and artifact production | Repository branch write, no direct merge |
| `hermes-reviewer` | Review plans, diffs, receipts, and risks | Read-only with comment authority |
| `hermes-operator` | Approved deployments and operational actions | Explicit approval required |

Each agent should have its own key, channel membership, tool policy, credentials, and audit trail.

## Security and governance boundaries

### Identity is not authority

A signed agent identity proves which key produced an event. It does not automatically authorize shell access, repository writes, deployment, payments, or external publication.

### Runtime enforcement

Capability restrictions must be enforced outside the language model through:

- per-agent tool allowlists
- scoped filesystem roots
- isolated worktrees or containers
- short-lived credentials
- domain and network restrictions
- timeouts and output limits
- approval gates for destructive or external actions

### Community isolation

Buzz communities are selected by relay URL and remain semantically isolated. The adapter must not carry jobs, memory, memberships, credentials, or artifacts across communities unless an explicit export is approved.

### Prompt injection resistance

Content retrieved from websites, files, messages, or repositories is untrusted data. It must not silently expand the mandate or capability set.

### Receipt-first operations

Every meaningful execution should produce a receipt containing:

- requester and request event
- agent identity
- capability policy used
- tools invoked
- files created or changed
- external side effects
- hashes and timestamps
- approval events
- final disposition

## Integration surfaces

The first adapter should prefer existing standards and interfaces:

1. **Buzz event intake** through `buzz-cli`, relay APIs, or a narrowly scoped service account.
2. **Hermes invocation** through its supported local command, API, or MCP-compatible surface.
3. **Artifact storage** in an operator-controlled workspace or repository worktree.
4. **Status return** as signed Buzz events linked to the original thread.
5. **Approval handling** through explicit workflow events rather than ambiguous natural-language consent.

Avoid coupling the core Buzz relay directly to Hermes-specific runtime code. Keep the integration in a separate adapter process so either system can evolve independently.

## Suggested adapter package

```text
integrations/hermes-buzz/
  README.md
  config.example.yaml
  schemas/
    job-envelope.schema.json
    receipt.schema.json
  src/
    intake
    policy
    dispatcher
    publisher
  tests/
    community-isolation
    capability-denial
    duplicate-event-idempotency
    approval-gates
    artifact-verification
```

## Delivery phases

### Phase 0: documentation and threat model

- finalize event-to-job schema
- define agent identities and channel memberships
- define capability classes and approval rules
- document secrets, revocation, and incident response

### Phase 1: read-only researcher

- accept mentions from one test channel
- dispatch one Hermes researcher
- allow web and workspace writes only
- publish artifacts and receipts back to the thread
- reject duplicate events idempotently

### Phase 2: repository builder

- create isolated branches or worktrees
- generate patches and tests
- post commit and PR references into Buzz
- require humans to merge

### Phase 3: guarded operations

- add scheduled jobs and deployment workflows
- require signed approval events
- use short-lived credentials
- record all external side effects

### Phase 4: multi-agent routing

- route work among researcher, builder, reviewer, and operator
- preserve one thread root across handoffs
- add stalled-job callbacks and escalation
- expose status to AGENTROPOLIS Mission Control

## Acceptance criteria for the first working slice

A Phase 1 implementation is complete only when all of the following are true:

- a signed mention in an authorized Buzz channel creates exactly one Hermes job
- the job cannot access tools outside its configured capability policy
- the job writes an artifact and receipt to approved workspace paths
- the completion event links to the original thread and contains verified hashes
- restarting the adapter does not duplicate completed work
- the same event from another community is not accepted automatically
- revoking the agent key or adapter credential stops new work
- failures are visible, bounded, and leave a receipt

## AGENTROPOLIS designation

```text
Buzz                 Civic Communications Grid
Hermes Agent         Operator and Execution Runtime
Shared Workspace     Durable Operational Memory
Forgejo or Git       Versioned Artifact Authority
Mission Control      Human Governance Plane
AEGIS                 Policy and Risk Enforcement
Audit Ledger         Receipt and Accountability Layer
```

This integration turns chat into a governed production loop:

```text
IDENTITY -> MANDATE -> PLAN -> EXECUTE -> ARTIFACT -> RECEIPT -> REVIEW -> AUDIT
```

The goal is not autonomous conversation. The goal is accountable work with bounded authority, durable outputs, and evidence humans can inspect.
