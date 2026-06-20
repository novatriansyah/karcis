# Karcis (Event Ticketing Platform)

Karcis is a self-service event ticketing platform engineered for high-concurrency ticket sales ("ticket wars"), secure gate verification, and compliance with Indonesian electronic commerce regulations. 

---

## Project Vision & Architecture

The system is designed with a hybrid-split transaction ledger where platform service fees are routed to the company merchant account immediately, and event ticket prices are held in escrow for organizers. 

The codebase is organized as a monorepo structure separating the client application from backend services:

```
karcis/
├── backend/            # Go backend codebase (Clean Architecture)
├── frontend/           # Next.js frontend codebase (PWA & HeroUI)
├── PRD.md              # Detailed Product Requirements & Engineering Rules
└── README.md           # Project overview and workspace guide
```

---

## Tech Stack Summary

* **Backend API:** Golang (Standard Library + Chi Router)
* **Database & Cache:** PostgreSQL (ACID transactions) + Redis (atomic counters, TTL reservation locks)
* **Asynchronous Workers:** Go background workers + RabbitMQ/Redis Streams (for email/WhatsApp notification routing)
* **Frontend Web App:** Next.js (App Router, TS) + HeroUI (formerly NextUI) + Tailwind CSS + Framer Motion
* **Theme System:** Light Mode default with dynamic, semantic-token-driven Dark Mode switching (using `next-themes`)
* **Entry Gate Scanner:** Progressive Web App (PWA) incorporating HTML5 camera streaming, local LAN Redis caching, and Web Audio/Device Haptic validation feedback.

---

## Detailed Specifications

Refer to the complete Product Requirements Document (PRD) at [PRD.md](file:///d:/Projects/karcis/PRD.md) for architectural rules, database models, and legal compliance structures including:
- **UU PDP No. 27/2022** (Personal Data Protection consent & 72-hour breaches)
- **UU ITE No. 1/2024** (Legality of e-tickets and contracts)
- **UU HKPD No. 1/2022** (Entertainment PBJT local tax cap calculations)
- **PBI No. 23/6/PBI/2021** (Payment Service Provider compliance routing)
- **Redis Concurrency Control & Ticket Wars**
- **Asymmetric Cryptographic Gate Scanning** (Ed25519 offline validation)
