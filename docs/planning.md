# Planning Notes

## Tech Stack
- Backend: Node.js + Express
- Frontend: React (Vite)
- Database: PostgreSQL
- Cache: Redis (for seat locking)

---

## System Architecture
- Frontend communicates with backend via REST APIs
- Backend handles business logic and seat locking
- PostgreSQL stores permanent data (events, seats, bookings)
- Redis manages temporary seat locks with expiry (TTL)

---

## Data Models

### Event
- id
- name
- date
- venue

### Seat
- id
- event_id
- row
- column
- status (available / held / booked)

### Booking
- id
- user_id
- event_id
- seat_id
- status (pending / confirmed)

---

## API Design

### Events
- `GET /events` → Fetch all events
- `GET /events/:id/seats` → Fetch seat map

### Seats
- `POST /seats/:id/hold` → Hold a seat (create lock)
- `DELETE /seats/:id/hold` → Release a seat

### Bookings
- `POST /bookings` → Confirm booking

---

## Seat Locking Strategy
- Use Redis `SETNX` with expiry (TTL)
- When user selects a seat:
  - Lock is created for that seat with user ID
- Only the user holding the lock can book the seat
- Lock automatically expires after a fixed time (e.g., 5 minutes)
- Prevents multiple users from booking the same seat

---

## Folder Structure
ticketapp/
├── backend/
│ ├── routes/
│ ├── controllers/
│ ├── services/
│ ├── db/
│ ├── middleware/
│ └── app.js
│
├── frontend/
│ ├── pages/
│ ├── components/
│ ├── api/
│ └── main.jsx
│
└── docs/
├── discovery.md
└── planning.md

---

## Development Plan

1. Setup backend and database
2. Implement event APIs
3. Build seat map API
4. Implement seat locking using Redis (core feature)
5. Create frontend (event list + seat map)
6. Implement booking flow
7. Add confirmation page

---

## Key Design Decisions
- Keep architecture simple for 3-hour constraint
- Focus on concurrency (seat locking) as priority
- Use polling instead of WebSockets for updates
- Avoid over-engineering (no microservices)
