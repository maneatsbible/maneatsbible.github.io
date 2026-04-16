# Better Dispute

## A system for structured disagreement that produces clarity, agreement, and resolution.

Better Dispute turns discussion into a constrained, auditable, turn-based event system where every claim, challenge, and answer is replayable from GitHub history.

It replaces free-form argument with deterministic interaction graphs reconstructed entirely on the client.

---

## Table of Contents

1. Core Principles  
2. System Overview  
3. Architecture  
4. MVC Contract  
5. Model  
6. Event Model (GitHub)  
7. Boot & Replay on Load  
8. State Reconstruction  
9. Core Rules  
10. Challenge Targeting  
11. Turn Model  
12. Referenced Nodes  
13. Resolution System  
14. Crickets  
15. Image Handling  
16. Abuse & Malicious Events  
17. Missing Data  
18. Branching Model  
19. UI/UX  
20. Controller Interface  
21. Reducer Specification  
22. Failure Handling  
23. Performance  
24. SDLC  
25. Versioning  
26. Patch Notes  

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

Better Dispute is an event-sourced simulation.

- GitHub Issues = immutable event log  
- Browser = deterministic simulation engine  
- Controller state = fully client-side  

---

## Architecture

- Frontend: Vanilla JavaScript  
- Backend: GitHub Issues (append-only event stream)  
- Auth: GitHub OAuth  
- Pattern: Strict MVC  
- No server-side state exists  

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

## Event Model (GitHub)

Each Dispute = GitHub Issue  
Comments = append-only event stream  

### Constraints

- No binary data in events  
- Images must be external URLs only  
- Events must remain compact (<65KB/comment)  

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

On startup:

1. Fetch GitHub issue comments  
2. Parse into events  
3. Sort by sequence  
4. Replay deterministically through reducer  
5. Drop invalid events  
6. Hydrate controller state  

Result:
> Browser becomes a deterministic replay engine of GitHub history.

---

## State Reconstruction

- Pure reducer function  
- Deterministic replay  
- Sequence is authoritative  

---

## Core Rules

- No self-challenges  
- One unresolved challenge at a time  
- All permissions scoped per dispute  

---

## Challenge Targeting

Each challenge includes:

- targetPersonId  

Only target may answer.

---

## Turn Model

- Exactly one unresolved challenge exists globally per dispute  
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

## Image Handling (OPTION 1)

Images are handled via GitHub Issue attachment flow.

### Upload Flow

1. Client attaches image in issue comment creation request  
2. GitHub uploads image to:
   ```
   https://user-images.githubusercontent.com/...
   ```
3. GitHub returns markdown with hosted URL  
4. System extracts URL from response  
5. Event stores ONLY the URL in `contentPic`  

### Constraints

- No raw image storage in events  
- No base64 encoding  
- No custom CDN required  
- Images are treated as external immutable resources  

### Properties

- CDN-backed by GitHub  
- Stable but not guaranteed permanent  
- Fully decoupled from event replay logic  

---

## Abuse & Malicious Events

Invalid events are ignored during replay:

- invalid sequence  
- unauthorized actor  
- rule violations  
- duplicate challenges  

---

## Missing Data

- Missing nodes render as placeholders  
- No deletion supported  

---

## Branching Model

All branches collapsed into summary cards  

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

Pure state machine:

state + event → next state  

Invalid transitions are ignored.

---

## Failure Handling

- retry GitHub writes  
- refetch on conflict  
- full replay on recovery  

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

## Release Notes

No releases yet.

### Patch Notes

No patches yet.

