# Technical Decisions — Finance Approval Service

> Architecture Decision Records (ADRs) and design choices specific to `resi-complex-finance-approval-service`.

---

### DEC-FIN-001: Append-Only Board Votes and Duplicate Vote Prevention
* **Status:** Accepted
* **Context:** Colombian condominium law (Ley 675 de 2001) requires transparent and non-repudiable minutes of Board decisions on extraordinary expenses.
* **Decision:** Enforce `existsByProposalIdAndBoardMemberId` before recording any vote and disallow update/delete operations on `expense_votes`.
* **Consequences:** Complete auditability of who voted, when, and with what rationale.

---

### DEC-FIN-002: Synchronous Quorum Evaluation on Vote Submission
* **Status:** Accepted
* **Context:** Waiting for a manual administrator action to close a proposal after votes are cast creates ambiguity.
* **Decision:** Evaluate `approvedCount` and `rejectedCount` immediately inside `castVote()`. When either count reaches the quorum majority threshold, transition the proposal to `APPROVED` or `REJECTED` and set `approvalDate`.
* **Consequences:** Immediate deterministic state transition and prevents late votes on already-decided proposals.
