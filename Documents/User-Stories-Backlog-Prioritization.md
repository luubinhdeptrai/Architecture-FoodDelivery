# Backlog — User Stories Grouping, Story Points, and Priority (MVP / Release 1)

Source: [User-Stories-and-Acceptance-Criteria.md](User-Stories-and-Acceptance-Criteria.md)

Assumptions for estimates:
- Story Points use Fibonacci (1, 2, 3, 5, 8, 13) and represent relative effort across mobile + backend + integrations.
- Estimates include typical MVP-level UI + API work + basic validation, but not large-scale refactors.

---

## Priority Legend

- **P0 (MVP Core)**: Required to support an end-to-end order flow (browse → cart → checkout → fulfill → track) with minimal operations.
- **P1 (MVP Ops / Enhancements)**: Important for smooth operations and support, but not strictly required for the first end-to-end demo.
- **P2 (Post-MVP / Governance & Reporting depth)**: Valuable, but can follow once core flows are stable.

---

## Grouped Backlog (by Role)

### Customer

| ID | Story (short) | SP | Priority | Key dependencies / notes |
|---:|---|---:|:---:|---|
| US-1 | Registration/Login | 5 | P0 | Needed for identity + order ownership.
| US-2 | Browse/Search Restaurants | 5 | P0 | Depends on restaurant onboarding + menu data.
| US-3 | Search/Filter Food Items by Category & Proximity | 8 | P1 | Depends on location/deliverability logic; adds major discovery value.
| US-4 | View Restaurant Availability | 3 | P0 | Depends on restaurant availability data (US-12).
| US-5 | Single-Restaurant Cart Constraint | 2 | P0 | Cart rule used by checkout; enforce consistently.
| US-6 | Delivery Address Within Radius | 5 | P0 | Depends on geocoding/distance; impacts checkout feasibility.
| US-7 | Checkout With COD or VNPay | 13 | P0 | Depends on cart + address + lifecycle; VNPay integration is a major spike.
| US-9 | Real-Time Order Status Updates | 8 | P0 | Depends on lifecycle events (US-8) and status changes from partner/shipper flows.
| US-22 | Shopping Cart Management | 5 | P0 | Depends on menu items (US-11) and rule (US-5).

### Restaurant Partner

| ID | Story (short) | SP | Priority | Key dependencies / notes |
|---:|---|---:|:---:|---|
| US-10 | Restaurant Onboarding Approval | 5 | P0 | Depends on admin access + approvals (US-25, US-18).
| US-11 | Restaurant Menu Management | 8 | P0 | Drives customer browse + cart.
| US-12 | Availability Control (sold out/closed) | 5 | P0 | Drives US-4 + prevents bad orders.
| US-13 | Accept/Reject Incoming Orders | 8 | P0 | Depends on checkout + lifecycle; includes timeout + alerting.
| US-23 | Update Preparation Status | 3 | P1 | Depends on lifecycle + notifications; improves tracking quality.
| US-24 | Cancel Order With Reason | 3 | P1 | Depends on lifecycle; also ties to customer notification (FR-2.4).

### Shipper (Delivery Personnel)

| ID | Story (short) | SP | Priority | Key dependencies / notes |
|---:|---|---:|:---:|---|
| US-14 | Shipper Onboarding Approval | 3 | P0 | Depends on admin access + approvals (US-25, US-18).
| US-15 | Availability Toggle | 3 | P0 | Needed before dispatch/assignment can be meaningful.
| US-16 | Accept Job & Confirm Pickup | 5 | P0 | Depends on order assignment/dispatch logic (implicit in MVP dispatch).
| US-17 | Confirm Delivery | 3 | P0 | Completes the end-to-end order flow.

### System Administrator

