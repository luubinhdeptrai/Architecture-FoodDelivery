# Quality Attributes (Easy Guide for a Food Delivery App)

Source of truth: This guide strictly follows the meanings in [14 Quality Attribute.md](14%20Quality%20Attribute.md).

---

## 1) Supportability (Hỗ trợ)

### 1. Simple Explanation
Supportability means the system helps you **find and fix problems while people are using it** by giving useful information (not just “something went wrong”).

### 2. Real-world Analogy
A car that shows a **clear dashboard warning + error code** (“Low oil pressure”) is supportable. A car that just stops with no clue is not.

### 3. Concrete Example (Software – Food Delivery App)
When orders get stuck in `Preparing`, the system shows **which threshold was breached**, logs the timeline, and links to the restaurant/shipper involved so ops can act quickly.

### 4. Bad vs Good Comparison
- Low: “500 error” with no request ID; logs missing; support can’t reproduce.
- High: Clear error message for user + request ID; structured logs; traces show where latency/failure happened.

### 5. How to Design for It (Practical)
- Add **structured logging** (JSON logs) with `requestId`, `userId/role`, `orderId`, `errorCode`.
- Use **centralized log/trace correlation** (traceId across NestJS services).
- Expose **operational dashboards** (stuck orders, queue depth, error spikes).
- Keep **audit trails** for admin actions (who/what/when).

### 6. How to Measure It
- ≥ 95% of errors have a non-empty `errorCode` and `requestId`.
- MTTR (mean time to resolve incident) < 30 minutes for common failures.
- 100% of admin actions write an audit entry (or are blocked/retried per policy).

### 7. Common Mistakes
- Confusing supportability with “friendly UI” only; it’s mainly about **diagnostic information**.
- Logging too much sensitive data (tokens, full card/PII) instead of safe identifiers.

---

## 2) Testability (Kiểm thử)

### 1. Simple Explanation
Testability means the system makes it **easy to define tests and run tests** to check whether it meets the criteria.

### 2. Real-world Analogy
A classroom test is testable when questions have **clear answers** and grading rules. It’s not testable if grading depends on “vibes.”

### 3. Concrete Example (Software – Food Delivery App)
“Commission rate must be 0–100%” is testable: you can write automated tests for boundary values and persistence.

### 4. Bad vs Good Comparison
- Low: Requirements like “fast” or “secure enough” with no pass/fail criteria.
- High: Clear acceptance criteria + deterministic rules you can unit/integration test.

### 5. How to Design for It (Practical)
- Make rules **deterministic** (validation rules, state transitions).
- Separate business logic into **pure services** (easy unit tests).
- Provide **test environments** (seed data, test DB, fake payment provider).
- Add **idempotency** so retries don’t create duplicates (easier to test).

### 6. How to Measure It
- ≥ 80% of critical business rules have automated tests.
- CI pipeline completes within 10 minutes (fast feedback).
- Flaky test rate < 1% per week.

### 7. Common Mistakes
- Thinking testability = “high test coverage.” Coverage helps, but testability starts with **clear criteria**.
- Coupling logic tightly to UI/DB so it’s hard to test without full system running.

---

## 3) Availability (Sẵn sàng)

### 1. Simple Explanation
Availability means **how much time the system is up and usable** (often as a percentage over a period).

### 2. Real-world Analogy
A convenience store that’s open and stocked most of the time has high availability. If it’s often closed, it has low availability.

### 3. Concrete Example (Software – Food Delivery App)
Customers can browse restaurants and place orders even during peak times without the app going down.

### 4. Bad vs Good Comparison
- Low: Frequent outages at lunch/dinner peaks.
- High: Service stays up even when traffic spikes or one server crashes.

### 5. How to Design for It (Practical)
- Run multiple backend instances + **load balancing**.
- Use **health checks** and auto-restart (process manager / container orchestration).
- Add **graceful degradation** (if recommendations fail, still show search & ordering).
- Use DB backups + replication strategy appropriate for your needs.

