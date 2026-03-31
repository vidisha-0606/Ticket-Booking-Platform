# Ticket-Booking-Platform

A minimal ticket booking system with seat selection and concurrency-safe seat locking.

**Stack:** Node.js + Express · React + Vite · SQLite (via better-sqlite3)  
**No Docker, no Redis, no external services required.**

---

## Project structure

```
ticketapp/
├── backend/
│   ├── src/
│   │   ├── app.js                  ← Express entry point
│   │   ├── routes/
│   │   │   ├── auth.js             ← POST /api/auth/login
│   │   │   ├── events.js           ← GET /api/events, /api/events/:id/seats
│   │   │   ├── seats.js            ← POST/DELETE /api/seats/:id/hold
│   │   │   └── bookings.js         ← POST /api/bookings, GET /api/bookings/:id
│   │   ├── services/
│   │   │   └── lockService.js      ← Seat locking (SETNX-style, SQLite-backed)
│   │   ├── middleware/
│   │   │   └── auth.js             ← JWT verification
│   │   └── db/
│   │       ├── client.js           ← SQLite setup + schema
│   │       └── seed.js             ← Seed events and seats
│   └── package.json
│
├── frontend/
│   ├── src/
│   │   ├── main.jsx                ← App entry + routing
│   │   ├── index.css               ← Global styles
│   │   ├── api/client.js           ← Fetch wrapper
│   │   ├── pages/
│   │   │   ├── LoginPage.jsx
│   │   │   ├── EventsPage.jsx
│   │   │   ├── SeatMapPage.jsx
│   │   │   ├── CheckoutPage.jsx
│   │   │   └── ConfirmationPage.jsx
│   │   └── components/
│   │       ├── SeatGrid.jsx        ← Visual seat map
│   │       └── CountdownTimer.jsx  ← Lock countdown
│   ├── index.html
│   └── package.json
│
└── README.md
```

---

## Setup (3 steps)

### 1. Install dependencies

```bash
# Backend
cd ticketapp/backend
npm install

# Frontend
cd ../frontend
npm install
```

### 2. Seed the database

```bash
cd ticketapp/backend
npm run seed
```

This creates `ticketapp.db` with 3 events and 50 seats each (~20% pre-booked for realism).

### 3. Run both servers

Open **two terminals**:

```bash
# Terminal 1 — backend (port 3001)
cd ticketapp/backend
npm run dev

# Terminal 2 — frontend (port 5173)
cd ticketapp/frontend
npm run dev
```

Open **http://localhost:5173** in your browser.

---

## How to use

1. **Log in** — enter any email address (no password needed in MVP)
2. **Browse events** — three seeded events are listed
3. **Select a seat** — click any white seat on the grid. It's immediately locked for 5 minutes.
4. **Checkout** — review your order and confirm
5. **Confirmation** — booking is saved, seat is marked permanently booked

---

## API reference

| Method | Path | Auth | Description |
|--------|------|------|-------------|
| POST | `/api/auth/login` | No | Get JWT token |
| GET | `/api/events` | No | List events |
| GET | `/api/events/:id` | No | Single event |
| GET | `/api/events/:id/seats` | No | Seat map with live status |
| POST | `/api/seats/:id/hold` | Yes | Lock a seat (5 min TTL) |
| DELETE | `/api/seats/:id/hold` | Yes | Release a lock |
| POST | `/api/bookings` | Yes | Confirm booking |
| GET | `/api/bookings/:id` | Yes | Get confirmation |

---

## Seat lock strategy

Instead of Redis, seat locks are stored in a `seat_locks` SQLite table with a Unix timestamp expiry.

```
seat_locks table:
  seat_id    TEXT PRIMARY KEY
  user_id    TEXT
  expires_at INTEGER  ← Unix timestamp (now + 300s)
```

**Acquire:** `INSERT ... WHERE NOT EXISTS active lock` — atomic, prevents double-holds.  
**Validate:** Check `seat_id + user_id + expires_at > now` before confirming.  
**Release:** On confirm, expiry, or user navigating away.  
**Auto-expiry:** Stale locks are cleaned up on every new hold request.

To swap in Redis later, replace `lockService.js` with Redis `SETNX` calls — the interface is identical.

---

## Extending beyond MVP

| Feature | What to add |
|---------|-------------|
| Real payments | Stripe `PaymentIntent` in `POST /bookings` |
| Email confirmation | Nodemailer in bookings route after DB write |
| Real auth | `users` table + bcrypt password check |
| WebSocket seat updates | `ws` package, broadcast on hold/release |
| Multiple seats | Pass `seatIds[]` to hold + booking endpoints |
| Redis locking | Swap `lockService.js` for `ioredis` SETNX calls |
