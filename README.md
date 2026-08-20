# EMR Hospital System — Backend API

A REST API for running a multi-specialty hospital or clinic: patients, appointments, clinical notes, billing, pharmacy, and specialty modules (dental, ENT, aesthetics, IV therapy), with role-based access control and audit logging throughout.

Built with **NestJS**, **PostgreSQL**, and **Prisma**.

## Why this exists

Hospitals need one system of record that different roles can safely share — a front-desk clerk booking appointments shouldn't be able to read clinical notes, and a nurse shouldn't be able to edit billing. This API enforces that with a role and guard system baked into every request, plus an audit trail so every sensitive action is traceable after the fact.

## What's implemented

| Area | What it does |
|---|---|
| **Auth** | Email/password login, JWT access + refresh tokens, 2FA (TOTP via `otplib`/`speakeasy` + QR codes), Google OAuth, password reset |
| **Users & Roles** | `SUPERADMIN`, `ADMIN`, `DOCTOR`, `NURSE`, `FRONTDESK` — enforced via guards on every route |
| **Patients** | Patient records with field-level encryption for sensitive data |
| **Appointments** | Scheduling with pagination and filtering |
| **Clinical Notes** | Encounter/visit documentation |
| **Billing** | Invoicing and charges |
| **Pharmacy** | Medication records |
| **Attendance** | Staff attendance tracking |
| **Specialty modules** | Dental, ENT, Aesthetics, IV Therapy — each with its own controller/service/DTOs |
| **Notifications & Reminders** | Scheduled reminders (`@nestjs/schedule`) and outgoing email (Nodemailer/SMTP) |
| **Audit Trail** | Records who did what, for compliance and debugging |
| **Rate limiting** | Global throttling (`@nestjs/throttler`) with per-route overrides for sensitive endpoints |
| **API docs** | Swagger/OpenAPI, auto-generated from decorators |

## Tech stack

- **Framework:** NestJS 11 (Express)
- **Database:** PostgreSQL via Prisma ORM
- **Auth:** Passport (JWT, Local, Google OAuth20), bcrypt, TOTP 2FA
- **Validation:** `class-validator` / `class-transformer`
- **Docs:** `@nestjs/swagger`
- **Testing:** Jest + Supertest

## Project layout

```
src/
  auth/            login, JWT strategy, guards, 2FA
  users/           staff accounts
  patients/        patient records
  appointments/    scheduling
  clinical-notes/  visit documentation
  billing/         invoices & charges
  pharmacy/        medications
  attendance/      staff clock-in/out
  dental/ ent/ aesthetics/ iv-therapy/   specialty encounter modules
  notification/    email + in-app notifications
  reminders/       scheduled reminder jobs
  audit-trail/     action logging
  common/          shared decorators, guards, filters, pagination, response helpers
  utils/           encryption, mail, patient ID generation
prisma/
  schema.prisma    database schema
  seed.ts          creates the initial superadmin account
```

Every domain module follows the same shape: `*.controller.ts` (routes), `*.service.ts` (logic), `*.dto.ts` (request/response validation), `*.module.ts` (wiring).

## Getting started

**Requirements:** Node.js, a PostgreSQL database.

```bash
# 1. Install dependencies
yarn install

# 2. Configure environment
cp .env.example .env
# fill in DATABASE_URL, JWT secrets, ENCRYPTION_KEY, SMTP creds, etc.

# 3. Set up the database
npx prisma migrate deploy
npx prisma db seed        # creates the initial superadmin, see SUPERADMIN_EMAIL/PASSWORD in .env

# 4. Run it
yarn start:dev             # http://localhost:3000
```

Once running, Swagger docs are available at `/api` (or wherever `main.ts` mounts them) for exploring every endpoint interactively.

## Scripts

| Command | Purpose |
|---|---|
| `yarn start:dev` | Run with hot reload |
| `yarn build` | Compile to `dist/` |
| `yarn start:prod` | Run the compiled build |
| `yarn test` | Unit tests |
| `yarn test:e2e` | End-to-end tests |
| `yarn test:cov` | Coverage report |
| `yarn lint` | Lint + autofix |

## Security notes

- Sensitive patient fields are encrypted at rest (`src/utils/crypto.util.ts`) — set a real `ENCRYPTION_KEY` before storing production data.
- JWT secrets, the encryption key, and SMTP credentials must be set via environment variables — nothing sensitive is hardcoded.
- Every route is guarded by default (JWT auth + role check); public routes must opt out explicitly via the `@Public()` decorator.
