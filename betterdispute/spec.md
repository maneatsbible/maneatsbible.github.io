# Better Dispute — Product Spec (Spec Kit)

## 1) Document Control
- **Product:** Better Dispute
- **Version:** v0.2 (draft)
- **Status:** Proposed for MVP
- **Scope:** Single-file browser app (`HTML + CSS + JS`) with production-grade behavior
- **Architecture:** Strict MVC (controller-authoritative)
- **Storage:** GitHub users for identity, GitHub Issues as append-only event store

## 2) Problem Statement
Most online disagreement is unstructured and spirals into noise. Better Dispute structures disagreement into explicit, auditable, turn-based exchanges between people so claims, questions, and answers can be tracked and resolved.

## 3) Objectives
1. Deliver a production-quality MVP in one HTML file, no frameworks.
2. Preserve immutable history through append-only issue events.
3. Enforce all business rules in controller methods.
4. Drive all UI state and controls from controller capability checks.
5. Make every post shareable by canonical URL.
6. Support duel-style dispute flow, including objections and crickets.

## 4) Non-Goals (MVP)
1. Multi-file frontend architecture.
2. Real-time sockets/push infrastructure.
3. Rich media beyond one optional image per post.
4. Full moderation platform (keep minimal guardrails only).
5. Private/distributed tenancy beyond one public data repo.

## 5) Product Principles
1. **Controller is source of truth.**
2. **View is dumb renderer.**
3. **All writes are append-only events.**
4. **Turn-state must always be obvious.**
5. **URL must reconstruct app context.**

## 6) Core Domain

### 6.1 Person
- GitHub-backed user identity.
- Fields: `id`, `githubId`, `handle` (e.g., `@name`), `isSystem`.
- `@strawman` is a reserved system person.

### 6.2 Post
- Types: `Assertion`, `Challenge`, `Answer`.
- Every post belongs to a rooted tree where root is an `Assertion`.
- Top-level assertion content constraint: **text XOR image** (exactly one).
- Non-root posts may include text and optional one image.
- `Challenge` subtypes:
  - `Interrogatory` (Yes/No)
  - `Objection` (form/validity/relevance/leading/etc.)
- `Answer` for `Interrogatory` requires `yesNo` plus optional text.

### 6.3 Dispute
- First-class object created by a challenge between two people.
- Unique `id`, participants, root assertion link, turn state, status.
- Counter-challenge supports turn-based duel continuation.
- Challenges inside disputes may spawn nested disputes.

### 6.4 CricketsCondition
- Negotiated timeout model for failure-to-answer outcomes.
- States: `proposed`, `agreed`, `disputed`, `rejected`, `expired`.

## 7) Controller Contract (Authoritative)
Controller methods (minimum):
- `canChallenge(person, post)`
- `canAnswer(person, challenge)`
- `canDispute(person, post)`
- `canAgree(person, assertion)`
- `canProposeCrickets(person, dispute)`
- `canResolve(person, dispute)`

Controller-enforced rules:
1. No person may challenge their own post.
2. A person may challenge a given post at most once (lifetime in MVP).
3. Only the targeted person may answer a challenge.
4. Controls are disabled if disallowed (not hidden).
5. Latest actionable post is highlighted.
6. Successful submit always refreshes state from source of truth.

## 8) View Contract (Dumb Rendering)
1. View reads controller state only.
2. View renders enabled/disabled controls from controller booleans.
3. View never decides permissions.
4. View never mutates model directly.

## 9) UX Specification

### 9.1 Global Shell
- Dark theme with selective colorful accents.
- Header: balances icon (home), “Better Dispute”, version (right).

### 9.2 Home View
- New top-level assertion composer:
  - text input
  - image input (URL for MVP)
  - post-as-`@strawman` option
- Summary cards of top-level assertions/disputes.
- Cards are clickable like X-style thread cards.
- “Your turn” badge where actor has pending move.
- Terminal cards (no challenges) shown subtly as non-actionable.

### 9.3 Dispute View
- Back button pops one level at a time to Home.
- Parent chain shown as arrows (no “lineage” label).
- Flattened duel projection of relevant subtree.
- Post icons: `!` assertion, `?` challenge, `✓` answer.
- Visual depth via slight card stacking.
- Challenge action opens inline slide-up composer.
- Answer composer:
  - required yes/no for interrogatory
  - optional text
  - optional counter-challenge field
- First counter-challenge switches layout to two lanes:
  - original lane left
  - counter lane right
  - chronologically interleaved entries
- Show “Your turn” indicator.
- Disable unavailable actions and highlight latest actionable post.

### 9.4 Notifications
- MVP in-app notifications:
  - “You were challenged”
  - “Your answer was challenged”
- Home-level awaiting-move badges.

## 10) URL & Routing Requirements
Canonical query params (MVP):
- `view=home|dispute`
- `root=<rootAssertionId>`
- `dispute=<disputeId>`
- `post=<postId>`

Requirements:
1. URL fully reconstructs current context.
2. Every post has copy-canonical-URL action.
3. Deep links work for internal/external sharing.

