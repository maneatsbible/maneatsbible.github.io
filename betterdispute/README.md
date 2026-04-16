# Better Dispute

## A system for structured disagreement that produces clarity, agreement, and resolution.

Better Dispute turns discussion into a constrained, auditable, turn-based system where every claim, challenge, and answer is traceable, replayable, and enforceable.

It replaces free-form argument with structured interaction graphs that can be reconstructed from an immutable event log.

---

## Table of Contents

1. [Core Principles](#core-principles)  
2. [System Overview](#system-overview)  
3. [Architecture](#architecture)  
4. [MVC Contract](#mvc-contract)  
5. [Model](#model)  
6. [Event Model (GitHub)](#event-model-github)  
7. [Boot & Replay on Load](#boot--replay-on-load)  
8. [State Reconstruction](#state-reconstruction)  
9. [Core Rules](#core-rules)  
10. [Challenge Targeting](#challenge-targeting)  
11. [Turn Model](#turn-model)  
12. [Referenced Nodes](#referenced-nodes)  
13. [Resolution System](#resolution-system)  
14. [Crickets](#crickets)  
15. [Abuse & Malicious Events](#abuse--malicious-events)  
16. [Missing Data](#missing-data)  
17. [Branching Model](#branching-model)  
18. [UI/UX](#uiux)  
19. [Controller Interface](#controller-interface)  
20. [Reducer Specification](#reducer-specification)  
21. [Failure Handling](#failure-handling)  
22. [Performance](#performance)  
23. [SDLC](#sdlc)  
24. [Versioning](#versioning)  
25. [Patch Notes](#patch-notes)  

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

Better Dispute models disagreement as a deterministic event graph.

Every interaction:
- is immutable
- is replayable
- is derived from GitHub issue events

The system behaves like a local simulation engine with GitHub acting as the persistence layer.

---

## Architecture

- Frontend: Vanilla JavaScript  
- Backend: GitHub Issues (append-only event log)  
- Auth: GitHub OAuth  
- Pattern: Strict MVC  
- Controller state: **fully client-side (no server-side state)**  

---

## MVC Contract

### Controller (Authoritative Layer)

- Runs entirely in the browser  
- Reconstructs all state from GitHub events  
- Enforces rules locally before write  
- Validates transitions during replay  

Responsibilities:

- Enforce all rules  
- Determine permissions  
- Compute derived state  
- Replay full history deterministically  
- Reject invalid actions before commit  

---

### View (Dumb Renderer)

- Reads controller state only  
- Never computes rules  
- Fully reactive to controller output  

---

## Model

### ID Format

- 11-character base62 strings  
- Globally unique  
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

## Event Model (GitHub)

Each Dispute = one GitHub Issue  
Comments = append-only event stream  

### Storage Constraints

- No binary data stored in GitHub comments  
- Images must be external URLs only  
- Events must remain compact (<65KB comment limit)  

---

### Event Encoding

```json
{
  "i": "eventId",
  "t": "TYPE",
  "a": "actorId",
  "s": 12,
  "ts": 1712345678901,
  "p": {}
}
```

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

## Boot & Replay on Load

On application startup:

### Step 1 — Fetch
- Load GitHub Issue for current dispute
- Retrieve all comments (events)

### Step 2 — Normalize
- Parse each comment into event objects
- Validate schema

### Step 3 — Sort
- Sort by sequence `s` ascending

### Step 4 — Reduce
- Run deterministic reducer:
  - state0 → event1 → state1 → event2 → state2 …

### Step 5 — Validate
- Drop invalid events during replay
- Log inconsistencies (dev mode)

### Step 6 — Hydrate Controller
- Final state becomes active runtime state

Result:
> The browser becomes a deterministic simulation of GitHub history.

---

## State Reconstruction

- Pure function reducer  
- Deterministic replay  
- Sequence is authoritative ordering  

---

## Core Rules

- No self-challenges  
- One unresolved challenge at a time  
- All permissions scoped to dispute  

---

## Challenge Targeting

Each challenge includes:
- targetPersonId  

Only target may answer.

---

## Turn Model

- Only one unresolved challenge exists  
- Turn = target of unresolved challenge  

Counter-challenge replaces unresolved challenge.

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

Derived from timestamps  
No stored timers  

---

## Abuse & Malicious Events

Invalid events are ignored during replay:

- invalid sequence  
- unauthorized actor  
- rule violations  

---

## Missing Data

Missing nodes render as placeholders  
No deletion supported  

---

## Branching Model

All branches are collapsed into summary cards  

---

## UI/UX

- Dark theme  
- Minimal interaction surface  
- Emphasis on actionable state  

---

## Controller Interface

Controller runs entirely in browser:

- getCurrentPerson()  
- switchPerson()  
- getHomeFeed()  
- getDispute()  
- getPost()  
- getAvailableActions()  

---

## Reducer Specification

Pure deterministic state machine:

state + event → next state  

All invalid transitions ignored.

---

## Failure Handling

- retry GitHub writes  
- refetch on conflict  
- full replay after recovery  

---

## Performance

- lazy event loading  
- incremental replay  
- caching per dispute  

---

## SDLC

Design → Implement → Test/Fix → Repeat → Deploy  

Patch → Test/Fix → Update README → Deploy  

---

## Versioning

vMAJOR.MINOR.PATCH  

---

## Patch Notes

(To be appended)
