# Pipper Reservations: WhatsApp-to-dashboard automation

> Showcase of a production system I designed and built for **Pipper Steakhouse** (two locations in São Paulo state, Brazil).
> The source code is private because it belongs to the client; this page explains what it does and how it works.

## The problem

Reservations arrived as free-text WhatsApp messages in each location's group. Staff copied every booking by hand into spreadsheets, which was slow and led to errors such as duplicates and booking conflicts.

I was the person doing that work, so I built a fix on my own initiative.

## What it does

- Reads every reservation message in the WhatsApp groups automatically, with no fixed command words
- Extracts **name, phone, date, party size, room and meal** from free text
- Recognises **new bookings, changes and cancellations**
- Detects **duplicates** (the same message forwarded twice, or the same booking created twice)
- Scores customers and flags **VIPs** and **event organisers** automatically
- **Real-time dashboard** with KPIs, charts, search, filters, inline editing and edit history
- Live updates in the browser through **Server-Sent Events**, so no page refresh is needed
- CSV export (Excel-ready) and a printable list of the day's reservations
- Editor login with **JWT**; visitors can only view

## Results

- **820+ reservations** and **770+ guests** processed
- Zero manual data entry for reservations
- Running in production every day at both locations

## Architecture

```
WhatsApp group (location A / location B)
        │
        ▼
Evolution API (own DigitalOcean droplet, keeps the WhatsApp session alive)
        │  POST /webhook (JSON)
        ▼
FastAPI app (Railway)
        │  parser extracts the fields from the message (regex, no AI)
        ▼
PostgreSQL (Railway) via SQLAlchemy
        │
        ▼
Dashboard (Server-Sent Events, real-time)
```

## Stack

| Layer | Technology |
|---|---|
| Backend | Python 3.11 · FastAPI |
| Database | PostgreSQL (production) · SQLite (local dev) |
| ORM | SQLAlchemy |
| Frontend | HTML/CSS/JS served by FastAPI |
| WhatsApp bridge | Evolution API (Baileys) on DigitalOcean |
| Hosting | Railway, auto-deploy on every push |

## Security

- JWT authentication on every endpoint that exposes customer data or allows edits
- Escaped HTML output against stored and reflected XSS
- Being rolled out: password hashing (PBKDF2-SHA256), login rate limiting and a shared secret on the webhook
- Message de-duplication by WhatsApp ID, and booking de-duplication by phone, date and meal

## How it started

The first version ran on n8n with Google Sheets as the database. When it outgrew that, I rewrote it as a proper backend with a relational database, a parser and a live dashboard.

---

Built by **Maria Yossimi** · [LinkedIn](https://www.linkedin.com/in/maria-yossimi-6b641a355/) · [Yossimi Solutions](https://yossimisolutions.vercel.app/)
