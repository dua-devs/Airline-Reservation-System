# Airline Reservation System (Single Airline) — ERD Notes & Data Model Walkthrough

This document explains how the database model works for a **single-airline reservation system (Darrbak)**.
It provides a clear walkthrough of the booking flow and how it maps to the ERD.

---

## 1) Overview (What a PNR / Booking contains)

A **Booking (PNR)** is the main reservation record. One PNR can include:
- Multiple **passengers**
- One-way or round-trip (multiple **flight legs**)
- **Tickets** (one per passenger per scheduled leg)
- **Payments** (Card/Wallet)
- Optional **seat assignment** (during booking or later at check-in)
- **Check-in + boarding pass**
- **Cancellation + refunds** (partial or full)

---

## 2) Core Concept: Ticket is the “center” of the model

A **Ticket** represents:

> **One passenger** on **one scheduled flight occurrence** (`flight_instance`).

So if a booking has:
- 2 passengers  
- 2 flight legs (outbound + return)  
Then total tickets = `2 × 2 = 4`.

---

## 3) End-to-End Scenario Walkthrough

### Step 1 — Create a Booking (PNR)
A customer creates a booking:
- `trip_type = ROUND_TRIP`
- system generates `pnr_code = AB12CD`

**Relationship**
- `customer (1) → (M) booking`

---

### Step 2 — Add passengers under the same PNR
Passengers are stored in `passenger`, then linked to the booking using a junction table:

- `booking_passenger (booking_id, passenger_id)`

**Why a junction table?**
Bookings and passengers form a **many-to-many** relationship:
- one booking can contain many passengers
- the same passenger can appear in many bookings

So we resolve it with `booking_passenger`.

---

### Step 3 — Understand `flight` vs `flight_instance` (Very important)
- `flight` = the flight number/definition (e.g., DB101 MCT → DXB)
- `flight_instance` = the scheduled occurrence (same flight number but specific date/time)

**Relationships**
- `route (1) → (M) flight`
- `flight (1) → (M) flight_instance`
- `aircraft (1) → (M) flight_instance`

---

### Step 4 — Issue tickets (Passenger × Flight_Instance)
For each passenger and each flight_instance included in the booking, the system creates a ticket:
- `ticket.booking_id`
- `ticket.passenger_id`
- `ticket.flight_instance_id`
- `ticket.fare_class_id`

**Relationships**
- `passenger (1) → (M) ticket`
- `flight_instance (1) → (M) ticket`
- `fare_class (1) → (M) ticket`

---

### Step 5 — Seat assignment (During booking OR at check-in)
Seat selection is stored on the ticket:
- `ticket.seat_id` (nullable)

Rules:
- seat assigned during booking → `seat_id` is set early
- seat assigned at check-in → `seat_id` remains NULL until check-in

**Business rule**
The selected seat must belong to the aircraft used by the ticket’s `flight_instance`.
This is typically enforced via application logic and/or a DB trigger/constraint.

---

### Step 6 — Payments (Card / Wallet)
Payments are recorded per booking:
- `payment.booking_id`
- `payment.method IN (CARD, WALLET)`

A booking can have multiple payment records (retries/partials/adjustments):
- `booking (1) → (M) payment`

---

### Step 7 — Check-in + Boarding Pass
Each ticket can be checked in at most once:
- `check_in.ticket_id` is UNIQUE

A boarding pass is generated after check-in:
- `boarding_pass.check_in_id` is UNIQUE

**Relationships**
- `ticket (0..1) → (1) check_in`
- `check_in (1) → (1) boarding_pass`

---

### Step 8 — Cancellation + Refunds
Refunds are stored per booking (supports partial or full refunds):
- `refund.booking_id`
- optionally linked to `payment_id`

A booking can have multiple refund records:
- `booking (0..M) → (0..M) refund`

---

## 4) Quick Glossary

- **PNR / Booking**: Reservation “file”
- **Passenger**: Traveler
- **Flight**: Flight number definition
- **Flight_Instance**: Scheduled flight occurrence (date/time)
- **Ticket**: Passenger on a specific flight_instance
- **Seat**: Assigned via ticket (nullable)
- **Check-in / Boarding pass**: Per ticket
- **Payment / Refund**: Per booking

---

## 5) Requirements covered (Confirmed)

- Supports **one-way + round-trip**
- Seat assignment supports **booking or check-in**
- Payments: **Card + Wallet** (no cash)
- Supports **multiple passengers per booking (PNR)**
- Supports **cancellation + refunds**
- Stores **check-in and boarding pass** in the database
