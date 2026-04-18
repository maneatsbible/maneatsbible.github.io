# Better Dispute — Product Spec (Spec Kit)

## 1) Document Control
- **Product:** Better Dispute
- **Version:** v0.1 (draft)
- **Status:** Proposed
- **Scope:** Browser-only vanilla JavaScript app in a single HTML file for initial production release
- **Primary runtime:** Modern desktop/mobile browsers

## 2) Problem Statement
Unstructured disagreement in comments/threads often produces confusion and escalation instead of clarity. Better Dispute structures disagreement as a governed duel between two people, centered on clear challenges and explicit yes/no answers, with append-only records and shareable canonical URLs.

## 3) Goals
1. Deliver a production-grade MVP in one HTML file (no external frameworks).
2. Use GitHub identity + GitHub Issues as append-only storage.
3. Enforce rules in controller only (strict MVC).
4. Provide Home and Dispute views with turn-based duel UX.
5. Make every post shareable via canonical URL.
6. Support assertion/challenge/answer flow, objections, and dispute resolution offers.

## 4) Non-Goals (MVP)
1. Real-time websockets/push infrastructure.
2. Multi-file frontend architecture (deferred until post-MVP).
3. Rich media beyond one optional image per post.
4. Formal moderation/appeals system beyond basic abuse controls.
5. Cryptographic proof beyond deterministic validation and GitHub timestamps.

## 5) Product Principles
1. **Controller authority:** all eligibility and business rules are enforced before writes.
2. **Dumb view:** view renders state and control availability only.
3. **Append-only records:** immutable posts; corrections occur via new posts.
4. **Explicit turn state:** users should always see whose move it is.
5. **URL-first navigation:** app state is reproducible from URL params.

## 6) Personas
- **Disputer:** participates directly in a duel, challenges and answers.
- **Observer:** reads disputes and may join by agreeing with an assertion.
- **Strawman initiator:** starts top-level assertions as `@strawman`.

## 7) Core User Stories
1. As a user, I can start a top-level assertion (text OR image).
2. As a user, I can challenge another person’s post once per target post.
3. As challenged user, I can answer with required Yes/No plus optional text.
4. As answerer, I can counter-challenge in the same turn.
5. As user, I can object to challenge form/validity.
6. As user, I can open any post/dispute from canonical URL.
7. As user, I can see “Your turn” and actionable controls.
8. As user, I can copy canonical post URL from any post.
9. As user, I can propose and negotiate crickets timeout conditions.
10. As user, I can propose resolution offers as assertions.

## 8) Functional Requirements

### 8.1 Identity and Access
- Users are GitHub users represented as `Person` with stable `id`, `@name`, `githubId`.
- App supports authenticated actor context and unauthenticated read-only mode.
- `@strawman` is system-defined and cannot authenticate.

### 8.2 Post Types
- `Assertion`, `Challenge`, `Answer`.
- `Challenge` has subtype: `Interrogatory` (Y/N) or `Objection`.
- `Answer` to interrogatory requires `yesNo` value; optional text.

### 8.3 Tree and Dispute Model
- Every post is a node in a tree rooted at an assertion.
- A challenge creates or links to a first-class `Dispute` with two participants.
- Dispute may spawn nested disputes when eligible third-party or branch challenges occur.

### 8.4 Controller Rules (authoritative)
- No self-challenge.
- A person may challenge a specific post at most once.
- Only eligible target may answer a challenge.
- Controls rendered disabled when disallowed (never hidden by permission).
- Latest actionable post is highlighted.
- View refreshes after successful write.

### 8.5 Home View
- Header: home icon (balances), title, version.
- New assertion composer: text, optional image, optional post-as-`@strawman`.
- Summary cards for top-level assertions/disputes.
- “Your turn” badges on cards with pending actor move.
- Non-clickable terminal cards are visually subdued.

