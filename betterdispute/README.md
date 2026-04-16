# Better Dispute

## A system for structured disagreement that produces clarity, agreement, and resolution.

Better Dispute turns discussion into a constrained, auditable, turn-based event system where every claim, challenge, and answer is replayable from a persistent event stream.

It replaces free-form argument with deterministic interaction graphs reconstructed entirely on the client.

---

## Table of Contents

> All entries are clickable links to sections below.

1. [Core Principles](#core-principles)  
2. [System Overview](#system-overview)  
3. [Architecture](#architecture)  
4. [MVC Contract](#mvc-contract)  
5. [Model](#model)  
6. [Event Model](#event-model)  
7. [Event Stream Storage](#event-stream-storage)  
8. [Boot & Replay on Load](#boot--replay-on-load)  
9. [State Reconstruction](#state-reconstruction)  
10. [Core Rules](#core-rules)  
11. [Challenge Targeting](#challenge-targeting)  
12. [Turn Model](#turn-model)  
13. [Referenced Nodes](#referenced-nodes)  
14. [Resolution System](#resolution-system)  
15. [Crickets](#crickets)  
16. [Image Handling](#image-handling)  
17. [Abuse & Malicious Events](#abuse--malicious-events)  
18. [Missing Data](#missing-data)  
19. [Branching Model](#branching-model)  
20. [UI/UX](#uiux)  
21. [Controller Interface](#controller-interface)  
22. [Reducer Specification](#reducer-specification)  
23. [Failure Handling](#failure-handling)  
24. [Performance](#performance)  
25. [GitHub Integration Specifications](#github-integration-specifications)  
26. [SDLC](#sdlc)  
27. [Versioning](#versioning)  
28. [Release & Patch Notes](#release--patch-notes)  

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

Better Dispute is an event-sourced simulation system.

- Event stream = immutable history  
- Browser = deterministic simulation engine  
- Controller state = fully client-side  

---

## Architecture

- Frontend: Vanilla JavaScript  
- Backend: Append-only event stream storage layer  
- Authentication: OAuth-based identity layer  
- Pattern: Strict MVC  
- No server-side state is maintained  

---

## MVC Contract

### Controller (Authoritative Layer)

Runs entirely in browser:

- Enforces all rules  
- Reconstructs state via replay  
- Computes permissions  
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
- publicKey  

---

### Post

Types:
- Assertion (!)
- Challenge (?)
- Answer (✓)

Fields:
- id  
- type  
- authorId  
- parentId  
- disputeId  
- contentText  
- contentPic (external URL only)  
- contentHash  
- createdAt  

---

### Dispute

- id  
- rootAssertionId  
- participants[]  
- status  
- createdAt  

---

## Event Model

Each dispute = one persistent event stream  
Events are append-only and immutable  

---

### Constraints

- No binary data in events  
- Images must be external URLs only  
- Events must remain compact  

---

### Event Encoding

```json
{
  "i": "eventId",
  "v": 1,
  "t": "TYPE",
  "a": "actorId",
  "prev": "previousEventId",
  "ts": 1712345678901,
  "sig": "signature",
  "p": {}
}
```

---

### Event Guarantees

- `prev` forms a hash-linked chain of events  
- Only events with valid signatures are accepted  
- Event order is derived from chain linkage, not sequence numbers  
- Duplicate events (same `i`) are ignored  
- Events must be idempotent  

---

### Event Types

- PC → POST_CREATED  
- CC → CHALLENGE_CREATED  
- AS → ANSWER_SUBMITTED  
- RO → RESOLUTION_OFFERED  
- RA → RESOLUTION_ACCEPTED  
- RF → RESOLUTION_FINALIZED  
- CP → CRICKETS_PROPOSED  
- CA → CRICKETS_AGREED  
- CT → CRICKETS_TRIGGERED  
- CD → CRICKETS_DISPUTED  
- MU → MUTED  
- BL → BLOCKED  

---

## Event Stream Storage

The event stream is stored as an append-only log inside an external platform-backed issue thread.

### Storage Mechanism

- Each dispute maps to a single issue thread  
- Each event is stored as a single comment  
- Comments are append-only by design  

### Retrieval

On load:

- All comments are fetched via API  
- Parsed into event objects  
- Reconstructed into a valid event chain  
- Longest valid chain is selected  
- Replayed through reducer  

### Constraints

- Storage is untrusted and validated client-side  
- Invalid or malicious events are ignored during replay  

---

## Boot & Replay on Load

1. Fetch event stream  
2. Parse events  
3. Verify signatures  
4. Build valid event chains  
5. Select longest valid chain  
6. Replay through reducer  
7. Drop invalid events  
8. Hydrate controller state  

Result:
> Deterministic reconstruction of full interaction state.

---

## State Reconstruction

- Pure reducer function  
- Deterministic replay based on chain order  
- Only valid events are applied  

---

## Core Rules

- No self-challenges  
- One unresolved challenge at a time per dispute  
- All permissions scoped per dispute  

---

## Challenge Targeting

Each challenge includes:

- targetPersonId  

Only target may answer.

---

## Turn Model

- Exactly one unresolved challenge exists per dispute  
- Turn = target of unresolved challenge  

### Concurrency Rule

- First valid challenge in replay order is accepted  
- Subsequent concurrent challenges are rejected  

Counter-challenge replaces current unresolved challenge.

---

## Referenced Nodes

- External posts may be referenced  
- Visually distinct  
- Do not mutate original context  

---

## Resolution System

- Offers are assertions  
- Must be mutually accepted  
- Finalized via `RF` event  
- No further events allowed after finalization  

---

## Crickets

Derived, not stored:

Uses:
- latest event timestamp instead of wall clock  

```
latestEventTime - proposalTime >= duration
```

---

## Image Handling

Images are handled via platform attachment flow.

### Upload Flow

1. Image attached during comment creation  
2. Platform uploads image to CDN  
3. Platform returns hosted URL  
4. Only URL is stored in event payload  

### Constraints

- No base64 storage  
- No binary event data  
- Images are external immutable resources  

---

## Abuse & Malicious Events

Invalid events are ignored during replay:

- invalid signature  
- broken chain linkage  
- unauthorized actor  
- rule violations  
- duplicate eventId  

### Rate Heuristics

Clients may ignore:

- excessive rapid events from same actor  
- high volumes of invalid events  

---

## Missing Data

- Missing nodes render as placeholders  
- No deletion supported  

---

## Branching Model

- Multiple chains may exist  
- Only the longest valid chain is canonical  
- Non-canonical branches are ignored  

---

## UI/UX

- Dark theme  
- Minimal interaction model  
- Emphasis on actionable state  

---

## Controller Interface

- getCurrentPerson()  
- switchPerson()  
- getHomeFeed()  
- getDispute()  
- getPost()  
- getAvailableActions()  

---

## Reducer Specification

```
state + event → next state
```

### Validation Layers

1. Schema validation  
2. Signature validation  
3. Authorization validation  
4. State transition validation  

Rules:

- Reducer must be pure  
- Deterministic across clients  
- Only prior accepted events influence validation  
- Invalid transitions are ignored  

---

## Failure Handling

- retry writes with same eventId (idempotent)  
- refetch on conflict  
- full replay on recovery  

---

## Performance

- lazy event loading  
- incremental replay  
- caching per dispute  

---

## GitHub Integration Specifications

### OAuth

- Authentication uses OAuth-based identity provider  
- Client obtains access token via standard OAuth flow  
- Token is stored only in memory (client-only controller constraint)  
- Token is used to:
  - identify Person identity (`githubId`)
  - authorize event stream reads/writes  

---

### Append-only Event Storage

- Each Dispute maps to a single issue thread  
- Each event is appended as a new comment  
- Comments are treated as immutable history  
- Clients reconstruct canonical chain via `prev` linkage  
- Client performs full validation during replay  

Properties:
- append-only  
- tamper-evident (via signatures + chain)  
- eventually consistent  
- replayable to deterministic state  

---

### Image Hosting

- Images are uploaded via platform comment attachment flow  
- Platform returns hosted CDN URL  
- Only URL is persisted in event payload (`contentPic`)  
- Image loss is treated as missing external resource  

---

## SDLC

Design → Implement → Test/Fix → Repeat → Deploy  

Patch → Test/Fix → Update README → Deploy  

### Deployment Constraint

- Each deployment must produce a versioned `bd-vX.Y.Z.html`
- `index.html` must always point to latest versioned file  

### Notes Requirement

- Every release and patch must include:
  - summary of changes  
  - version number  
  - timestamp  
  - GPT model used to generate the changes  

---

## Versioning

vMAJOR.MINOR.PATCH  

---

## Release & Patch Notes

> These notes track differences between deployed releases.  
> No deployments have been made yet.