# Async Order Processing

## Why Async?

Synchronous payment calls hold database connections.

Formula:

Connections Held =
(RPS × 80% × 0.02)
+
(RPS × 20% × 0.8)

Pool exhaustion occurs around 2840 RPS.

Async processing removes payment wait time from API servers.

---

## SQS Message Format

```json
{
  "bookingId":"uuid",
  "userId":"uuid",
  "totalAmount":2500.00,
  "paymentToken":"token",
  "seatIds":[101,102],
  "eventId":10,
  "idempotencyKey":"booking_uuid"
}
```

### Fields

bookingId:
Unique booking reference.

userId:
Booking owner.

totalAmount:
Payment amount.

paymentToken:
Gateway authorization token.

seatIds:
Seats reserved.

eventId:
Event identifier.

idempotencyKey:
Prevents duplicate charging.

---

## Worker Logic

Success Flow

1. Receive SQS message
2. Verify booking
3. Process payment
4. Update booking status = confirmed
5. Update seat status = booked
6. Send confirmation email
7. Delete SQS message

Failure Flow

1. Receive message
2. Payment fails
3. Update booking = failed
4. Release seats
5. Delete message

DLQ Flow

1. Retry multiple times
2. Move to DLQ
3. Alert operations team

---

## Edge Cases

### Server Crashes After SQS Publish

Booking remains pending.

Worker processes message.

User may see delayed confirmation.

No booking lost.

---

### Payment Gateway Timeout

Worker retries.

Use idempotency key.

Maximum retries before DLQ.

---

## SQS Configuration

Visibility Timeout

60 seconds

Reason:

Longer than payment processing duration.

Max Receive Count

5

Reason:

Transient failures often recover within 5 retries.

After that send to DLQ.