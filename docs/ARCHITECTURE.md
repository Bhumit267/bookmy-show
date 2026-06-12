# ShowTime System Architecture

```text
                            ┌─────────────────────┐
                            │       Users         │
                            │ 500,000 concurrent  │
                            └──────────┬──────────┘
                                       │
                                       ▼
                            ┌─────────────────────┐
                            │   CloudFront CDN    │
                            │ Cache static assets │
                            │ Cache event pages   │
                            │ TTL-based caching   │
                            └───────┬─────┬───────┘
                                    │Hit  │Miss/API
                                    │     │
                                    ▼     ▼
                           ┌──────────────────────┐
                           │ Application Load     │
                           │ Balancer (ALB)       │
                           │ SSL Termination      │
                           │ Health Check: 30s    │
                           │ Rate Limiting        │
                           └──────────┬───────────┘
                                      │
                                      ▼
                 ┌────────────────────────────────────┐
                 │ Node.js API Servers (Auto Scaling) │
                 └──────┬────────┬──────────┬─────────┘
                        │        │          │
                        │        │          │
                        ▼        ▼          ▼

          ┌────────────────┐  ┌──────────────┐
          │ Redis Cluster  │  │ PostgreSQL   │
          │                │  │ Primary DB   │
          │ Cache Keys     │  │ Write Ops    │
          │ Seat Locks     │  │ Bookings     │
          │ SETNX + TTL    │  │ Seat Updates │
          └──────┬─────────┘  └──────┬───────┘
                 │                   │
                 │                   │
                 ▼                   ▼

          ┌────────────────┐  ┌──────────────┐
          │ Hold Counter   │  │ Read Replica │
          │ Max 8 Seats    │  │ Read Queries │
          └────────────────┘  └──────────────┘

                                      │
                                      ▼

                             ┌────────────────┐
                             │   SQS Queue    │
                             │ Visibility:60s │
                             │ DLQ Enabled    │
                             └──────┬─────────┘
                                    │
                                    ▼

                           ┌───────────────────┐
                           │ Payment Worker    │
                           │ Read Message      │
                           │ Process Payment   │
                           │ Update Booking    │
                           │ Update Seats      │
                           └───────┬───────────┘
                                   │
                                   ▼

                         ┌─────────────────────┐
                         │ Payment Gateway     │
                         └─────────┬───────────┘
                                   │
                                   ▼

                             ┌───────────┐
                             │    SNS    │
                             └─────┬─────┘
                                   │
                     ┌─────────────┴─────────────┐
                     ▼                           ▼
              ┌────────────┐             ┌────────────┐
              │ SES Email  │             │ SMS Service│
              └────────────┘             └────────────┘

Monitoring:
CloudWatch Alarm
Trigger: SQS Depth > 10,000
```

## Part A References

* Redis SETNX locking → CONCURRENCY.md
* Async SQS payment processing → QUEUE.md
* Cache invalidation strategy → CACHE.md
* PostgreSQL schema and indexes → SCHEMA.md
* Read replicas added to separate reads from writes
* Seat hold counter added after panel feedback
