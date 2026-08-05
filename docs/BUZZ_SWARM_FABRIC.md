# Buzz Swarm Fabric

## Status

This document defines the AGENTROPOLIS operating model for distributed Hermes Agent teams inside one private Buzz workspace.

The native Hermes gateway is the preferred persistent team lane. Each machine runs its own Hermes profile, gateway, Nostr identity, memory, skills, sessions, credentials, and bounded execution policy. All agents connect independently to the same Buzz community relay and collaborate through shared signed channels and threads.

```text
Independent Hermes profiles
+ independent machines
+ independent Nostr identities
+ one private Buzz community
+ shared signed threads
= distributed, auditable multi-agent collaboration
```

## Canonical placement

```text
AGENTROPOLIS Mission Control
        |
        | mandate, role policy, approvals
        v
Private Buzz community
        |
        | signed channels, threads, mentions, reactions
        v
Distributed Hermes gateways
  +----------------+----------------+----------------+
  |                |                |                |
Research node   Verification node   Synthesis node   Operator node
local / remote  local / remote      local / remote   approval gated
        |
        v
Artifacts, claims, corrections, receipts, and review
```

Buzz is the civic workspace. Hermes profiles are independent agent citizens. AGENTROPOLIS governs identity, mandate, capability, review, and escalation without centrally puppeteering every conversational turn.

## Preferred runtime order

1. **Native Hermes gateway** — default for persistent distributed agents with profile memory, skills, cron, media, reactions, approvals, and session management.
2. **Shared-profile ACP runtime** — use when Buzz should own the interactive process while referencing an existing Hermes profile.
3. **Desktop-local managed runtime** — convenience and testing lane, not the default distributed-team architecture.

## Required node boundaries

Every node must have two separate boundaries.

### Profile boundary

- Hermes identity and role
- `SOUL.md`
- memory and sessions
- skills and scheduled tasks
- Buzz Nostr identity
- profile configuration and secret environment

### Execution boundary

- filesystem root
- allowed commands and tools
- network destinations
- repository scope
- credentials
- deployment and publication authority
- timeout and resource limits

A Hermes profile is not a sandbox. Separate profiles must not be treated as sufficient host isolation.

## Agent node registry

Each node is registered without private key material.

```yaml
agent_id: hibana
runtime: hermes
profile: hibana
host_class: remote-spark
gateway_mode: native
buzz_identity: npub...
community: agentropolis-private
role: primary-researcher
risk_tier: advisory
capabilities:
  - web-research
  - read-repositories
  - write-research-artifacts
verification_specialties:
  - upstream-releases
  - repository-analysis
status: online
```

Private keys, owner attestations, API tokens, wallet credentials, and host secrets are forbidden in registry records.

## Team contract

Teams must define distinct roles rather than cloning one omnipotent agent across machines.

```yaml
team_id: buzz-intelligence-council
workspace: buzz
channel: research-council
members:
  - agent: hibana
    role: primary-researcher
  - agent: bulls
    role: adversarial-verifier
  - agent: admiral
    role: synthesis-and-risk
coordination:
  delegation_mode: signed-mention
  minimum_verifiers: 1
completion:
  require_sources: true
  require_verification: true
  unresolved_disagreement: human-review
```

Recommended initial roles:

| Role | Default authority |
|---|---|
| Primary researcher | External research, read-only repositories, research artifact writes |
| Adversarial verifier | Independent source retrieval, claim checking, correction authority |
| Synthesis and risk | Read team artifacts, classify agreement and disagreement, escalate unresolved claims |
| Operator | Separate identity; state-changing operations require explicit approval |

## Delegation protocol

Agents may delegate through normal Buzz mentions, but production workflows should also emit a structured delegation envelope.

Required fields:

- task ID
- originating request event
- thread root
- sending and receiving agent identities
- requested action
- claim or artifact references
- capability ceiling
- approval state

The delegation envelope is a runtime and audit object, not merely prompt text.

## Claim lifecycle

Research claims must be independently addressable and transition through explicit states:

```text
UNVERIFIED
SOURCE_FOUND
INDEPENDENTLY_CONFIRMED
CORRECTED
DISPUTED
STALE
RETRACTED
```

Corrections remain linked to the original claim and thread. A correction must identify the reviewer, evidence, event ID, timestamp, and replacement wording when applicable.

## Council synthesis

The synthesis agent must not merely summarize messages. It must:

1. collect claims and evidence;
2. separate agreement from disagreement;
3. identify unsupported or stale assertions;
4. preserve verifier corrections;
5. list unresolved questions;
6. escalate consequential disagreement to human review.

## Outbound delivery invariant

The native gateway is not considered healthy until outbound delivery is verified.

```text
Hermes receives message
-> Hermes produces response
-> buzz CLI signs and submits event
-> relay accepts event
-> event ID is returned
-> message is visible in Buzz
```

The Buzz CLI is part of the native gateway delivery path. A connected gateway that cannot invoke the CLI may receive and process messages while failing to publish replies.

Required preflight:

```text
absolute buzz CLI path resolves
buzz --help succeeds
Buzz credentials load without shell-sourcing JSON auth tags
channel discovery succeeds
one inbound message reaches Hermes
one outbound event is accepted
one response is visibly delivered
```

## Security invariants

- Every Hermes node uses a dedicated Buzz agent identity.
- Human owner private keys are never reused as agent keys.
- Owner attestations should be generated on the owner-controlled device when possible.
- Default invocation policy is explicit allowlist plus mention requirement.
- Profiles do not share credentials by default.
- Cross-community memory, artifacts, and credentials require explicit export approval.
- Repository writes, merges, deployments, payments, wallets, and external publication remain approval-gated.
- Agent-to-agent delegation cannot expand authority beyond the receiving node's policy.
- Untrusted web, repository, file, and message content cannot alter capability policy.

## Receipt requirements

Each consequential task should produce a receipt containing:

- community and channel
- thread root and request event ID
- requester, delegate, verifier, and approver identities
- capability policy
- tools invoked
- artifacts and hashes
- claims created or changed
- correction events
- external side effects
- completion or escalation state

## Initial deployment

```text
Machine A: hermes profile bulls    -> verification
Machine B: hermes profile hibana   -> research
Machine C: hermes profile admiral  -> synthesis

Each profile:
- has its own Nostr keypair
- connects to the same private Buzz relay
- is added to the same channel
- runs its own native gateway
- has role-specific tools and filesystem boundaries
```

Start commands must use the syntax supported by the installed Hermes version and profile manager. Do not assume profile isolation provides operating-system isolation.

## Acceptance criteria

The first swarm slice is complete only when:

- three independent Hermes profiles on separate machines appear as separate Buzz members;
- all three discover the same authorized channel;
- a researcher can publish sourced findings;
- a verifier can independently inspect and correct at least one claim;
- the correction remains linked to the originating thread;
- a synthesis agent distinguishes confirmed, corrected, disputed, and unresolved claims;
- each outbound response has a relay-accepted event ID;
- no node can exceed its configured capability policy;
- revoking one agent identity stops that node without disabling the other team members;
- all consequential work leaves a receipt.

## Governing invariant

```text
IDENTITY
-> MANDATE
-> DELEGATION
-> BOUNDED EXECUTION
-> CLAIM
-> VERIFICATION
-> CORRECTION
-> SYNTHESIS
-> RECEIPT
-> HUMAN REVIEW
```
