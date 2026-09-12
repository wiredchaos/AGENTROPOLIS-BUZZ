# BUZZ Spatial Intelligence Events

BUZZ carries signed coordination events for spatial intelligence without becoming the source of truth for the underlying observations.

## Event families

- `spatial.observation.updated`
- `spatial.source.degraded`
- `spatial.source.recovered`
- `spatial.scene.focused`
- `spatial.quantization.changed`
- `spatial.plan.recompile_requested`
- `spatial.aegis.blocked`
- `spatial.receipt.emitted`

## Minimum envelope

Every event should include event id, actor/agent identity, timestamp, correlation id, entity or scene reference, source/provenance references where applicable, and execution-envelope/receipt references for consequential workflows.

BUZZ events coordinate work. They do not replace ATLAS provenance, Ontology identity, ATG compilation, AEGIS policy, or Audit receipts.
