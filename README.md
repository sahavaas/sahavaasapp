# Sahavaas Launch Backend (Supabase + Vercel)

Production-ready multi-tenant backend for **Sahavaas** (Zero-Fee UPI QR Maintenance, RWA Balance Funds, Technicians Directory, Priority Complaints, Events/RSVPs, Multi-Vehicle Parking Finder, and SaaS Subscription Verification).

## Architecture Overview

- **Database & Storage**: Supabase PostgreSQL (16 tables with Row-Level Security enabled) + `payment-proofs` private Storage bucket.
- **API Layer**: 9 Vercel Serverless Functions (`api/`) using `@supabase/supabase-js` (service role) with stateless `HS256` JWT `HttpOnly` cookie auth, RBAC (`superadmin`, `admin`, `resident`, `security`), input sanitization, and sliding-window rate limiting.
- **Frontend**: `public/index.html` + `public/sahavaas-api.js` served via `vercel.json`.

## 1. Supabase Setup (5 Minutes)

1. Create a new project at [supabase.com](https://supabase.com).
2. Open **SQL Editor** and run `supabase/schema.sql`.
3. (Optional demo data) Run `supabase/seed.sql` to populate `OAK-01` (Oakridge Residency) and `GRN-02` (GreenView Meadows).
4. Go to **Project Settings -> API** and copy:
   - `SUPABASE_URL`
   - `SUPABASE_ANON_KEY`
   - `SUPABASE_SERVICE_ROLE_KEY`

## 2. Vercel Deployment (3 Minutes)

1. Push this folder to GitHub or deploy directly via Vercel CLI:
   ```bash
   npm install
   npx vercel --prod
   ```
2. In **Vercel Project Settings -> Environment Variables**, add:
   - `SUPABASE_URL`
   - `SUPABASE_ANON_KEY`
   - `SUPABASE_SERVICE_ROLE_KEY`
   - `JWT_SECRET` (generate with `openssl rand -hex 32`)
   - `ALLOWED_ORIGIN` (your production domain, e.g., `https://sahavaas.vercel.app`)
   - `PAYMENT_PROOFS_BUCKET=payment-proofs`
3. Verify deployment by visiting `https://your-app.vercel.app/api/health`.

## API Endpoints

| Endpoint | Methods | Purpose |
| :--- | :--- | :--- |
| `/api/health` | `GET` | Supabase connectivity & latency check |
| `/api/auth` | `GET`, `POST` | Tenant list, Login (`admin`/`resident`/`security`/`superadmin`), Register, Logout |
| `/api/workspace` | `GET` | Role-scoped workspace hydration (security role isolated from financials) |
| `/api/bills` | `POST` | UPI QR UTR + screenshot proof upload, Admin verify/reject, Cash/Cheque offline entry, Monthly bill generation |
| `/api/operations` | `POST` | RWA opening corpus, expenses, technicians, complaints triage, UPI config, flats |
| `/api/community` | `POST` | Notices (with app-open popup flag), Events/Game RSVPs, 1-Flat-1-Vote Polls |
| `/api/parking` | `GET`, `POST` | Multi-vehicle management & plate/last-4/slot owner search |
| `/api/subscription` | `GET`, `POST` | Flat-count pricing, RWA UPI proof submission, Super Admin verification |
| `/api/system` | `POST` | Soft-delete recycle bin, restore, empty trash, JSON backup snapshots |
