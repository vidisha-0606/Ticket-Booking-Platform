# Discovery Notes

## Problem Understanding
Build a ticket booking platform where users can:
- Browse events
- View seat availability on a seat map
- Select seats
- Book tickets and receive confirmation

The system must prevent multiple users from booking the same seat at the same time.

---

## Functional Requirements
- Display list of events (name, date, venue, price)
- Show seat map with seat states (available, held, booked)
- Allow users to select and hold seats temporarily
- Confirm booking and generate confirmation
- Release seats automatically if booking is not completed within a time limit

---

## Non-Functional Requirements
- No double booking (strong consistency required)
- Seat holds expire automatically (5–10 minutes)
- UI reflects near real-time seat availability
- Simple and responsive user experience

---

## Assumptions
- User authentication is not implemented (single test user)
- Payment is mocked (no real payment gateway)
- Events are pre-seeded in the database
- Seat layout is a simple grid (rows × columns)
- Single region with limited concurrent users (~100)
- Notifications are simulated (no real email service)
- Refunds and cancellations are out of scope
- Single pricing tier for all seats

---

## Core Challenge
Handling **concurrent seat selection**:
- Multiple users may attempt to book the same seat simultaneously
- System must ensure only one successful booking

### Chosen Approach
- Use **Redis-based seat locking (SETNX with TTL)**
- When a seat is selected:
  - Lock is created with expiry (e.g., 5 minutes)
- Only the lock owner can confirm booking
- Lock automatically expires if not confirmed

---

## MVP Scope (3-Hour Target)

### Backend APIs
- `GET /events` → list events
- `GET /events/:id/seats` → fetch seat map
- `POST /seats/:id/hold` → hold a seat
- `DELETE /seats/:id/hold` → release a seat
- `POST /bookings` → confirm booking

### Frontend
- Event listing page
- Seat map UI (grid-based)
- Seat selection and hold interaction
- Booking confirmation flow
- Confirmation screen

### Data Storage
- PostgreSQL → events, seats, bookings
- Redis → seat locks with TTL

---

## Out of Scope / Simplifications
- Real payment integration (mocked)
- Authentication system
- Admin panel
- Complex seat layouts
- Multi-seat selection (single seat only)
- Real-time updates (use polling instead of WebSockets)
- Email service (console-based simulation)
- Mobile responsiveness
- Automated testing

---

## Build Priority
1. Database schema setup
2. Seat locking mechanism (core logic)
3. Seat map API and UI
4. Booking confirmation flow
5. Seat hold expiry handling
