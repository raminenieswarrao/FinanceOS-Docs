
---

## 3) `docs/LLD.md` — Low Level Design
Purpose: concrete design: packages, endpoints, DTOs, DB schema, flows, caching keys, security details.

✅ Paste this into `C:\FinanceOS-Docs\docs\LLD.md`

```md
# FinanceOS — Low Level Design (LLD)

**Version:** 1.0  
**Owner:** Eswar  
**Date:** 2026-01-30

---

## 1. Backend Project Structure (Spring Boot)

Recommended package layout:

```text
com.financeos
  ├─ config
  ├─ security
  │   ├─ jwt
  │   ├─ filters
  │   ├─ handlers
  │   └─ model
  ├─ common
  │   ├─ error
  │   ├─ util
  │   └─ dto
  ├─ auth
  │   ├─ controller
  │   ├─ service
  │   ├─ dto
  │   └─ repository
  ├─ user
  │   ├─ controller
  │   ├─ service
  │   ├─ dto
  │   ├─ entity
  │   └─ repository
  ├─ portfolio
  │   ├─ controller
  │   ├─ service
  │   ├─ dto
  │   ├─ entity
  │   └─ repository
  ├─ expenses
  │   ├─ controller
  │   ├─ service
  │   ├─ dto
  │   ├─ entity
  │   └─ repository
  ├─ dashboard
  │   ├─ controller
  │   ├─ service
  │   └─ dto
  ├─ suggestions
  │   ├─ service
  │   └─ rules
  ├─ cache
  │   ├─ CacheService
  │   └─ RedisConfig
  └─ scheduler
      └─ BackupScheduler


Layering rules:

Controller: request/response mapping only

Service: business logic

Repository: DB access only

DTOs: API contract; Entities: DB mapping

2. API Design

Base path (versioned):

/api/v1

2.1 Auth APIs

POST /api/v1/auth/register

POST /api/v1/auth/login

POST /api/v1/auth/refresh

POST /api/v1/auth/logout

2.2 User/Profile APIs

GET /api/v1/users/me

PUT /api/v1/users/me

POST /api/v1/users/me/avatar

2.3 Portfolio APIs

POST /api/v1/portfolio/assets

GET /api/v1/portfolio/assets

POST /api/v1/portfolio/transactions

GET /api/v1/portfolio/holdings

2.4 Expenses APIs

POST /api/v1/expenses

GET /api/v1/expenses?month=YYYY-MM

GET /api/v1/expenses/summary?month=YYYY-MM

2.5 Dashboard APIs

GET /api/v1/dashboard/summary?month=YYYY-MM

2.6 Suggestions APIs

GET /api/v1/suggestions?month=YYYY-MM

3. DTOs (Examples)
RegisterRequest

email

password

fullName

LoginRequest

email

password

AuthResponse

accessToken

expiresInSeconds

user (basic)

UpdateProfileRequest

fullName

currency (USD/INR)

timezone (e.g., America/New_York)

CreateAssetRequest

type (STOCK/MF/ETF)

symbol

name

CreateTransactionRequest

assetId

type (BUY/SELL/SIP)

quantity

price

tradeDate

notes (optional)

CreateExpenseRequest

amount

category

expenseDate

note (optional)

DashboardSummaryResponse

totalPortfolioValue

totalInvested

unrealizedPnL

monthExpenseTotal

charts:

portfolioAllocation[]

expensesByCategory[]

netWorthTrend[] (optional later)

4. Database Schema (MVP)
4.1 users

id (PK)

email (unique)

password_hash

role (USER/ADMIN)

status (ACTIVE/LOCKED)

created_at

updated_at

Indexes:

unique(email)

4.2 user_profile

user_id (PK/FK users.id)

full_name

avatar_path

currency

timezone

created_at

updated_at

4.3 assets

id (PK)

user_id (FK users.id)

asset_type (STOCK/MF/ETF)

symbol

name

created_at

updated_at

Indexes:

(user_id, asset_type)

(user_id, symbol)

