# FinanceOS Docs

Design and architecture documentation for **FinanceOS** — a secure, scalable personal finance tracking platform.

## What is FinanceOS?
FinanceOS helps users:
- Track investments (stocks, mutual funds)
- Track expenses and budgets
- Track subscriptions
- Track goals (car, emergency fund, etc.)
- View dashboards & charts
- Get rule-based suggestions (AI-ready, but disabled initially)

## Docs
- `docs/HLD.md` — High Level Design (architecture, modules, security, deployment)
- `docs/LLD.md` — Low Level Design (package structure, DB schema, APIs, flows)

## Tech Stack (Target)
- Backend: Java 21, Spring Boot, Spring Security (JWT), Spring Data JPA, Maven
- Frontend: React (Vite + TypeScript), charts library
- DB: MySQL (XAMPP)
- Cache: Redis
- Tools: IntelliJ CE, Git, Postman, Draw.io, JUnit, Mockito

## Branch Strategy
- `main` = stable
- `develop` = active integration
- `feature/*` = feature work

## Notes
- AI features will be designed but controlled via feature flag (`ai.enabled=false`) until monetization.
- Security is a primary requirement (OWASP baseline, secure auth, rate limiting, secure upload, encrypted backups).
