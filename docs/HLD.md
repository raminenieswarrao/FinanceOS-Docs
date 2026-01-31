# FinanceOS — High Level Design (HLD)

**Version:** 1.0  
**Owner:** Eswar  
**Date:** 2026-01-30

---

## 1. Purpose

FinanceOS is a secure, scalable personal finance platform to:
- Track investments (stocks, mutual funds)
- Track expenses, budgets, and subscriptions
- Track goals and progress
- Show dashboards and charts
- Provide rule-based suggestions (AI-ready, disabled initially)

---

## 2. Scope

### 2.1 MVP (Initial Version)
**In scope**
- Secure auth: register/login/logout
- User profile + avatar upload
- Portfolio: assets + transactions + holdings computation
- Expenses: add/list + monthly summary
- Dashboard: KPI cards + charts
- Redis: caching + rate limiting
- Backups: MySQL backup process

**Out of scope (later)**
- AI insights/chat (feature flag OFF)
- Bank auto-sync
- Mobile apps (Flutter/React Native later)

---

## 3. Architecture Overview

### 3.1 System Context
```text
React UI (Vite + TS)  --->  Spring Boot REST API  --->  MySQL (XAMPP)
           |                      |
           |                      +--> Redis (cache + rate limit)
           |                      +--> File Storage (avatars)
           |                      +--> Scheduler (backups/cleanup)
           |
         HTTPS (TLS)
3.2 Architecture Style

Modular Monolith (single deployable backend) with clear domain modules.

Supports future evolution into services if needed.

3.3 CAP Theorem Choice

FinanceOS is CP-oriented (Consistency + Partition Tolerance).

Financial correctness is prioritized over always responding during network issues.

4. Key Modules

Auth Module

register/login/logout

password hashing

JWT (access + refresh)

rate limiting

User Profile Module

profile data: name, currency, timezone, preferences

avatar upload with validation

Portfolio Module

assets (stock/MF/ETF types)

transactions (buy/sell/sip)

holdings and P/L calculations

Expenses & Budget Module

expense capture

monthly summaries

budget planning (MVP can start with expenses only)

Dashboard Module

aggregates portfolio + expenses + goals

provides chart-friendly DTOs

Suggestions Module

rule-based insights (always-on)

AI-ready hooks (disabled)

Cache Module (Redis)

dashboard caching

rate limiting store

optional session store (if needed later)

Scheduler/Backup Module

daily DB backups

cleanup tasks (logs/cache)

5. Technology Stack
5.1 Backend

Java 21

Spring Boot

Spring Security (JWT)

Spring Data JPA (Hibernate)

Maven

5.2 Frontend

React + Vite + TypeScript

UI: Tailwind or Material UI (choose later)

Charts: Chart.js or Recharts (choose later)

5.3 Data

MySQL (XAMPP)

Redis cache

5.4 Tooling (Free)

IntelliJ IDEA Community

Git + GitHub

Postman

Draw.io

JUnit 5 + Mockito

6. Security (High Level)
6.1 Authentication & Authorization

Password hashing using BCrypt/Argon2 (server side)

JWT:

short-lived access token

refresh token (stored securely; recommended HttpOnly cookie)

Role-based access: USER, ADMIN

Session fixation protection if sessions are used later

6.2 OWASP Baseline Protections

SQL Injection: prepared statements / JPA

XSS: output encoding + secure headers

CSRF: required if using cookies; handled using standard patterns

CORS: allow only known frontend origins

Rate limiting: Redis-backed for auth endpoints

Audit logs for critical actions

6.3 Data Protection

TLS (HTTPS) in transit (production)

Encrypted backups (recommended)

Secure file upload for avatars:

allowlist file types

size limit

random file name

store safely

Note: True “end-to-end encryption” (server cannot read data) is optional and may be added later for special “vault” fields. MVP will use TLS + secure storage + access control.

7. Caching Strategy (Redis)

Cache dashboard summaries and computed aggregates

TTL typically 60–300 seconds

Cache invalidation on write events (transactions/expenses updates)

8. Backup Strategy

MySQL dumps via mysqldump

Scheduled daily backups

Retention: last 7 daily + last 4 weekly (configurable)

Store backups with restricted permissions; optional encrypted zip

9. Deployment (HLD)
9.1 Local Dev

React: localhost:5173

Spring Boot: localhost:8080

MySQL: XAMPP

Redis: Docker or WSL

9.2 Future Production

Host backend on a VM/container

Managed DB (optional)

HTTPS with valid cert

React served via CDN or via Spring static hosting

10. Non-Functional Requirements (NFRs)

Security: OWASP baseline, secure auth, rate limiting

Performance: dashboard under 2 seconds for typical user datasets

Reliability: daily backups; graceful failure handling

Maintainability: modular packages + layering; strong tests

Scalability: support growth with caching, indexes, pagination

11. Roadmap
Phase 1 (MVP)

Auth + profile + avatar

Portfolio + expenses

Dashboard + charts

Redis caching + rate limit

Backups

Phase 2

Subscriptions + goal tracking

Suggestions engine improvements

Export reports

Phase 3

AI insights (feature flag ON for paid users)

Mobile app (Flutter) using same REST APIs