### 6. How to Measure It
- Uptime ≥ 99.9% monthly for core ordering APIs.
- Error budget: ≤ 43 minutes downtime/month at 99.9%.
- Successful request rate ≥ 99.5% over rolling 5 minutes.

### 7. Common Mistakes
- Mixing availability with performance: a slow system might still be “up,” but unusable; you need both.
- Single points of failure (one DB instance, one Redis, one server).

---

## 4) Interoperability (Hợp tác)

### 1. Simple Explanation
Interoperability means the system can **work with other systems** and makes it easy to **exchange and reuse data**.

### 2. Real-world Analogy
A USB-C charger works with many devices. A proprietary charger works with only one.

### 3. Concrete Example (Software – Food Delivery App)
Exporting reports to CSV with stable columns so accounting can import it into spreadsheets or BI tools.

### 4. Bad vs Good Comparison
- Low: Hard-coded, undocumented formats; every integration needs manual fixes.
- High: Stable APIs/contracts; predictable data formats; versioning.

### 5. How to Design for It (Practical)
- Use **well-defined REST contracts** (OpenAPI) and version endpoints.
- Use **stable identifiers** (UUIDs for orders/users).
- Export data in **standard formats** (CSV/JSON) with stable headers/fields.
- Isolate third-party providers behind **adapter interfaces** (payment, maps, notifications).

### 6. How to Measure It
- Time to add a new integration (e.g., payment gateway) < 2 weeks.
- Backward compatibility: 0 breaking API changes within a major version.
- CSV export: column headers unchanged across releases (unless versioned).

### 7. Common Mistakes
- Treating interoperability as “we have an API.” It also needs **stable contracts and reusable data**.
- No versioning; clients break when you change fields.

---

## 5) Manageability (Quản lý)

### 1. Simple Explanation
Manageability means admins have **tools to manage the system, find faults, and tune it**.

### 2. Real-world Analogy
A restaurant kitchen is manageable if there’s a clear order board, timers, and roles. If everyone shouts orders with no tracking, it’s chaos.

### 3. Concrete Example (Software – Food Delivery App)
Admin can filter orders by status, see stuck orders, suspend a partner, and adjust thresholds without database access.

### 4. Bad vs Good Comparison
- Low: Only developers can fix problems by running SQL in production.
- High: Admin dashboard + safe configs + clear operational controls.

### 5. How to Design for It (Practical)
- Build an **admin control plane** (RBAC, partner status, order intervention).
- Make key thresholds **configuration-based** (timeouts, stuck detection) with validation.
- Provide **operational metrics** (queue depth, order lifecycle distribution).
- Add **feature flags** to turn risky features on/off safely.

### 6. How to Measure It
- Time to suspend a fraudulent partner < 2 minutes.
- Time to change a threshold (with audit) < 10 minutes.
- % of operational actions possible without direct DB access ≥ 95%.

### 7. Common Mistakes
- Confusing manageability with supportability. Supportability = diagnose faults; manageability = **control and tune**.
- Exposing dangerous admin actions without RBAC and audit trails.

---

## 6) Performance (Hiệu năng)

### 1. Simple Explanation
Performance is how fast the system responds when executing requests **within a specified time**.

### 2. Real-world Analogy
A fast cashier line processes customers quickly; a slow line causes people to leave.

### 3. Concrete Example (Software – Food Delivery App)
Restaurant search results load quickly so users don’t abandon checkout.

### 4. Bad vs Good Comparison
- Low: Search takes 8–10 seconds; app feels frozen.
- High: Search feels instant and remains fast at peak time.

### 5. How to Design for It (Practical)
- Add DB **indexes** for frequent queries (restaurant search, orders by status).
- Use **caching** where safe (restaurant list, menu data, config).
- Use pagination and **limit payload size**.
- For tracking, push updates via **WebSocket** rather than polling.

