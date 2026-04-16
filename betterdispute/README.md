# Better Dispute

## A system for structured disagreement that produces clarity, agreement, and resolution.

Better Dispute turns discussion into a constrained, auditable, graph-based system where every claim, challenge, and answer is stored as an independent, queryable node.

It replaces replay-based simulation with deterministic state derived from structured queries.

---

## Table of Contents

> All entries below are clickable links to sections within this document.

1. [Core Principles](#core-principles)  
2. [System Overview](#system-overview)  
3. [Architecture](#architecture)  
4. [MVC Contract](#mvc-contract)  
5. [Model](#model)  
6. [Storage Model](#storage-model)  
7. [Query Model](#query-model)  
8. [Canonical Ordering](#canonical-ordering)  
9. [State Reconstruction](#state-reconstruction)  
10. [Core Rules](#core-rules)  
11. [Challenge Targeting](#challenge-targeting)  
12. [Turn Model](#turn-model)  
13. [Referenced Nodes](#referenced-nodes)  
14. [Resolution System](#resolution-system)  
15. [Crickets](#crickets)  
16. [Image Handling](#image-handling)  
17. [Abuse & Malicious Data](#abuse--malicious-data)  
18. [Missing Data](#missing-data)  
19. [Branching Model](#branching-model)  
20. [UI/UX](#uiux)  
21. [Controller Interface](#controller-interface)  
22. [Validation Rules](#validation-rules)  
23. [Failure Handling](#failure-handling)  
24. [Performance](#performance)  
25. [GitHub Integration Specifications](#github-integration-specifications)  
26. [Rate Limiting Strategy](#rate-limiting-strategy)  
27. [Known Constraints](#known-constraints)  
28. [SDLC](#sdlc)  
29. [Versioning](#versioning)  
30. [Release & Patch Notes](#release--patch-notes)  

---

## Core Principles

- Dialogue is structured through challenges and answers  
- The system enforces clarity, accountability, and progression  
- Deterministic state derived from queries  
- Append-only, auditable data model  

---

## System Overview

Better Dispute is a query-driven state system.

- Each node = independent record  
- GitHub = distributed document store  
- Browser = deterministic query engine  
- State = derived from queries, not replay  

---

## Architecture

- Frontend: Vanilla JavaScript  
- Backend: GitHub Issues  
- Authentication: OAuth-based identity layer  
- Pattern: Strict MVC  
- No server-side state is maintained  

---

## MVC Contract

### Controller (Authoritative Layer)

Runs entirely in browser:

- Enforces all rules  
- Executes queries  
- Computes derived state  
- Rejects invalid actions before write  

---

### View (Dumb Renderer)

- Reads controller state only  
- No business logic  
- Pure rendering layer  

---

## Model

### ID Format

- UUIDv7  
- Globally unique  
- Time-ordered  

---

### Person

- id  
- @name  
- githubId  

---

### Node (Issue)

Each node is stored as a GitHub Issue.

Types:
- Assertion  
- Challenge  
- Answer  
- Resolution  

Fields:

- id  
- type  
- authorId  
- disputeId  
- parentId  
- targetPersonId  
- contentText  
- contentPic  
- contentHash  
- createdAt  
- status  

---

### Dispute

- id  
- rootNodeId  
- participants[]  
- status  
- activeChallengeId  
- createdAt  

---

### Content Hash

```
contentHash = SHA-256(
  type +
  authorId +
  disputeId +
  parentId +
  contentText +
  createdAt
)
```

---

## Storage Model

- Each node = GitHub Issue  
- Relationships via disputeId and parentId  
- Append-only  
- Eventually consistent  

---

## Query Model

State is derived using GitHub search APIs.

Queries are:

- deterministic  
- idempotent  
- order-independent  

---

## Canonical Ordering

```
ORDER BY:
  createdAt ASC,
  id ASC
```

---

## State Reconstruction

1. Fetch nodes  
2. Validate schema + hash  
3. Sort canonically  
4. Apply selection rules  

---

## Core Rules

- No self-challenges  
- One unresolved challenge per dispute  
- Permissions scoped per dispute  

---

## Challenge Targeting

Each challenge includes targetPersonId.

Only the target may answer.

---

## Turn Model

- Derived from active challenge  

---

## Referenced Nodes

Nodes may reference external nodes without mutation.

---

## Resolution System

- Resolution is a node  
- Must be accepted by both parties  
- Marks dispute resolved  

---

## Crickets

Derived from timestamps:

```
currentTime - createdAt >= duration
```

---

## Image Handling

- Upload via GitHub endpoint  
- CDN URL embedded  
- Stored as contentPic  

---

## Abuse & Malicious Data

Invalid nodes ignored:

- schema violations  
- hash mismatch  
- rule violations  

---

## Missing Data

- Missing nodes render as placeholders  
- No deletion supported  

---

## Branching Model

- Tree via parentId  
- Multiple branches allowed  
- UI collapses non-relevant branches  

---

## UI/UX

- Dark theme  
- Minimal interaction model  
- Emphasis on actionable state  

---

## Controller Interface

- getCurrentPerson()  
- switchPerson()  
- getDispute()  
- getNode()  
- getAvailableActions()  

---

## Validation Rules

- Schema validation  
- Hash validation  
- State rule validation  

---

## Failure Handling

- Retry writes  
- Refetch queries  
- Tolerate partial data  

---

## Performance

- Query-based loading  
- Partial fetches  
- Client caching  

---

## GitHub Integration Specifications

### OAuth

- Token stored in memory  
- Used for API access  

---

### Storage

- Issues used as node store  
- Labels + body for indexing  

---

### Image Hosting

- GitHub upload endpoint  
- CDN-backed URLs  

---

## Rate Limiting Strategy

- No real-time polling  
- Debounced queries (≥2s)  
- Cache per dispute  
- Manual refresh preferred  

---

## Known Constraints

- GitHub Search API limits  
- Eventual consistency  
- No hard write prevention  
- Client-side validation  

---

## SDLC

Design → Implement → Test → Deploy  

---

## Versioning

vMAJOR.MINOR.PATCH  

---

## Release & Patch Notes

No releases yet.