### 8.6 Dispute View
- Back navigation pops level to Home.
- Parent chain shown with arrows.
- Flattened duel projection of subtree.
- Icons: Assertion `!`, Challenge `?`, Answer `✓`.
- Single lane initially; switches to dual lane after first counter-challenge.
- Inline slide-up composer for challenge/answer; per-post draft text preserved on cancel.

### 8.7 URL and Navigation
- Canonical URL params must fully address a root assertion and optional dispute/post focus.
- Deep links restore correct view and highlighted context.
- Copy button returns canonical URL for the specific post.

### 8.8 Notifications (MVP)
- In-app notification center with unread indicators for:
  - “You were challenged”
  - “Your answer was challenged”
- Home card badges indicate awaiting user move.

### 8.9 Crickets
- Disputers can propose countdown timeout conditions.
- Conditions are challengeable/disputable.
- Crickets event is surfaced prominently when timeout elapses without valid answer.

## 9) Non-Functional Requirements
1. **Framework-free:** no external JS/CSS libraries.
2. **Single-file deliverable:** one HTML containing CSS+JS for MVP.
3. **Deterministic state reconstruction:** same inputs produce same state.
4. **Performance:** initial render target <2s on typical broadband for small disputes.
5. **Resilience:** tolerate eventual consistency and partial GitHub fetch failures.
6. **Accessibility:** keyboard focus states, semantic controls, readable contrast.
7. **Auditability:** append-only writes; immutable history display.

## 10) Domain Model

### 10.1 Person
- `id` (UUID)
- `name` (`@handle`)
- `githubId`
- `isSystem` (for `@strawman`)

### 10.2 Post
- `id` (UUID)
- `type` (`assertion|challenge|answer`)
- `challengeType` (`interrogatory|objection|null`)
- `authorPersonId`
- `targetPersonId` (required for challenge)
- `rootAssertionId`
- `parentPostId` (null only for top-level assertion)
- `disputeId`
- `text`
- `imageUrl` (optional)
- `yesNo` (`yes|no|null`)
- `createdAt`
- `status` (`active|superseded|resolved|invalid`)

### 10.3 Dispute
- `id` (UUID)
- `rootAssertionId`
- `participantA`, `participantB`
- `state` (`open|resolved|crickets|closed`)
- `activeChallengePostId`
- `turnPersonId`
- `createdAt`

### 10.4 CricketsCondition
- `id`
- `disputeId`
- `proposedBy`
- `durationSeconds`
- `status` (`proposed|agreed|disputed|rejected`)

## 11) GitHub Storage Specification (Append-Only)
- Backend store: GitHub Issues in a dedicated repository.
- Each post/dispute event persisted as a new issue.
- Issue body stores canonical JSON payload + human-readable text block.
- Labels index entity type and keys (e.g., root, dispute, parent, participants).
- Canonical ordering uses GitHub issue number then entity id.
- Never mutate prior issues for state changes; emit new event issue.

## 12) Security & Abuse Constraints
1. Reject malformed payloads in controller.
2. Escape all rendered user content to prevent XSS.
3. Validate and constrain image URLs.
4. Minimize token exposure (memory-only session token, no localStorage for auth token).
5. Rate-limit write attempts client-side.
6. Mark invalid events without deleting them.

## 13) UX Design Constraints
- Dark theme baseline with selective accent colors.
- Minimalist controls and icon-first actions.
- Disabled state is visible and explainable (tooltip/hint).
- Chronology is explicit and scannable in duel timeline.

## 14) Clarifying Questions (Consultant Review)
1. Should storage be in one global repo, one repo per community, or one repo per dispute domain?
2. Must all dispute data be public, or do we require private repositories?
3. What is the required identity flow for browser-only auth (PAT, OAuth app + backend broker, or GitHub App via companion service)?
4. Are anonymous read-only users allowed for all public disputes?
5. What are final canonical URL parameters and permanence guarantees?
6. Is one challenge per person per post lifetime-only, or reset after resolution?
7. How should third-party participants join a dispute after “agree with assertion”?
8. What exact rules determine terminal non-clickable summary cards?
9. What default crickets duration should apply before negotiated agreement?
10. What abuse controls are mandatory at launch (blocklist, report, profanity filters)?