### 6. How to Measure It
- p95 API response time: search ≤ 2s, order creation ≤ 1s.
- p95 WebSocket message delivery ≤ 1s.
- Database query p95 latency ≤ 100ms for top endpoints.

### 7. Common Mistakes
- Measuring only average latency; p95/p99 usually matters more.
- Making endpoints do too much work (N+1 queries, huge payloads).

---

## 7) Reliability (Tin cậy)

### 1. Simple Explanation
Reliability means the system **keeps working correctly over time** and does not fail its intended functions during a time period.

### 2. Real-world Analogy
A bus schedule is reliable if buses actually arrive as promised day after day, not just once.

### 3. Concrete Example (Software – Food Delivery App)
An order should not randomly jump states or get duplicated; payment confirmation should not mark the wrong order as paid.

### 4. Bad vs Good Comparison
- Low: Duplicate orders on retry; inconsistent status history; “paid but no order.”
- High: Correct state transitions; idempotent operations; consistent notifications.

### 5. How to Design for It (Practical)
- Enforce a **state machine** for orders (valid transitions only).
- Use **database transactions** for multi-step updates.
- Implement **idempotency keys** for create/confirm actions.
- Use **outbox pattern** for reliable event/notification sending.

### 6. How to Measure It
- Failed business transactions < 0.1% (order placement, payment confirm).
- Duplicate order rate = 0 (or < 0.001%).
- Consistency checks: 100% orders have valid status transition history.

### 7. Common Mistakes
- Treating reliability as “no downtime” (that’s availability). Reliability is also about **correctness over time**.
- Ignoring retries and partial failures (they create duplicates and corruption).

---

## 8) Scalability (Mở rộng)

### 1. Simple Explanation
Scalability means the system can handle **more load** without hurting performance, often by scaling resources (more servers, more capacity).

### 2. Real-world Analogy
A café can scale by adding more baristas and machines so wait times don’t explode during rush hour.

### 3. Concrete Example (Software – Food Delivery App)
At dinner peak, you can handle 10× more concurrent users by running more NestJS instances and optimizing the DB.

### 4. Bad vs Good Comparison
- Low: Traffic spike causes timeouts and crashes.
- High: Adding instances increases throughput predictably.

### 5. How to Design for It (Practical)
- Make backend **stateless** so you can scale horizontally.
- Use **connection pooling** for PostgreSQL and avoid per-request new connections.
- Separate read-heavy workloads (search) from write-heavy (orders) with caching or read replicas when appropriate.
- For WebSocket, plan for **sticky sessions** or a message broker.

### 6. How to Measure It
- Throughput scales: doubling instances yields ≥ 1.8× throughput (same p95 latency).
- Supports N concurrent connections (e.g., 50k WebSocket clients) within p95 delivery target.
- No single query causes CPU > 80% for sustained periods.

### 7. Common Mistakes
- Scaling only app servers while the DB becomes the bottleneck.
- Storing user session state in-memory on one server (breaks horizontal scaling).

---

## 9) Security (An ninh)

### 1. Simple Explanation
Security means the system **resists unintended actions** and **protects important data**.

### 2. Real-world Analogy
A locked door + ID check protects a building. A sign saying “staff only” without enforcement does not.

### 3. Concrete Example (Software – Food Delivery App)
RBAC prevents a normal user from accessing admin endpoints; sensitive data is protected even if someone tries to call APIs directly.

### 4. Bad vs Good Comparison
- Low: Admin actions protected only by UI; API accepts requests from anyone.
- High: Server-side authorization checks, audit logs, secure storage.

### 5. How to Design for It (Practical)
- Enforce **authentication + authorization** on every request (NestJS guards).
- Use **least privilege** roles (customer/restaurant/shipper/admin).
- Protect data with **encryption in transit** (TLS) and secure secrets handling.
- Add **rate limiting** and abuse detection for login/order endpoints.