4.4 transactions

id (PK)

user_id (FK users.id)

asset_id (FK assets.id)

txn_type (BUY/SELL/SIP)

quantity (decimal)

price (decimal)

trade_date (date)

notes

created_at

Indexes:

(user_id, trade_date)

(asset_id, trade_date)

4.5 expenses

id (PK)

user_id (FK users.id)

category

amount (decimal)

expense_date (date)

note

created_at

Indexes:

(user_id, expense_date)

(user_id, category)

4.6 login_attempts (rate limit support)

id (PK)

email (nullable)

ip_address

success (boolean)

attempted_at

Indexes:

(ip_address, attempted_at)

(email, attempted_at)

Note: rate limiting is primarily via Redis; this table is optional for audit/reporting.

5. Core Flows
5.1 Register Flow

Validate email + password policy

Hash password

Create user + profile

Return success (optionally auto-login)

5.2 Login Flow (JWT)

Apply rate limit check (Redis)

Validate credentials

Issue access token (short TTL)

Issue refresh token (longer TTL, store hash server-side or store in secure cookie strategy)

Return tokens

5.3 Add Transaction Flow

Validate request

Insert transaction

Invalidate dashboard/holdings caches for the user

5.4 Dashboard Summary Flow

Try Redis cache:

key: dash:summary:{userId}:{month}

If miss:

query totals (portfolio + expenses)

build chart data DTOs

store in cache with TTL

Return response

6. Redis Design
6.1 Keys (suggested)

rl:login:ip:{ip} -> counter with TTL

rl:login:email:{email} -> counter with TTL

dash:summary:{userId}:{YYYY-MM} -> cached dashboard summary

holdings:{userId} -> cached holdings (optional)

6.2 TTL

rate limit TTL: 60–300 seconds

dashboard TTL: 60–300 seconds

7. Security (LLD)
7.1 Password Policy (suggested MVP)

Minimum 10 characters

At least 1 uppercase, 1 lowercase, 1 digit, 1 symbol

Block common passwords (optional later)

7.2 JWT Details

Access token: 10–15 min

Refresh token: 7–30 days

Rotate refresh token on use

Store refresh token in HttpOnly cookie (recommended for web)

7.3 File Upload (Avatar)

Allowed MIME: image/jpeg, image/png, image/webp

Max size: e.g., 2MB

Random file name + store under user folder

Prevent script execution in uploads folder

7.4 Headers (production)

HSTS

CSP

X-Frame-Options

X-Content-Type-Options

8. Backup (LLD)
8.1 Backup Script

uses mysqldump

daily scheduled (Windows Task Scheduler)

output filename includes timestamp

retention policy cleanup

9. Frontend (React) Structure
src/
  api/        (axios clients)
  auth/       (auth hooks, token handling)
  pages/      (Login, Register, Dashboard, Portfolio, Expenses, Profile)
  components/ (Navbar, Charts, Tables, Forms)
  store/      (state mgmt if needed)
  utils/

Auth handling:

Prefer HttpOnly cookie refresh token

Access token in memory (or short-term storage)

Axios interceptor adds Authorization header

10. Testing Strategy

Unit tests:

services (business rules)

security utils

Integration tests:

controllers with MockMvc

repository tests (optional)

Optional: Testcontainers for MySQL and Redis integration tests


---

## Where to add what (summary)
- **README.md** → “what is this repo + docs map + tech stack”
- **HLD.md** → architecture, modules, security strategy, deployment, NFRs, roadmap
- **LLD.md** → package structure, endpoints, DTOs, DB schema, flows, Redis keys, security details

---

## Next (when you paste these)
Tell me once you’ve saved them and I’ll give you the next doc set:
1) `docs/API-SPEC.md` (full endpoint request/response examples)  
2) `docs/DB-ERD.md` (tables + relations, ready for draw.io)  
3) `docs/INSTALLATION.md` (exact installs + commands for Java21/Spring/React/Redis/MySQL)
::contentReference[oaicite:0]{index=0}

