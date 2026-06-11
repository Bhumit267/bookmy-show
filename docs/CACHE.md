# Cache Design

## Event Details

Key

event:{event_id}

TTL

300 seconds

Invalidation

- Event updated
- Event cancelled
- Event sold out

---

## Seat Availability Count

Key

availability:{event_id}:{category}

TTL

30 seconds

Invalidation

Any seat status change.

Example:

availability:10:VIP

---

## Seat Map Layout

Key

seatmap:{event_id}

TTL

24 hours

Invalidation

Venue configuration change.

---

## What We Will NOT Cache

Individual seat status.

Reason:

Can become stale within milliseconds.

Stale seat information can cause overselling attempts.

---

# Invalidation Strategy

Use Cache Aside Pattern.

Flow:

1. Update PostgreSQL
2. Delete Redis cache
3. Next read repopulates cache

Pseudo Flow:

When seat changes:

1. Update DB
2. Delete availability:{event}:{category}
3. Next request loads fresh data
4. Cache for 30 seconds