### 6. How to Measure It
- 0 critical auth bypass vulnerabilities in security testing.
- 100% admin endpoints require RBAC.
- % requests over HTTPS = 100%.
- Account takeover signals: suspicious login rate monitored; lockouts after N failures.

### 7. Common Mistakes
- Believing “we use JWT” automatically means secure.
- Logging secrets/PII; forgetting to secure internal admin APIs.

---

## 10) Conceptual Integrity (Toàn vẹn khái niệm)

### 1. Simple Explanation
Conceptual integrity means the system design feels **consistent and coherent**—components and modules follow the same core ideas.

### 2. Real-world Analogy
A school with one clear set of rules and consistent grading feels coherent. If every teacher uses different rules and terms, students get confused.

### 3. Concrete Example (Software – Food Delivery App)
“Order status” means the same thing everywhere: same names, same transitions, same source of truth (not one meaning in the DB and another in the UI).

### 4. Bad vs Good Comparison
- Low: Three different meanings of “active restaurant” across services.
- High: Shared domain model; consistent APIs; one source of truth per concept.

### 5. How to Design for It (Practical)
- Define a clear **domain model** (Order, Restaurant, Shipper, Commission) and stick to it.
- Use consistent **API conventions** (naming, error format, pagination).
- Keep architecture boundaries clear (modules/bounded contexts).
- Use ADRs (architecture decision records) to keep consistency across teams.

### 6. How to Measure It
- API consistency: ≥ 95% endpoints follow standard error schema + pagination rules.
- Duplicate concept count: 0 duplicated “status” enums across modules (or tracked and justified).
- Architecture review: 0 high-severity violations of module boundaries per release.

### 7. Common Mistakes
- Thinking conceptual integrity is “beauty.” It’s about **consistency that reduces confusion and bugs**.
- Copy-pasting models independently across services without governance.

---

## 11) Flexibility (Mềm dẻo)

### 1. Simple Explanation
Flexibility means the system can **adapt to different situations/environments** and handle changes in **policies and business rules**, often through configuration.

### 2. Real-world Analogy
A backpack with adjustable straps fits different people. A fixed-size bag doesn’t.

### 3. Concrete Example (Software – Food Delivery App)
Changing commission rate rules, service area rules, or “stuck order” thresholds without rewriting core code.

### 4. Bad vs Good Comparison
- Low: Any policy change requires redeploying and touching many modules.
- High: Policy changes are configuration-driven with validation and history.

### 5. How to Design for It (Practical)
- Use **configuration** for thresholds/rates (with validation and audit history).
- Use **feature flags** to roll out rules gradually.
- Isolate business rules in a **policy module** (not spread across controllers).
- Design extensible schemas (e.g., commission history table).

### 6. How to Measure It
- Time to change a policy (commission rate / threshold) < 30 minutes end-to-end.
- % of policy changes done by configuration (no code change) ≥ 80%.
- Rollback time for a bad config change < 10 minutes.

### 7. Common Mistakes
- Flexibility is not “anything goes.” Without constraints it becomes chaos.
- Adding too many flags/configs without ownership and documentation.

---

## 12) Maintainability (Có thể bảo trì)

### 1. Simple Explanation
Maintainability means the system can **accept change** (new features, bug fixes) and changes don’t create huge ripple effects.

### 2. Real-world Analogy
A LEGO model is maintainable: you can replace one part without breaking everything. A glued model is not.

### 3. Concrete Example (Software – Food Delivery App)
Adding “scheduled delivery” should not require rewriting payment, tracking, and restaurant modules.

### 4. Bad vs Good Comparison
- Low: Small change breaks many unrelated features.
- High: Changes are localized; clear module interfaces; easy debugging.

### 5. How to Design for It (Practical)
- Split code into clear modules (auth, orders, restaurants, payments).
- Keep interfaces stable (DTOs, service contracts).
- Use consistent error handling and logging.
- Keep automated tests to prevent regressions.

