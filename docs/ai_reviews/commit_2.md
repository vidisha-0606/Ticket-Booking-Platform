# AI Review – Commit 2

## Commit Summary
Added planning documentation including system architecture, tech stack, API design, data models, and seat locking strategy.

---

## AI Assistance
Used Claude to generate an initial system design covering:
- Tech stack selection
- API structure
- Database schema
- Concurrency handling approach

---

## Decisions Taken
- Selected Node.js + Express for backend due to simplicity and speed
- Chose React (Vite) for frontend for fast development
- Used PostgreSQL for persistent storage
- Used Redis for implementing seat locking with TTL
- Preferred REST APIs over microservices to keep the system simple
- Chose polling instead of WebSockets for real-time updates

---

## What Was Accepted
- Overall architecture design
- API structure and endpoint breakdown
- Use of Redis SETNX for seat locking
- Suggested folder structure

---

## What Was Modified
- Simplified data models to match MVP scope
- Removed unnecessary complexity (e.g., authentication, advanced features)
- Reduced scope to fit 3-hour time constraint

---

## What Was Rejected
- Over-engineered components not required for MVP
- Advanced scalability patterns (microservices, event queues)

---

## Reflection
The AI-generated design provided a strong starting point. It was refined to ensure simplicity, clarity, and feasibility within the given time constraints while focusing on the core challenge of concurrency handling.
