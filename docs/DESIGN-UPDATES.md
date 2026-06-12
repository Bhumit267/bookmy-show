# Design Updates

## Update 1: Seat Hold Limit

**Triggered By:**
Panel Question 2

**What Changed:**

Added Redis key:

holds:{userId}:count

Maximum active seat holds = 8

**Why This Is Necessary:**

Prevents users or bots from reserving hundreds of seats and blocking inventory.

**What It Costs:**

One additional Redis lookup.

**What It Still Doesn't Solve:**

Users can still create multiple accounts.

---

## Update 2: Queue Depth Monitoring

**Triggered By:**
Panel Question 4

**What Changed:**

Added CloudWatch alarm.

Condition:

SQS Queue Depth > 10,000

Action:

Notify operations team.

**Why This Is Necessary:**

Detects payment backlog before user impact becomes severe.

**What It Costs:**

Minor monitoring cost.

**What It Still Doesn't Solve:**

Manual intervention may still be required.

---

## Update 3: Dynamic Seat Hold TTL

**Triggered By:**
Panel Question 1

**What Changed:**

Normal Events:
Seat Hold TTL = 5 minutes

High Demand Events:
Seat Hold TTL = 3 minutes

**Why This Is Necessary:**

Reduces inventory lock duration during flash sales.

**What It Costs:**

Users have less time to complete payment.

**What It Still Doesn't Solve:**

Cannot prevent all abandoned carts.