### 6. How to Measure It
- Lead time for change: small feature delivered in < 3 days.
- Regression rate: < 5% of releases require hotfix.
- Mean time to understand a module (new dev onboarding) < 2 days.

### 7. Common Mistakes
- Confusing maintainability with “clean code” only; architecture boundaries matter.
- Over-engineering abstractions that make changes harder.

---

## 13) Reusability (Tái sử dụng)

### 1. Simple Explanation
Reusability means parts of the system can be **reused in other apps or contexts**, reducing duplication and effort.

### 2. Real-world Analogy
A reusable recipe base sauce can be used in many dishes; you don’t reinvent it each time.

### 3. Concrete Example (Software – Food Delivery App)
A shared “address validation + geocoding” module reused across customer checkout, restaurant onboarding, and shipper navigation.

### 4. Bad vs Good Comparison
- Low: Three separate implementations of “money rounding” in different modules.
- High: Shared library with a single tested implementation.

### 5. How to Design for It (Practical)
- Build shared libraries (validation, money math, status machine helpers).
- Define reusable UI components in Next.js (tables, forms, status badges).
- Publish stable internal APIs (e.g., notification service).
- Keep modules decoupled and well-documented.

### 6. How to Measure It
- Code duplication < 5% (measured by tooling).
- % of new features using shared components ≥ 60%.
- Defect rate in reused module decreases over time.

### 7. Common Mistakes
- Forcing reuse too early; premature shared libraries become constraints.
- Copy-paste “reuse” (duplicates) without ownership.

---

## 14) Usability (Dễ dùng)

### 1. Simple Explanation
Usability means the system is **convenient and easy to use**, including UI clarity and access for different users.

### 2. Real-world Analogy
A GPS is usable if it gives clear directions and reroutes smoothly. It’s not usable if it’s confusing and hard to read.

### 3. Concrete Example (Software – Food Delivery App)
Checkout should be simple: clear fees, delivery time, payment method, and an obvious “Place Order” action.

### 4. Bad vs Good Comparison
- Low: Users can’t find “track order”; confusing error messages.
- High: Few steps to place order; clear states; accessible UI.

### 5. How to Design for It (Practical)
- Use consistent UI patterns (Next.js components, clear navigation).
- Provide helpful error messages and recovery paths.
- Optimize for mobile interactions (one-hand usage, large tap targets).
- Support accessibility basics (contrast, keyboard navigation where relevant).

### 6. How to Measure It
- Task success rate: ≥ 95% users can place an order without help.
- Checkout drop-off rate reduced below a target (e.g., < 20%).
- Support tickets per 1,000 orders for “can’t place order” < 2.

### 7. Common Mistakes
- Treating usability as “pretty UI” only; it’s about **successful tasks**.
- Optimizing only for one role (customer) and ignoring restaurant/shipper/admin workflows.

---

# 🔥 Architecture Application (Very Important)

## The 3 Most Important Quality Attributes for a Food Delivery System

### 1) Availability
**Why it’s critical (business impact):** If the app is down, customers can’t order, restaurants lose revenue immediately, and shippers can’t complete deliveries. Outages during meal peaks cause direct revenue loss and long-term trust damage.

### 2) Reliability
**Why it’s critical (real-world behavior):** Food delivery is an end-to-end flow: create order → accept → prepare → pickup → deliver → pay/report. If the system is unreliable (duplicate orders, wrong states, missing notifications), you get refunds, disputes, and operational chaos.

### 3) Security
**Why it’s critical (risk & governance):** The system stores important data and exposes privileged actions (admin approvals, suspensions, cancellations). Weak security enables fraud, data leaks, and marketplace manipulation.

---

## How These 3 Attributes Shape Your Architecture
Target stack:
- Frontend: Next.js
- Backend: NestJS
- Database: PostgreSQL
- Communication: REST + WebSocket

---

