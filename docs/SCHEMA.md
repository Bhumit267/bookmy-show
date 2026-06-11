# PostgreSQL Schema

```sql
CREATE TABLE venues (
    id BIGSERIAL PRIMARY KEY,
    name VARCHAR(255) NOT NULL,
    city VARCHAR(100) NOT NULL,
    capacity INT NOT NULL CHECK (capacity > 0)
);

CREATE INDEX idx_venues_city
ON venues(city);

CREATE TABLE events (
    id BIGSERIAL PRIMARY KEY,
    venue_id BIGINT NOT NULL REFERENCES venues(id) ON DELETE RESTRICT,
    name VARCHAR(255) NOT NULL,
    start_time TIMESTAMP NOT NULL,
    status VARCHAR(20) NOT NULL
    CHECK (status IN ('upcoming','on_sale','sold_out','cancelled')),
    total_seat_count INT NOT NULL CHECK (total_seat_count > 0)
);

CREATE INDEX idx_events_status
ON events(status);

CREATE TABLE users (
    id UUID PRIMARY KEY,
    email VARCHAR(255) UNIQUE NOT NULL,
    phone VARCHAR(20) UNIQUE NOT NULL,
    name VARCHAR(255) NOT NULL,
    created_at TIMESTAMP NOT NULL DEFAULT NOW()
);

CREATE INDEX idx_users_email
ON users(email);

CREATE TABLE seats (
    id BIGSERIAL PRIMARY KEY,
    event_id BIGINT NOT NULL REFERENCES events(id) ON DELETE CASCADE,
    section VARCHAR(50) NOT NULL,
    row_name VARCHAR(10) NOT NULL,
    seat_number VARCHAR(10) NOT NULL,
    price NUMERIC(10,2) NOT NULL CHECK(price > 0),
    category VARCHAR(20) NOT NULL
    CHECK(category IN ('VIP','Premium','General')),
    status VARCHAR(20) NOT NULL
    CHECK(status IN ('available','held','booked')),
    held_until TIMESTAMP,
    held_by UUID REFERENCES users(id) ON DELETE SET NULL,
    version INT NOT NULL DEFAULT 1
);

CREATE INDEX idx_seats_event_status
ON seats(event_id,status);

CREATE TABLE bookings (
    id UUID PRIMARY KEY,
    user_id UUID NOT NULL REFERENCES users(id) ON DELETE RESTRICT,
    event_id BIGINT NOT NULL REFERENCES events(id) ON DELETE RESTRICT,
    status VARCHAR(20) NOT NULL
    CHECK(status IN ('pending','confirmed','failed','refunded')),
    total_amount NUMERIC(10,2) NOT NULL CHECK(total_amount > 0),
    payment_reference VARCHAR(255),
    created_at TIMESTAMP NOT NULL DEFAULT NOW(),
    updated_at TIMESTAMP NOT NULL DEFAULT NOW()
);

CREATE INDEX idx_bookings_user
ON bookings(user_id,created_at DESC);

CREATE INDEX idx_bookings_active
ON bookings(status)
WHERE status IN ('pending');

CREATE TABLE booking_seats (
    booking_id UUID NOT NULL REFERENCES bookings(id) ON DELETE CASCADE,
    seat_id BIGINT NOT NULL REFERENCES seats(id) ON DELETE RESTRICT,
    PRIMARY KEY (booking_id,seat_id)
);

CREATE INDEX idx_booking_seats_seat
ON booking_seats(seat_id);
```

## Why UUID for booking.id?

UUIDs are globally unique and prevent predictable IDs.
Useful when multiple services generate bookings.

## Why version column?

Supports optimistic locking.
Detects concurrent updates.

## Why held_until?

Allows automatic seat release if payment is abandoned.

## Why partial index on bookings?

Most queries target pending bookings.
Smaller index = faster lookups.