| ID | Story (short) | SP | Priority | Key dependencies / notes |
|---:|---|---:|:---:|---|
| US-18 | Approve/Reject Partners | 5 | P0 | Requires secure admin access (US-25).
| US-19 | Monitor Order/Platform Health | 5 | P1 | Builds operational confidence; can follow core flow.
| US-25 | Admin Dashboard Access & RBAC | 8 | P0 | Foundation for all admin stories.
| US-26 | Search User Accounts | 5 | P1 | Depends on user data model; supports operations.
| US-27 | Suspend/Reactivate Partners | 3 | P1 | Depends on partner status model + enforcement.
| US-28 | Monitor Orders & View Details | 5 | P1 | Depends on order model + status history.
| US-29 | Cancel Order With Reason | 3 | P1 | Depends on lifecycle constraints; ties to notifications.
| US-30 | Configure Commission % & History | 5 | P2 | Depends on commission model (US-21).
| US-31 | Reports & CSV Export | 8 | P2 | Depends on stable data + calculations.
| US-32 | Audit Log for Actions | 8 | P2 | Cross-cutting; implement after core flows stabilize (or earlier if required).

### Platform / Internal

| ID | Story (short) | SP | Priority | Key dependencies / notes |
|---:|---|---:|:---:|---|
| US-8 | Order Lifecycle Integrity | 8 | P0 | Foundation for tracking + enforcement across all roles.
| US-20 | Geographic Scope Restriction | 3 | P0 | Should be enforced early to keep MVP controlled.
| US-21 | Commission Calculation | 5 | P2 | Depends on delivered orders + payment confirmation; supports financial governance.

---

## Recommended Implementation Order (Dependency-aware)

This is a practical build order that keeps the system runnable at each step.

### Phase 1 — Foundations + Marketplace Data (get real catalog + governance)
1) **US-8 (Lifecycle)** — define statuses/transitions early (everything else relies on this).
2) **US-25 (Admin access + RBAC)** — secure operations entrypoint.
3) **US-18 (Approve/reject partners)** — enable onboarding.
4) **US-10 (Restaurant onboarding)** + **US-14 (Shipper onboarding)** — create supply-side actors.
5) **US-11 (Menu management)** + **US-12 (Availability control)** — create real catalog/menu data.
6) **US-20 (Geographic restriction)** — enforce MVP operational scope.

### Phase 2 — Customer Ordering (first end-to-end ordering demo)
7) **US-1 (Customer auth/profile)**
8) **US-2 (Browse/search restaurants)** + **US-4 (Availability visibility)**
9) **US-5 (Single-restaurant constraint)** + **US-22 (Cart management)**
10) **US-6 (Address within radius)**
11) **US-7 (Checkout COD/VNPay)**

### Phase 3 — Fulfillment + Tracking (make orders actually complete)
12) **US-13 (Restaurant accept/reject)**
13) **US-15 (Shipper availability)**
14) **US-16 (Pickup)** + **US-17 (Delivery confirmation)**
15) **US-9 (Real-time status updates)** — wire lifecycle events through to customers (can start earlier with polling, then upgrade).

### Phase 4 — Operations hardening + discoverability improvements
16) **US-23 (Prep status updates)**
17) **US-24 (Restaurant cancel w/ reason)** + **US-29 (Admin cancel w/ reason)**
18) **US-28 (Admin order monitoring)** + **US-19 (Health monitoring)**
19) **US-26 (Search user accounts)** + **US-27 (Suspend/reactivate partners)**
20) **US-3 (Item search/filter by proximity)** — high-value discovery enhancement once base catalog + deliverability is stable.

### Phase 5 — Finance + Reporting + Audit (post-MVP stabilization)
21) **US-21 (Commission calculation)**
22) **US-30 (Commission configuration + history)**
23) **US-31 (Reports + CSV export)**
24) **US-32 (Audit log)**

---

## Notes / Risks

- **US-7 (VNPay)** is a large, integration-heavy story; consider splitting internally (payment initiation vs callback/webhook handling vs idempotency) even if you keep it as one user-facing story.
- **Dispatch/assignment** is implied by US-16/US-15; if your SRS expects a specific dispatch rule (nearest shipper, round-robin, etc.), consider adding an explicit story for “Assign shipper to order”.
