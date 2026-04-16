# Better Dispute

## A system for structured disagreement that produces clarity, agreement, and resolution.

Better Dispute turns discussion into a constrained, auditable, turn-based event system where every claim, challenge, and answer is replayable from a persistent event stream.

It replaces free-form argument with deterministic interaction graphs reconstructed entirely on the client.

---

## Table of Contents

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
28. [Patch Notes](#patch-notes)  

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

---

### Person

- id  
- @name  
- githubId  

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
- createdAt  

---

### Dispute

- id  
- rootAssertionId  
- participants[]  
- status  
- sequence  
- createdAt  

---

## Event Model

Each dispute = one persistent event stream  
Events are append-only and immutable  

### Constraints

- No binary data in events  
- Images must be external URLs only  
- Events must remain compact  

---

### Event Encoding

{
  "i": "eventId",
  "t": "TYPE",
  "a": "actorId",
  "s": 12,
  "ts": 1712345678901,
  "p": {}
}

---

### Event Types

- PC → POST_CREATED  
- CC → CHALLENGE_CREATED  
- AS → ANSWER_SUBMITTED  
- RO → RESOLUTION_OFFERED  
- RA → RESOLUTION_ACCEPTED  
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
- Sorted by sequence  
- Replayed through reducer  

### Constraints

- Event order is authoritative via sequence number  
- Storage is untrusted and validated client-side  
- Invalid or malicious events are ignored during replay  

---

## Boot & Replay on Load

1. Fetch event stream  
2. Parse events  
3. Sort by sequence  
4. Replay through reducer  
5. Drop invalid events  
6. Hydrate controller state  

Result:
> Deterministic reconstruction of full interaction state.

---

## State Reconstruction

- Pure reducer function  
- Deterministic replay  
- Sequence-based ordering  

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

---

## Crickets

Derived, not stored:

currentTime - proposalTime >= duration  

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

- invalid sequence  
- unauthorized actor  
- rule violations  
- duplicate actions  

---

## Missing Data

- Missing nodes render as placeholders  
- No deletion supported  

---

## Branching Model

All branches are collapsed into summary cards  

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

state + event → next state  

Invalid transitions ignored.

---

## Failure Handling

- retry writes  
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
- No server-side token storage exists  
- All requests are executed directly from browser to API  

---

### Append-only Event Storage

- Each Dispute maps to a single issue thread  
- Each event is appended as a new comment  
- Comments are treated as immutable history  
- Sequence number determines canonical ordering  
- Client performs full validation during replay  

Properties:
- append-only  
- tamper-tolerant (invalid events ignored)  
- eventually consistent  
- replayable to deterministic state  

---

### Image Hosting

- Images are uploaded via platform comment attachment flow  
- Platform returns hosted CDN URL (user-images domain)  
- Only URL is persisted in event payload (`contentPic`)  
- Images are not part of event state machine  
- Image loss is treated as missing external resource  
- No binary or base64 storage allowed anywhere in system  

---

## SDLC

Design → Implement → Test/Fix → Repeat → Deploy  

Patch → Test/Fix → Update README → Deploy  

---

## Versioning

vMAJOR.MINOR.PATCH  

---

## Patch Notes

No releases have been made yet.

---

## Release Notes

No releases have been made yet.