## 11) GitHub Storage Model (Append-Only)
1. One dedicated public GitHub repository stores app data.
2. Every event is a new GitHub issue (never mutate historical events for state transitions).
3. Issue body contains canonical JSON payload + readable content.
4. Labels index entity type and relational keys (root/dispute/parent/participants).
5. Canonical ordering: issue number ascending, tie-break by event id.
6. Invalid events are flagged by derived state, not deleted.

## 12) Security, Safety, and Reliability
1. Strict payload schema validation before writes.
2. Escape/sanitize rendered content to prevent XSS.
3. Constrain image sources and file characteristics.
4. Keep auth token in memory only for MVP.
5. Handle GitHub API errors, retries, and eventual consistency.
6. Throttle write actions client-side to reduce abuse/rate-limit pressure.

## 13) Clarifying Questions (to improve final product spec)
1. Should there be one global dispute repo forever, or future multi-tenant repos?
2. Is PAT-based in-browser auth acceptable for MVP, or do you want OAuth broker now?
3. Should observers be allowed to challenge directly, or must they “agree” first?
4. Do objections require categories, or free-form objection text only?
5. What exact visual rule should define a “terminal” non-clickable summary card?
6. Should challenge uniqueness reset after dispute resolution, or stay lifetime-unique?
7. What is the default crickets duration if no agreement is reached?
8. Should crickets countdown pause when GitHub API is unavailable?
9. Are “resolution offers” a dedicated post subtype or assertions with a resolution flag?
10. What minimum abuse controls are required at launch (report, mute, blocklist)?

## 14) Integrated MVP Decisions (assumptions applied now)
To unblock implementation, this draft integrates the following:
1. **Data tenancy:** single dedicated public repo.
2. **Auth:** in-session token for authenticated writes.
3. **Read mode:** anonymous read-only allowed.
4. **Challenge uniqueness:** lifetime one-per-person-per-target-post.
5. **Join rule:** must agree with root assertion before answering linked challenges.
6. **Terminal card rule:** root has no challenges.
7. **Crickets default:** 24 hours when no agreed condition exists.
8. **Objection model:** free-form text for MVP.
9. **Image input:** URL-based only in MVP (upload flow deferred).
10. **Notifications:** in-app only (no push/email).

## 15) Acceptance Criteria (MVP)
1. User can create root assertion with text XOR image.
2. User cannot challenge own posts.
3. Duplicate challenge by same person against same post is blocked.
4. Interrogatory answer requires Yes/No.
5. Counter-challenge can be submitted during answer flow.
6. First counter-challenge flips dispute view to two-lane layout.
7. “Your turn” indicators are correct on Home and Dispute views.
8. Disallowed actions are rendered disabled, never silently hidden.
9. Every post has working canonical URL copy action.
10. Deep links reconstruct view/dispute/post focus after refresh.
11. App runs from one HTML file with no external JS/CSS frameworks.
12. Derived state remains deterministic after reload.

## 16) Implementation Plan (Draft)

### Phase 0 — Foundation & Contracts
1. Define JSON schemas for `Person`, `Post`, `Dispute`, `CricketsCondition`, and event envelope.
2. Define deterministic serializer/parser and validator utilities.
3. Define controller action surface and state shape contract.

### Phase 1 — GitHub Data Integration
1. Implement API wrapper for GitHub Issues list/search/create with pagination and retry.
2. Implement in-memory auth session handling and token input UX.
3. Implement event query layer for root assertions, disputes, and post chains.

### Phase 2 — Model Reconstruction & Rule Engine
1. Reconstruct canonical model from append-only issue events.
2. Implement authoritative controller checks (`canChallenge`, `canAnswer`, etc.).
3. Enforce all action validations before any write.

### Phase 3 — Home View
1. Build app shell/header/version display.
2. Build new assertion composer with strawman option and image URL.
3. Render summary cards with turn badges and terminal-state visuals.

### Phase 4 — Dispute View
1. Render parent chain, timeline cards, and post-type icons.
2. Implement challenge and answer inline slide-up composers with draft persistence.
3. Implement turn indicators, disabled actions, and latest-actionable highlighting.
4. Implement automatic single-lane to two-lane transition on first counter-challenge.

### Phase 5 — Routing & Shareability
1. Implement query-param router and deep-link hydration.
2. Implement canonical URL builder for root/dispute/post scope.
3. Implement copy-link UX on every post.

### Phase 6 — Crickets & Resolution Flow
1. Implement proposing/disputing/agreeing crickets conditions.
2. Implement countdown expiration and prominent crickets event rendering.
3. Implement resolution-offer assertions and resolution transitions.

### Phase 7 — Hardening
1. Security pass: sanitization, strict validation, token handling.
2. Reliability pass: retries, stale-state refresh, conflict handling.
3. UX/accessibility pass: keyboard, focus, contrast, concise status cues.
4. Performance pass for large trees and summary-card lists.

## 17) Risks & Mitigations
1. **GitHub API rate limits** → caching, debouncing, manual refresh controls.
2. **Eventual consistency** → optimistic UI plus confirm-and-refresh cycle.
3. **Client-only auth risk** → memory-only token and clear MVP warning.
4. **Spam/abuse** → validation, local throttling, minimal moderation hooks.

## 18) Post-MVP Evolution
1. Move from single HTML file to modular architecture.
2. Add OAuth/GitHub App broker for secure auth.
3. Add stronger moderation, reputation, and policy controls.
4. Support richer media and upload workflows.