## 15) Integrated Improvements / Assumptions for MVP
To unblock implementation, this spec adopts the following defaults until product decisions change:
1. **Repository scope:** one dedicated public GitHub repo for all Better Dispute data.
2. **Auth:** authenticated writes require user-supplied GitHub token in-session (temporary MVP compromise).
3. **Read mode:** public read-only access without login.
4. **Challenge uniqueness:** one challenge per person per target post for all time.
5. **Crickets default:** 24h timeout if no agreed custom condition exists.
6. **Terminal card:** non-clickable when root assertion has no challenges.
7. **Join rule:** third party must explicitly agree with root assertion before answering linked challenges.
8. **URL schema:** `?view=home|dispute&root=<id>&dispute=<id>&post=<id>`.
9. **Notifications:** in-app only for MVP, no email/push.
10. **Image upload MVP:** accept URL input first; direct upload deferred.

## 16) Acceptance Criteria (MVP)
1. User can create assertion (text OR image) and see it on Home.
2. Challenge button availability always matches controller rules.
3. Answer form enforces yes/no where required.
4. Counter-challenge transitions dispute view from one lane to two lanes.
5. Every post has working copy-canonical-URL action.
6. Deep link opens correct view and context.
7. “Your turn” indicators appear correctly on Home and Dispute views.
8. Disabled controls are visible where action is disallowed.
9. App reconstructs identical state after browser refresh.
10. No external framework/library is loaded.

## 17) Implementation Plan (Draft)

### Phase 0 — Foundations
1. Define canonical schemas and validation helpers in single HTML module sections.
2. Implement deterministic parser/serializer for issue payloads.
3. Add centralized error/event bus and UI state store.

### Phase 1 — GitHub Integration
1. Build API client for list/search/create issues with pagination and retry.
2. Implement auth session handling (token input + memory-only storage).
3. Implement query layer for roots, disputes, and post timelines.

### Phase 2 — Controller + Rules Engine
1. Implement model reconstruction from issue events.
2. Implement `canChallenge`, `canAnswer`, `canDispute`, `canAgree`, `canProposeCrickets`.
3. Enforce business rules at action dispatch.

### Phase 3 — View Layer
1. Build app shell/header/version.
2. Build Home view: assertion composer, strawman option, summary cards, badges.
3. Build Dispute view: lineage, lane projection, inline composers, action buttons.
4. Implement disabled state UX and latest-actionable highlighting.

### Phase 4 — URL Routing + Shareability
1. Implement router parsing and state hydration from query params.
2. Implement canonical URL builder per post/dispute/root.
3. Implement copy link action and deep-link restore.

### Phase 5 — Crickets + Resolution
1. Implement crickets condition offers and disputes.
2. Implement resolution offer assertions and agreement transitions.
3. Render crickets state prominently in timeline and cards.

### Phase 6 — Hardening
1. Input sanitization and XSS-safe rendering.
2. Performance passes (memoization/cache, selective re-render).
3. Accessibility pass (keyboard, labels, focus, contrast).
4. Manual QA matrix across desktop/mobile browsers.

## 18) Risks and Mitigations
1. **GitHub API rate limits** → cache, debounce, backoff, manual refresh affordance.
2. **Browser-only auth limitations** → clearly mark MVP auth compromise, plan broker service.
3. **Eventual consistency** → refresh controls + optimistic-but-verified UI state.
4. **Data abuse/spam** → validation, visibility filtering, and report hooks.

## 19) Post-MVP Evolution
1. Split single HTML into modular files while preserving behavior.
2. Introduce secure auth broker service.
3. Add richer moderation and reputation controls.
4. Add richer media and attachment handling.
