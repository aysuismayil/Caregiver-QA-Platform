# Database Validation (SQL)

I had **read-only** access to a staging PostgreSQL database (through DBeaver). I used it to confirm that actions in the UI or API really saved the correct data, not just a correct-looking screen. This catches bugs where the screen shows success but the data is wrong, or the other way around.

> **Read-only:** I only ran `SELECT` queries. I checked data, I never changed it.

> **Illustrative schema:** the tables and columns below are a simple example made for this portfolio. They do not show the real database structure.

---

## Example schema

```sql
-- customers and providers
users (id, email, role, created_at)

-- provider profile data
provider_profiles (id, user_id, hourly_rate, lat, lng, is_active, created_at, updated_at)

-- one row per booking
bookings (
  id, customer_id, provider_id, slot_start, slot_end,
  status,  -- 'held' | 'pending_provider_response' | 'confirmed' | 'declined' | 'cancelled_by_customer' | 'completed'
  created_at, updated_at
)

-- one row per charge attempt
payments (id, booking_id, amount, status, idempotency_key, created_at)

-- refunds linked to a payment
refunds (id, payment_id, amount, created_at)

-- temporary holds during checkout
slot_holds (id, booking_id, slot_id, expires_at, created_at)
```

---

## Query 1: Overlapping confirmed bookings for the same provider

Used to confirm [BUG-002](../bug-reports/BUG-002-double-booking-race-condition.md) at the data level, not only through API responses.

```sql
SELECT
  b1.id AS booking_id_1,
  b2.id AS booking_id_2,
  b1.provider_id,
  b1.slot_start,
  b1.slot_end
FROM bookings b1
JOIN bookings b2
  ON  b1.provider_id = b2.provider_id
  AND b1.id < b2.id
  AND b1.status = 'confirmed'
  AND b2.status = 'confirmed'
  AND b1.slot_start < b2.slot_end
  AND b2.slot_start < b1.slot_end
ORDER BY b1.slot_start;
```

**What I check:** this should always return **zero rows**. Any row means two confirmed bookings overlap for the same provider: a double booking.

---

## Query 2: Duplicate successful payments for one booking

Used to confirm [BUG-003](../bug-reports/BUG-003-payment-double-charge-retry.md) and [BUG-006](../bug-reports/BUG-006-mobile-duplicate-charge-on-reconnect.md).

```sql
SELECT
  booking_id,
  COUNT(*)              AS payment_count,
  SUM(amount)           AS total_charged,
  ARRAY_AGG(id)         AS payment_ids,
  ARRAY_AGG(idempotency_key) AS idempotency_keys
FROM payments
WHERE status = 'succeeded'
GROUP BY booking_id
HAVING COUNT(*) > 1
ORDER BY total_charged DESC;
```

**What I check:** a booking with more than one successful payment is a duplicate charge. Different or empty idempotency keys on the duplicates point to the root cause (missing or unused key).

---

## Query 3: Amount charged for a booking (compare with UI and API)

Used in [BUG-005](../bug-reports/BUG-005-price-mismatch-frontend-backend.md) to confirm the real charged amount.

```sql
SELECT
  p.id          AS payment_id,
  p.booking_id,
  p.amount      AS amount_charged,
  b.slot_start,
  b.slot_end,
  pp.hourly_rate AS provider_rate_now
FROM payments p
JOIN bookings b           ON b.id = p.booking_id
JOIN provider_profiles pp ON pp.id = b.provider_id
WHERE p.booking_id = :booking_id;
```

**What I check:** `amount_charged` against the price shown at checkout (from DevTools/screenshot) and the booking details from the API. All three must match.

---

## Query 4: Refund for a cancelled booking matches the policy

```sql
SELECT
  b.id         AS booking_id,
  b.status,
  p.amount     AS original_amount,
  r.amount     AS refund_amount,
  ROUND((r.amount / NULLIF(p.amount, 0)) * 100, 1) AS refund_percentage,
  b.updated_at AS cancelled_at,
  b.slot_start
FROM bookings b
JOIN payments p ON p.booking_id = b.id AND p.status = 'succeeded'
LEFT JOIN refunds r ON r.payment_id = p.id
WHERE b.status = 'cancelled_by_customer'
ORDER BY b.updated_at DESC
LIMIT 20;
```

**What I check:** cancellations inside the free window (example policy: 48 hours or more before `slot_start`) must show 100% refund. Late cancellations must match the late fee (example: 50%). Anything else is flagged for follow-up with [TC-PAY-006 / TC-PAY-007](../test-cases/05-payments-test-cases.md).

---

## Query 5: Check search distance against stored coordinates

Used with [BUG-001](../bug-reports/BUG-001-search-radius-boundary-excluded.md) to calculate the distance independently, so I don't rely on the same code that has the bug.

```sql
SELECT
  pp.id AS provider_id,
  pp.is_active,
  ROUND(
    3958.8 * ACOS(
      COS(RADIANS(:origin_lat)) * COS(RADIANS(pp.lat)) *
      COS(RADIANS(pp.lng) - RADIANS(:origin_lng)) +
      SIN(RADIANS(:origin_lat)) * SIN(RADIANS(pp.lat))
    ), 2
  ) AS distance_miles
FROM provider_profiles pp
WHERE pp.is_active = true
ORDER BY distance_miles ASC;
```

**What I check:** this straight-line distance is my independent reference. Comparing it with what the search API returns for a radius confirmed that the boundary bug was real and not a mistake in my seed data.

---

## Why this matters

Testing only through the UI means trusting that the UI, API and database all agree. Bugs often hide between these layers (like BUG-005, where the UI and the API/database disagreed). My own SQL checks look at the actual saved data.
