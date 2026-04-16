# Better Dispute

## A system for structured disagreement that produces clarity, agreement, and resolution.

Better Dispute turns discussion into a constrained, auditable, graph-based system where every claim, challenge, and answer is stored as an independent, queryable node.

It replaces replay-based simulation with deterministic state derived from structured queries.

---

## Table of Contents

> All entries are clickable links to sections below.

1. [Core Principles](#core-principles)  
2. [System Overview](#system-overview)  
3. [Architecture](#architecture)  
4. [MVC Contract](#mvc-contract)  
5. [Model](#model)  
6. [Storage Model](#storage-model)  
7. [Query Model](#query-model)  
8. [State Reconstruction](#state-reconstruction)  
9. [Core Rules](#core-rules)  
10. [Challenge Targeting](#challenge-targeting)  
11. [Turn Model](#turn-model)  
12. [Referenced Nodes](#referenced-nodes)  
13. [Resolution System](#resolution-system)  
14. [Crickets](#crickets)  
15. [Image Handling](#image-handling)  
16. [Abuse & Malicious Data](#abuse--malicious-data)  
17. [Missing Data](#missing-data)  
18. [Branching Model](#branching-model)  
19. [UI/UX](#uiux)  
20. [Controller Interface](#controller-interface)  
21. [Validation Rules](#validation-rules)  
22. [Failure Handling](#failure-handling)  
23. [Performance](#performance)  
24. [GitHub Integration Specifications](#github-integration-specifications)  
25. [SDLC](#sdlc)  
26. [Versioning](#versioning)  
27. [Release & Patch Notes](#release--patch-notes)  

---

## Core Principles

- Dialogue is structured through challenges and answers  
- The system enforces clarity, accountability, and progression  
- Outcomes prioritize:
  - Understanding  
  - Agreement  
  - Conflict resolution  

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
- Backend: GitHub Issues + Search API  
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

- 11-character base62  
- Client-generated  
- Must be globally unique  

---

### Person

- id  
- @name  
- githubId  

---

### Node (Issue)

Each node is stored as a GitHub Issue.

Types:
- Assertion (!)
- Challenge (?)
- Answer (✓)
- Resolution  

Fields (stored in issue body or metadata):

- id  
- type  
- authorId  
- disputeId  
- parentId  
- targetPersonId (challenge only)  
- contentText  
- contentPic (GitHub attachment URL)  
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

## Storage Model

- Each node = one GitHub Issue  
- Relationships defined via:
  - `disputeId`
  - `parentId`  
- Metadata stored in:
  - issue body (JSON block)
  - labels (type, status)  

Properties:

- append-only (no edits assumed)  
- tamper-tolerant via validation  
- eventually consistent  

---

## Query Model

State is derived using GitHub search/filter APIs.

Examples:

- All nodes in dispute:
  - `disputeId = X`

- Active challenge:
  - `type = challenge`
  - `status = open`

- Answers to challenge:
  - `parentId = challengeId`

Queries are:

- deterministic  
- idempotent  
- order-independent  

---

## State Reconstruction

No replay required.

State is computed as:

- Fetch relevant nodes  
- Filter valid nodes  
- Apply deterministic selection rules  

---

## Core Rules

- No self-challenges  
- One unresolved challenge per dispute  
- All permissions scoped per dispute  

---

## Challenge Targeting

Each challenge includes:

- targetPersonId  

Only target may answer.

---

## Turn Model

- Turn is derived from active challenge  
- `activeChallengeId` stored on dispute  

### Rule

- Only one challenge may have `status = open`  
- Creating a new challenge closes previous one  

---

## Referenced Nodes

- Nodes may reference external nodes  
- References do not mutate original  

---

## Resolution System

- Resolution is a node  
- Must be accepted by both parties  
- Dispute marked `resolved`  

---

## Crickets

Derived from timestamps:

```
currentTime - createdAt >= duration
```

---

## Image Handling

Images are handled via GitHub issue attachments.

### Upload Flow

1. User attaches image during issue creation  
2. GitHub uploads image to its CDN  
3. Markdown in issue body references the attachment  
4. Extracted CDN URL is stored as `contentPic`  

### Properties

- Images are immutable after upload  
- URLs are stable and platform-hosted  
- No external image dependencies  

### Constraints

- Subject to GitHub file size limits  
- No binary data stored outside platform  
- Duplicate uploads are not deduplicated  

---

## Abuse & Malicious Data

Invalid nodes ignored:

- schema violations  
- unauthorized actor  
- rule violations  

### Rate Heuristics

Clients may ignore:

- excessive node creation  
- spam patterns  

---

## Missing Data

- Missing nodes render as placeholders  
- No deletion supported  

---

## Branching Model

- Tree structure via `parentId`  
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
- Authorization validation  
- State rule validation  

Rules:

- Deterministic  
- Query-based  
- Independent of fetch order  

---

## Failure Handling

- retry writes  
- refetch queries  
- tolerate partial data  

---

## Performance

- query-based loading  
- partial fetches  
- caching per dispute  

---

## GitHub Integration Specifications

### OAuth

- OAuth-based identity  
- Token stored in memory  
- Used for read/write operations  

---

### Storage

- Each node = GitHub Issue  
- Queries via Search API  
- Labels + body used for indexing  

---

### Image Hosting

- Images stored as GitHub issue attachments  
- CDN URLs persisted as `contentPic`  
- No external hosting required  

---

## SDLC

Design → Implement → Test/Fix → Repeat → Deploy  

---

## Versioning

vMAJOR.MINOR.PATCH  

---

## Release & Patch Notes

> These notes track differences between deployed releases.  
> No deployments have been made yet.