## A) Availability → Architecture Decisions

### Specific decisions (by layer)
- Next.js:
  - Serve static assets via CDN; cache public pages where safe.
  - Add client-side fallbacks (show last-known restaurant list if search temporarily fails).
- NestJS:
  - Run multiple instances behind a load balancer; health checks and auto-restart.
  - Graceful degradation: if “recommendations” fails, keep ordering endpoints working.
- PostgreSQL:
  - Automated backups + tested restore procedure.
  - Connection pooling to avoid connection exhaustion during spikes.
- REST + WebSocket:
  - For WebSocket, handle reconnects and resume state (client can re-subscribe to order updates).

### Trade-offs
- Gain: fewer outages and better peak-time stability.
- Sacrifice: more infrastructure complexity (monitoring, deployment, multi-instance coordination).

### Concrete example
During dinner peak, one NestJS instance crashes. Load balancer routes traffic to remaining instances; customers still place orders, and tracking continues after reconnection.

### SRS-quality requirement examples
- Core ordering APIs uptime ≥ 99.9% monthly.
- Automated failover/restart restores service within 2 minutes after instance failure.

---

## B) Reliability → Architecture Decisions

### Specific decisions (by layer)
- Next.js:
  - Show clear order states that come from a single backend source of truth.
  - Prevent double-submit on checkout (disable button after click; show in-progress state).
- NestJS:
  - Implement order lifecycle as a strict state machine (valid transitions only).
  - Use idempotency keys for `POST /orders` and payment confirmation.
  - Use transactional writes for multi-step operations (create order + line items + initial status).
- PostgreSQL:
  - Enforce constraints (foreign keys, unique constraints for idempotency keys).
  - Store status history as an append-only table to preserve traceability.
- REST + WebSocket:
  - WebSocket events derived from committed DB state (don’t emit “delivered” before DB commit).

### Trade-offs
- Gain: fewer duplicates, fewer disputes, predictable order flow.
- Sacrifice: extra implementation work (transactions, outbox, careful retries).

### Concrete example
Customer taps “Place Order” twice because network is slow. With idempotency, backend creates only one order, and the client receives one consistent confirmation.

### SRS-quality requirement examples
- Duplicate order creation rate < 0.001%.
- Invalid order state transitions = 0 (rejected with explicit error).

---

## C) Security → Architecture Decisions

### Specific decisions (by layer)
- Next.js:
  - Do not trust UI for authorization; hide admin features but assume attackers can call APIs.
  - Store tokens safely; avoid leaking secrets in logs.
- NestJS:
  - Enforce authentication + authorization guards on every endpoint.
  - RBAC for admin actions (approve/reject, suspend, cancel order, change commission).
  - Audit log for privileged actions (actor, action, target, timestamp, before/after).
  - Rate limit login and sensitive endpoints; lockouts after repeated failures.
- PostgreSQL:
  - Least-privilege DB users (app vs admin maintenance).
  - Encrypt sensitive fields if required; always TLS in transit.
- REST + WebSocket:
  - Authenticate WebSocket connections; authorize channel subscriptions (only watch your own order).

### Trade-offs
- Gain: reduced fraud/data leak risk; stronger governance.
- Sacrifice: more friction for developers (permissions, audits) and sometimes slightly higher latency.

### Concrete example
A malicious user tries to call an admin cancellation API directly. Server-side RBAC blocks it; the attempt is logged for investigation.

### SRS-quality requirement examples
- 100% admin endpoints require RBAC.
- All privileged actions must create an immutable audit entry, or the action is blocked/retried per policy.

---

## Quick “How to Write SRS-Quality QA Requirements” (Pattern)
For each attribute, write measurable scenarios:
- **Stimulus** (what happens) → **Response** (what the system does) → **Response measure** (numbers/time/percent).
Example (Performance): “Given a typical search query, when a customer searches restaurants, then results are returned within p95 ≤ 2 seconds.”
