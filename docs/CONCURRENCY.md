# Concurrency Strategy

## Option A - PostgreSQL SELECT FOR UPDATE

Example:

```sql
BEGIN;

SELECT *
FROM seats
WHERE id = 101
FOR UPDATE;

UPDATE seats
SET status='held'
WHERE id=101;

COMMIT;
```

Prevents two transactions from modifying the same seat simultaneously.

---

### Pool Exhaustion Calculation

Formula:

Connections Held =
(RPS × 80% × 0.02)
+
(RPS × 20% × 0.8)

= 0.176 × RPS

Max Connections = 500

500 = 0.176 × RPS

RPS = 2840

Pool exhausted around 2840 RPS.

---

### Deadlock Risk

If:

User A locks Seat 1 then Seat 2

User B locks Seat 2 then Seat 1

Deadlock occurs.

Mitigation:

Always lock seats in ascending seat_id order.

---

## Option B - Redis SETNX

Lock Key:

seat_lock:{event_id}:{seat_id}

Example:

SET seat_lock:10:101 user123 NX EX 300

TTL = 300 seconds.

---

### Failure Scenario

If Redis crashes before booking confirmation:

Lock disappears.

Potential race condition exists.

Therefore Redis alone cannot be source of truth.

---

## Chosen Strategy

Hybrid Approach

Redis:
- High-speed temporary locking

PostgreSQL:
- Final source of truth

Reason:

- Handles 33k+ RPS better
- Fits within $2000 budget
- Prevents double booking

Limit:

Single PostgreSQL instance remains bottleneck.

Upgrade path:

Sharding events across databases.