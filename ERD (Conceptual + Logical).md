# ERD — Airline Reservation System (Single Airline: )

## 1) Conceptual ERD (Chen)

### 1.1 Entities
- Customer
- Booking (PNR)
- Passenger
- Booking_Passenger (junction)
- Airport
- Route
- Flight
- Flight_Instance (scheduled occurrence)
- Aircraft
- Seat
- Fare_Class
- Ticket
- Payment
- Check_In
- Boarding_Pass
- Refund

### 1.2 Relationships & Cardinalities
- Customer **(1)** —creates— **(M)** Booking
- Booking **(1)** —includes— **(M)** Booking_Passenger —links— **(M)** Passenger
- Airport **(1)** —origin_of— **(M)** Route
- Airport **(1)** —destination_of— **(M)** Route
- Route **(1)** —has— **(M)** Flight
- Flight **(1)** —schedules— **(M)** Flight_Instance
- Aircraft **(1)** —assigned_to— **(M)** Flight_Instance
- Aircraft **(1)** —contains— **(M)** Seat
- Flight_Instance **(1)** —issues— **(M)** Ticket
- Passenger **(1)** —holds— **(M)** Ticket
- Fare_Class **(1)** —applies_to— **(M)** Ticket
- Booking **(1)** —has— **(M)** Payment
- Ticket **(0..1)** —checks_in_as— **(1)** Check_In
- Check_In **(1)** —generates— **(1)** Boarding_Pass
- Ticket **(0..1)** —assigned_to— **(1)** Seat  
  *(Seat assignment is stored via `ticket.seat_id` and can happen during booking or later at check-in.)*
- Booking **(0..M)** —results_in— **(0..M)** Refund  
  *(Supports partial/multiple refunds.)*

---

## 2) Logical ERD (Keys + Attributes)

> Naming convention: `snake_case`  
> PK = Primary Key, FK = Foreign Key

### 2.1 customer
- **customer_id (PK)**
- full_name
- email *(UNIQUE)*
- phone
- created_at

### 2.2 booking (PNR)
- **booking_id (PK)**
- pnr_code *(UNIQUE)*
- customer_id *(FK → customer.customer_id)*
- trip_type *(ONE_WAY / ROUND_TRIP)*
- status *(e.g., CONFIRMED / CANCELLED)*
- booked_at

### 2.3 passenger
- **passenger_id (PK)**
- first_name
- last_name
- date_of_birth
- passport_no
- nationality

### 2.4 booking_passenger (junction)
- **booking_id (PK, FK → booking.booking_id)**
- **passenger_id (PK, FK → passenger.passenger_id)**
- passenger_role *(ADULT / CHILD / INFANT)*

### 2.5 airport
- **airport_code (PK)** *(IATA code)*
- name
- city
- country

### 2.6 route
- **route_id (PK)**
- origin_code *(FK → airport.airport_code)*
- dest_code *(FK → airport.airport_code)*
- UNIQUE(origin_code, dest_code)
- CHECK(origin_code <> dest_code)

### 2.7 aircraft
- **aircraft_id (PK)**
- model
- tail_number *(UNIQUE)*
- status

### 2.8 seat
- **seat_id (PK)**
- aircraft_id *(FK → aircraft.aircraft_id)*
- seat_no
- cabin_class
- UNIQUE(aircraft_id, seat_no)

### 2.9 flight
- **flight_id (PK)**
- route_id *(FK → route.route_id)*
- flight_number *(UNIQUE)*

### 2.10 flight_instance (scheduled flight)
- **flight_instance_id (PK)**
- flight_id *(FK → flight.flight_id)*
- aircraft_id *(FK → aircraft.aircraft_id)*
- depart_at
- arrive_at
- status *(SCHEDULED / DELAYED / DEPARTED / CANCELLED)*
- UNIQUE(flight_id, depart_at)
- CHECK(arrive_at > depart_at)

### 2.11 fare_class
- **fare_class_id (PK)**
- code *(UNIQUE; e.g., Y/J)*
- name *(Economy/Business)*
- rules *(text)*

### 2.12 ticket
- **ticket_id (PK)**
- ticket_number *(UNIQUE)*
- booking_id *(FK → booking.booking_id)*
- passenger_id *(FK → passenger.passenger_id)*
- flight_instance_id *(FK → flight_instance.flight_instance_id)*
- fare_class_id *(FK → fare_class.fare_class_id)*
- seat_id *(FK → seat.seat_id, NULL allowed)*
- price_amount
- currency
- status *(ISSUED / CANCELLED)*
- issued_at
- UNIQUE(booking_id, passenger_id, flight_instance_id)

**Business integrity rule (enforced by app logic and/or DB trigger):**
- `ticket.seat_id` must belong to the aircraft used by the ticket’s `flight_instance`.

### 2.13 payment
- **payment_id (PK)**
- booking_id *(FK → booking.booking_id)*
- method *(CARD / WALLET)*
- provider *(optional)*
- amount
- currency
- status *(PAID / FAILED / REFUNDED / PARTIAL)*
- paid_at

### 2.14 check_in
- **check_in_id (PK)**
- ticket_id *(FK → ticket.ticket_id, UNIQUE)*  *(one check-in per ticket)*
- checked_in_at
- channel *(WEB / KIOSK / AGENT)*

### 2.15 boarding_pass
- **boarding_pass_id (PK)**
- check_in_id *(FK → check_in.check_in_id, UNIQUE)*
- bp_number *(UNIQUE)*
- gate
- boarding_time
- issued_at

### 2.16 refund
- **refund_id (PK)**
- booking_id *(FK → booking.booking_id)*
- payment_id *(FK → payment.payment_id, NULL allowed)*
- refund_amount
- currency
- reason
- status *(PENDING / APPROVED / REJECTED / COMPLETED)*
- requested_at

---

## 3) Requirements Covered (Confirmed)
- Supports one-way + round-trip bookings
- Seat assignment during booking or later during check-in
- Payment methods: Card + Wallet
- Multiple passengers per booking (PNR)
- Cancellation + refunds (partial/full)
- Check-in and boarding pass stored in the database
