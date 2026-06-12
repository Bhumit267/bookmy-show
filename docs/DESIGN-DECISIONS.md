# Design Decisions

## Decision: Concurrency Strategy

**Context:**
Zero double-bookings allowed.

**Options Considered:**

1. PostgreSQL SELECT FOR UPDATE

   * Strong consistency
   * Pool exhaustion risk

2. Redis SETNX + PostgreSQL (Chosen)

   * Faster locking
   * Better scalability

**Why Chosen:**
Redis can handle far higher request volume while PostgreSQL remains source of truth.

**Tradeoffs Accepted:**
Redis failure can temporarily affect locking.

**Revision Trigger:**
Move to PostgreSQL-only locking if traffic becomes much smaller.

---

## Decision: Cache Invalidation

**Context:**
Seat availability changes frequently.

**Options Considered:**

1. TTL-only cache

   * Simple
   * Can become stale

2. Event-driven invalidation (Chosen)

   * More accurate
   * Slightly more complex

**Why Chosen:**
Prevents users from seeing outdated seat counts.

**Tradeoffs Accepted:**
Extra cache management logic.

**Revision Trigger:**
If invalidation traffic becomes excessive.

---

## Decision: UUID vs SERIAL

**Context:**
Booking IDs must be globally unique.

**Options Considered:**

1. SERIAL

   * Smaller
   * Predictable

2. UUID (Chosen)

   * Globally unique
   * Harder to guess

**Why Chosen:**
Safer for distributed systems.

**Tradeoffs Accepted:**
Larger index size.

**Revision Trigger:**
Never likely to change.

---

## Decision: SQS Visibility Timeout

**Context:**
Payment processing can take time.

**Options Considered:**

1. 30 Seconds

   * Faster retries
   * Higher duplicate risk

2. 60 Seconds (Chosen)

   * Enough for payment processing

**Why Chosen:**
Allows payment completion before message reappears.

**Tradeoffs Accepted:**
Longer recovery time on worker failure.

**Revision Trigger:**
If average payment latency exceeds 60 seconds.
    