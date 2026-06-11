# ShowTime Ticketing System

## Constraints

### Constraint 1: 5 Lakh Concurrent Users

Expected users: 500,000

Assuming each user performs:
- View event
- View seats
- Select seats
- Create booking

Total API calls:
500,000 × 4 = 2,000,000

Peak RPS:
2,000,000 / 60 = 33,333 RPS

The database becomes the first bottleneck if every request hits PostgreSQL directly.

---

### Constraint 2: Zero Double Bookings

A double booking means:

Two booking records successfully reserve the same seat.

Example:

Seat A12 booked by User X
Seat A12 booked by User Y

This must never happen.

Protection mechanisms:

- Database row locking
- Unique constraints
- Redis seat locks
- Idempotent booking flow

---

### Constraint 3: $2,000 AWS Budget

Budget allows roughly:

- Application Servers
- PostgreSQL RDS
- Redis Cluster
- SQS
- ALB

Cannot afford:

- Large Aurora clusters
- Multi-region deployments
- Dedicated Kafka clusters

Therefore architecture must remain simple and cost-efficient.