# Panel Q&A Notes

## Question 1

What happens if Redis crashes during seat locking?

Answer:
Locks disappear, but PostgreSQL remains source of truth. Booking confirmation still validates seat ownership.

Gap:
Need Redis HA configuration.

---

## Question 2

What stops a user from holding 200 seats?

Answer:
Original design had no protection.

Gap:
Added seat hold counter limiting users to 8 active holds.

---

## Question 3

At what point does PostgreSQL become bottleneck?

Answer:
Around 2800+ RPS based on connection pool calculations.

Gap:
Need sharding for future growth.

---

## Question 4

What if SQS queue grows too large?

Answer:
Workers auto-scale.

Gap:
Added CloudWatch alarm at queue depth > 10,000.

---

## Question 5

Why not use PostgreSQL FOR UPDATE only?

Answer:
Would create connection pool exhaustion under heavy load.

Gap:
Hybrid approach remains better for expected traffic.
