# Better Dispute

## A deterministic system for structured disagreement.

Better Dispute models disagreement as an append-only graph of claims, challenges, and answers.

All system state is derived deterministically from queryable records. No server-side state is required.

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