# Better Dispute

## Table of Contents

1. [Core Principles](#core-principles)  
2. [Architecture](#architecture)  
3. [MVC Contract](#mvc-contract)  
4. [Model](#model)  
5. [Event Model (GitHub)](#event-model-github)  
6. [State Reconstruction](#state-reconstruction)  
7. [Core Rules](#core-rules)  
8. [Challenge Targeting](#challenge-targeting)  
9. [Turn Model](#turn-model)  
10. [Referenced Nodes](#referenced-nodes)  
11. [Resolution System](#resolution-system)  
12. [Crickets](#crickets)  
13. [Abuse & Malicious Events](#abuse--malicious-events)  
14. [Missing Data](#missing-data)  
15. [Branching Model](#branching-model)  
16. [UI/UX](#uiux)  
17. [Controller Interface](#controller-interface)  
18. [Reducer Specification](#reducer-specification)  
19. [Failure Handling](#failure-handling)  
20. [Performance](#performance)  
21. [SDLC](#sdlc)  
22. [Versioning](#versioning)  
23. [Patch Notes](#patch-notes)  

---

## Core Principles

- Dialogue is structured through challenges and answers  
- The system enforces clarity, accountability, and progression  
- Outcomes prioritize:
  - Understanding  
  - Agreement  
  - Conflict resolution  

---

## Architecture

- Frontend: Vanilla JavaScript  
- Backend: GitHub  
  - Auth: GitHub OAuth  
  - Data: GitHub Issues (append-only event log)  
- Pattern: Strict MVC  

---

## MVC Contract

### Controller (Authoritative Layer)

Responsibilities:

- Enforce all rules  
- Determine permissions  
- Compute derived state  
- Reconstruct full state deterministically  
- Reject invalid actions pre-write  
- Ignore invalid actions during reconstruction  

---

### View (Dumb Renderer)

- Reads controller state only  
- Never determines permissions  
- Reflects controller output exactly  

---

## Model

### ID Format

- 11-character base62 strings  
- Globally unique  
- Client-generated with retry  

---

### Person

- `id`  
- `@name`  
- `githubId`  

---

### Post

Types:

- Assertion `!`  
- Challenge `?`  
- Answer `✓`  

Structure:

- `id`  
- `type`  
- `authorId`  
- `parentId`  
- `disputeId`  
- `contentText`  
- `contentPic` (optional URL)  
- `createdAt`  

Rules:

- Root must be Assertion  
- Root Assertion: text OR pic  
- Other posts: text + optional pic  

---

### Dispute

- `id`  
- `rootAssertionId`  
- `participants[]`  
- `status`: active | resolved | abandoned  
- `sequence`  
- `createdAt`  

---

## Event Model (GitHub)

Each Dispute = one Issue  
Comments = append-only events  

### Limits

- Comment size limit enforced (~65KB)  
- Payloads must be compact  
- Pics must be external URLs  

---

### Encoding Format (Compact JSON)

All events use compact keys to minimize payload size.

#### Base Structure

```json
{
  "i": "evtId",
  "t": "type",
  "a": "actorId",
  "s": 12,
  "ts": 1712345678901,
  "p": { }
}
```

Fields:

- `i` → event id  
- `t` → type  
- `a` → actor id  
- `s` → sequence  
- `ts` → timestamp (ms)  
- `p` → payload  

---

### Event Types (Codes)

- `PC` → POST_CREATED  
- `CC` → CHALLENGE_CREATED  
- `AS` → ANSWER_SUBMITTED  
- `RO` → RESOLUTION_OFFERED  
- `RA` → RESOLUTION_ACCEPTED  
- `CP` → CRICKETS_PROPOSED  
- `CA` → CRICKETS_AGREED  
- `CT` → CRICKETS_TRIGGERED  
- `CD` → CRICKETS_DISPUTED  
- `MU` → PARTICIPANT_MUTED  
- `BL` → PARTICIPANT_BLOCKED  

---

### Payload Schemas

#### POST_CREATED (`PC`)

```json
{
  "pi": "postId",
  "pt": "A|C|N",
  "pa": "parentId|null",
  "tx": "text",
  "pc": "picUrl|null"
}
```

#### CHALLENGE_CREATED (`CC`)

```json
{
  "pi": "postId",
  "ci": "challengeId",
  "tp": "targetPersonId",
  "ct": "I|O"
}
```

#### ANSWER_SUBMITTED (`AS`)

```json
{
  "ci": "challengeId",
  "ai": "answerId",
  "yn": true,
  "tx": "optional text"
}
```

#### COUNTER-CHALLENGE (embedded in AS)

```json
{
  "cc": {
    "ci": "challengeId",
    "tp": "targetPersonId",
    "ct": "I|O",
    "tx": "text"
  }
}
```

#### RESOLUTION_OFFERED (`RO`)

```json
{
  "pi": "postId"
}
```

#### RESOLUTION_ACCEPTED (`RA`)

```json
{
  "pi": "postId"
}
```

#### CRICKETS_PROPOSED (`CP`)

```json
{
  "ci": "challengeId",
  "d": 60000
}
```

#### CRICKETS_AGREED (`CA`)

```json
{
  "ci": "challengeId"
}
```

---

## State Reconstruction

### Ordering

- Strictly by sequence  

---

### Reducer

- Pure  
- Deterministic  
- Validates transitions  
- Ignores invalid events  

---

### Idempotency

- Unique event IDs  
- Duplicate-safe  

---

### Concurrency

- Client submits with expectedSequence  
- First valid write wins  
- Others refetch and retry  

---

## Core Rules

- A Person cannot challenge their own Post  
- A Person may challenge a Post once per Dispute  
- All permissions scoped to Dispute  
- Posts globally addressable  

---

## Challenge Targeting

Every Challenge MUST include:

- targetPersonId

Rules:

- Target must be author of challenged Post  
- Only target may answer  

---

## Turn Model

- Only one unresolved Challenge may exist  

Turn:

- Target of unresolved Challenge  

After answer:

- Optional counter-challenge becomes next unresolved Challenge  

---

## Referenced Nodes

- External posts may be referenced  
- Visually distinct  
- Read-only projection  

Rules:

- Can be challenged locally  
- Do not affect original context  

---

## Resolution System

### Offers

- Assertions proposed within Dispute  
- Must be accepted  

---

### Agreement

- Occurs when participants accept same Assertion  

---

## Crickets

Derived:

currentTime - proposalTime >= duration

Requires agreement  

---

## Abuse & Malicious Events

Reducer rejects:

- Invalid sequence  
- Unauthorized actor  
- Turn violations  
- Duplicate challenges  

Invalid events are ignored  

---

## Missing Data

- Missing nodes rendered as placeholders  
- No deletion  

---

## Branching Model

- All branches collapsed into summary cards  

---

## UI/UX

### Home View

- Composer: "Start a fire... 🔥"  
- Feed sorted:
  1. Your turn  
  2. Activity  
  3. Recency  

---

### Dispute View

- Parent chain  
- Duel projection  
- Nested summary cards  

---

### Layout

- Single lane  
- Two lane after counter  

---

### Post UI

- Icons: ! ? ✓  
- Copy link  
- Depth stacking  

---

### Interaction

- Challenge primary  
- Answer Yes/No + optional  

---

### Terminal Nodes

- Not clickable  
- Fully challengeable  

---

### Notifications

- Challenged  
- Answer challenged  

---

### Test Mode

- Multiple Persons  
- Switchable sessions  

---

## Controller Interface

### Identity

- getCurrentPerson()  
- switchPerson(personId)  

---

### Queries

- getHomeFeed()  
- getDispute(disputeId)  
- getPost(postId)  
- getLineage(postId)  
- getAvailableActions(personId, context)  

---

### Permissions

- canChallenge(personId, postId)  
- canAnswer(personId, challengeId)  
- canCounterChallenge(personId, answerId)  
- canAgree(personId, assertionId)  
- canOfferResolution(personId, disputeId)  
- canAcceptResolution(personId, offerId)  
- canProposeCrickets(personId, challengeId)  
- canRespondCrickets(personId, proposalId)  

---

### Actions

- createAssertion(input)  
- createChallenge(input)  
- submitAnswer(input)  
- submitCounterChallenge(input)  
- agreeAssertion(input)  
- offerResolution(input)  
- acceptResolution(input)  
- proposeCrickets(input)  
- respondCrickets(input)  
- triggerCrickets(input)  
- muteParticipant(input)  
- blockParticipant(input)  

---

## Reducer Specification

### State Shape

state = {
  posts: Map,
  challenges: Map,
  answers: Map,
  participants: Set,
  unresolvedChallengeId: null
}

---

### POST_CREATED (PC)

- Add post  
- Add participant  

---

### CHALLENGE_CREATED (CC)

Valid if:

- No unresolved challenge  

Effects:

- Create challenge  
- Set unresolved  

---

### ANSWER_SUBMITTED (AS)

Valid if:

- Actor is target  

Effects:

- Add answer  
- Clear unresolved  

---

### COUNTER-CHALLENGE

- Same as CC  

---

### RESOLUTION_OFFERED (RO)

- Store offer  

---

### RESOLUTION_ACCEPTED (RA)

- Mark agreement  

---

### CRICKETS_PROPOSED (CP)

- Store  

---

### CRICKETS_AGREED (CA)

- Enable derived check  

---

### INVALID EVENTS

Ignored  

---

## Failure Handling

- Retry  
- Refetch  
- Re-render  

---

## Performance

- Lazy loading  
- Incremental reconstruction  
- Caching  

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
