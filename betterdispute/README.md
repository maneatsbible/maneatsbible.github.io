# Better Dispute

## A system for structured disagreement that produces clarity, agreement, and resolution.

Better Dispute models disagreement as an append-only graph of claims, challenges, and answers.

All system state is derived deterministically from queryable records. No server-side state is required.

---

## Table of Contents

> All entries below are clickable links to sections within this document.

1. [Core Principles](#core-principles)  
2. [Architecture](#architecture)  
3. [Model](#model)  
4. [Storage Model](#storage-model)  
5. [Query Model](#query-model)  
6. [Canonical Ordering](#canonical-ordering)  
7. [State Reconstruction](#state-reconstruction)  
8. [Active Challenge Rule](#active-challenge-rule)  
9. [Turn Model](#turn-model)  
10. [Validation Rules](#validation-rules)  
11. [Rate Limiting Strategy](#rate-limiting-strategy)  
12. [Failure Handling](#failure-handling)  
13. [Known Constraints](#known-constraints)  
14. [Future Improvements](#future-improvements)  

---

## Core Principles

- Structured dialogue through explicit challenge/answer relationships  
- Deterministic state derived from queries  
- Append-only, auditable data model  
- Client-side validation and reconstruction  

---

## Architecture

- Frontend: Browser (Vanilla JS)
- Backend: GitHub Issues (storage layer only)
- Auth: GitHub OAuth
- Pattern: Strict MVC
- No server-side state

---

## Model

### ID

All entities use **UUIDv7**:

- globally unique  
- time-ordered  
- lexicographically sortable  

---

### Node

Each node is stored as a GitHub Issue.

#### Types

- Assertion
- Challenge
- Answer
- Resolution

---

### Node Fields

- id (UUIDv7)
- type
- authorId
- disputeId
- parentId
- targetPersonId (challenge only)
- contentText
- contentPic
- contentHash
- createdAt
- status

---

### Content Hash

Each node includes a deterministic hash:

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

#### Rule

- Nodes with invalid hashes are ignored  
- Hash ensures integrity and tamper detection  

---

## Storage Model

- Each node = GitHub Issue  
- Relationships defined via:
  - disputeId
  - parentId  

Properties:

- append-only  
- eventually consistent  
- tamper-tolerant via validation  

---

## Query Model

State is derived via GitHub Search API.

All queries must be:

- deterministic  
- idempotent  
- order-independent  

---

## Canonical Ordering

All node sets MUST be sorted before selection:

```
ORDER BY:
  createdAt ASC,
  id ASC
```

This ensures deterministic behavior across clients.

---

## State Reconstruction

State is computed as:

1. Fetch relevant nodes  
2. Validate schema + hash  
3. Sort canonically  
4. Apply selection rules  

No replay is required.

---

## Active Challenge Rule

At most one challenge is considered active.

### Selection

```
activeChallenge =
  openChallenges
    .sorted(createdAt ASC, id ASC)
    .first()
```

All other open challenges are ignored.

---

## Turn Model

- Turn is derived from activeChallenge  
- Only targetPersonId may answer  

---

## Validation Rules

Nodes are ignored if:

- schema invalid  
- hash invalid  
- violates system rules  

Validation is deterministic and client-side.

---

## Rate Limiting Strategy

Due to GitHub Search API constraints:

- No real-time polling  
- Queries must be debounced (≥2s)  
- Aggressive caching per dispute  
- Manual refresh preferred over live sync  

---

## Failure Handling

- Retry writes  
- Refetch on failure  
- Tolerate partial data  

---

## Known Constraints

- GitHub Search API rate limits  
- Eventual consistency of search index  
- No hard prevention of malicious writes  
- Client-side validation is authoritative  

---

## Future Improvements

- Signed nodes (public/private key verification)  
- Indexed query layer (optional backend)  
- Improved moderation controls  