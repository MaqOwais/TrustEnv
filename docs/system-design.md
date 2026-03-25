# TrustEnv — System Design

## Overview

Single mobile app serving 4 roles (TrustEnv, Technician, Client, Lab) backed by
Supabase. TrustEnv orchestrates all jobs. Every stakeholder tracks progress in real time
through in-app notifications and live status updates.

---

## Architecture

```
┌─────────────────────────────────────────────────────┐
│                  Mobile App (Expo)                  │
│                                                     │
│  ┌──────────┐ ┌──────────┐ ┌────────┐ ┌─────────┐  │
│  │TrustEnv  │ │Technician│ │ Client │ │   Lab   │  │
│  │   View   │ │   View   │ │  View  │ │  View   │  │
│  └──────────┘ └──────────┘ └────────┘ └─────────┘  │
│                      │                              │
│              Role-based Router                      │
└──────────────────────┼──────────────────────────────┘
                       │
          ┌────────────▼────────────┐
          │       Supabase          │
          │                         │
          │  ┌─────────────────┐    │
          │  │   Auth (JWT)    │    │
          │  │  Magic Links    │    │
          │  └─────────────────┘    │
          │  ┌─────────────────┐    │
          │  │    Database     │    │
          │  │   (Postgres)    │    │
          │  └─────────────────┘    │
          │  ┌─────────────────┐    │
          │  │    Realtime     │    │
          │  │  (live updates) │    │
          │  └─────────────────┘    │
          │  ┌─────────────────┐    │
          │  │    Storage      │    │
          │  │  (PDFs, photos) │    │
          │  └─────────────────┘    │
          │  ┌─────────────────┐    │
          │  │ Edge Functions  │    │
          │  │ (business logic)│    │
          │  └─────────────────┘    │
          └─────────────────────────┘
                       │
          ┌────────────▼────────────┐
          │   Expo Push Service     │
          │  (in-app notifications) │
          └─────────────────────────┘
```

---

## Tech Stack

| Layer             | Technology              | Purpose                                      |
|------------------|-------------------------|----------------------------------------------|
| Mobile           | React Native + Expo     | Single app for all 4 roles (iOS + Android)   |
| Auth             | Supabase Auth           | JWT, magic links, role-based access          |
| Database         | Supabase (Postgres)     | All job, user, sample, status data           |
| Realtime         | Supabase Realtime       | Live job status updates across all roles     |
| Storage          | Supabase Storage        | Lab PDFs, sample photos, chain-of-custody    |
| Business Logic   | Supabase Edge Functions | Job state machine, auto-assignment, triggers |
| Push             | Expo Push Notifications | In-app notifications for all roles           |
| Deep Linking     | Supabase Auth Magic Link| Client onboarding without password           |

---

## Role-Based Access

```
Role        │ Can See                          │ Can Do
────────────┼──────────────────────────────────┼───────────────────────────────────
trustenv    │ All jobs, all users, all data    │ Create jobs, assign techs, send
            │                                  │ reports, advance any status
────────────┼──────────────────────────────────┼───────────────────────────────────
technician  │ Assigned jobs only               │ Log samples, upload photos,
            │                                  │ mark steps complete
────────────┼──────────────────────────────────┼───────────────────────────────────
client      │ Their own jobs only              │ Fill inquiry form, view report,
            │                                  │ approve scope & cost
────────────┼──────────────────────────────────┼───────────────────────────────────
lab         │ Jobs assigned to their lab       │ View sample manifest,
            │                                  │ upload report PDF
```

Row-level security (RLS) enforced in Supabase — each role only queries what they own.

---

## Database Schema

### users
```sql
id            uuid PRIMARY KEY
role          enum('trustenv', 'technician', 'client', 'lab')
name          text
email         text UNIQUE
phone         text
region        text          -- technician only
push_token    text          -- Expo push token
created_at    timestamptz
```

### jobs
```sql
id              uuid PRIMARY KEY
work_order_id   text UNIQUE   -- e.g., WO-19212
claim_number    text
status          enum(
                  'INTAKE', 'SCHEDULED', 'IN_SURVEY',
                  'SAMPLES_COLLECTED', 'SAMPLES_SENT', 'LAB_COMPLETE',
                  'REPORT_SENT', 'APPROVED', 'IN_REMEDIATION', 'CLOSED'
                )
survey_type     enum('asbestos','lead','mold','wildfire','heavy_metals','bacteria')
site_address    text
region          text
client_id       uuid REFERENCES users
technician_id   uuid REFERENCES users
lab_id          uuid REFERENCES users
cost_estimate   numeric
scope_of_work   text
created_at      timestamptz
updated_at      timestamptz
```

### samples
```sql
id              uuid PRIMARY KEY
job_id          uuid REFERENCES jobs
survey_type     text
area_tested     text
material        text
result          text
quantity        text
-- asbestos
friability      enum('F','NF')
condition       enum('G','D','SD')
p5_required     boolean
-- lead
xrf_reading     numeric
paint_condition text
-- mold
moisture_reading numeric
visible_growth  boolean
-- extensible per type
photos          text[]        -- Supabase Storage URLs
created_at      timestamptz
```

### documents
```sql
id            uuid PRIMARY KEY
job_id        uuid REFERENCES jobs
type          enum('lab_report','chain_of_custody','site_diagram','other')
url           text          -- Supabase Storage URL
uploaded_by   uuid REFERENCES users
created_at    timestamptz
```

### status_history
```sql
id            uuid PRIMARY KEY
job_id        uuid REFERENCES jobs
from_status   text
to_status     text
actor_id      uuid REFERENCES users
notes         text
created_at    timestamptz
```

### notifications
```sql
id            uuid PRIMARY KEY
user_id       uuid REFERENCES users
job_id        uuid REFERENCES jobs
message       text
read          boolean DEFAULT false
created_at    timestamptz
```

### survey_templates
```sql
id            uuid PRIMARY KEY
survey_type   text UNIQUE
sample_fields jsonb         -- dynamic fields for technician form
report_schema jsonb         -- expected structure for lab report
```

### technician_availability
```sql
id              uuid PRIMARY KEY
technician_id   uuid REFERENCES users
available       boolean DEFAULT true
region          text
current_jobs    int DEFAULT 0
updated_at      timestamptz
```

---

## Job State Machine

```
INTAKE
  │  TrustEnv reviews + assigns technician (auto or manual)
  ▼
SCHEDULED
  │  Technician travels to site, begins survey
  ▼
IN_SURVEY
  │  Technician collects all samples, marks complete
  ▼
SAMPLES_COLLECTED
  │  TrustEnv confirms dispatch to lab
  ▼
SAMPLES_SENT
  │  Lab uploads report PDF
  ▼
LAB_COMPLETE
  │  TrustEnv adds cost estimate + scope, sends to client
  ▼
REPORT_SENT
  │  Client reviews + approves in app
  ▼
APPROVED
  │  Technician dispatched, performs remediation
  ▼
IN_REMEDIATION
  │  Technician marks complete
  ▼
CLOSED
```

Each transition:
- Recorded in `status_history`
- Triggers push notification to relevant roles
- Updates `jobs.updated_at`

---

## Auto-Assignment Logic

When a job reaches `SCHEDULED`, the system runs:

```
1. Filter technicians by job.region === technician.region
2. Filter by technician_availability.available === true
3. Sort by current_jobs ASC (least loaded first)
4. Assign top result → update job.technician_id
5. Increment technician current_jobs count
6. Notify assigned technician via push
```

TrustEnv admin can override at any point by manually selecting a different technician.

---

## Client Onboarding — 3 Ways

```
Way 1: Magic Link (Supabase Auth)
  TrustEnv creates client profile
  Supabase sends magic link via email
  Client taps → auto-authenticated → opens to inquiry form

Way 2: App Store download
  Client finds app independently
  Signs up with name + email → role = client

Way 3: App download link
  TrustEnv sends App Store / Play Store URL
  Client downloads → signs up → fills inquiry form
```

---

## Notifications

All in-app. Powered by Expo Push Notifications + Supabase Realtime.

```
Supabase Edge Function trigger (on status change)
  ↓
Fetch push tokens for relevant roles
  ↓
Send via Expo Push API
  ↓
App receives push → updates in-app notification feed
  ↓
Supabase Realtime → job status view updates live
```

Notification feed stored in `notifications` table — full history, read/unread state.

---

## File Storage Structure (Supabase Storage)

```
/jobs
  /{job_id}
    /documents
      /lab_report.pdf
      /chain_of_custody.pdf
      /site_diagram.pdf
    /samples
      /{sample_id}
        /photo_1.jpg
        /photo_2.jpg
```

Access controlled by RLS — users can only access files for their own jobs.

---

## Edge Functions

| Function                  | Trigger                          | Action                                        |
|--------------------------|----------------------------------|-----------------------------------------------|
| `on-job-created`          | New row in `jobs`                | Notify TrustEnv, generate work order ID       |
| `on-status-change`        | `jobs.status` updated            | Push notification to relevant roles           |
| `auto-assign-technician`  | Job moves to `SCHEDULED`         | Run assignment algorithm, update technician   |
| `on-document-uploaded`    | New row in `documents`           | Notify TrustEnv, advance status if applicable |
| `generate-magic-link`     | TrustEnv requests client link    | Supabase Auth magic link → send to client     |

---

## Phase 2 Additions

```
Next.js web portal       →  TrustEnv admin + management dashboard
SendGrid                 →  email notifications (formal report delivery)
Separate microservices   →  decouple jobs, reports, notifications at scale
AI report generation     →  generate reports from sample data
Payment integration      →  client pays in-app post-approval
```

---

## Security

- All DB access via Supabase RLS — no role can read outside their scope
- JWT tokens expire and refresh via Supabase Auth
- Storage bucket policies match job ownership
- Edge Functions validate role before executing sensitive operations
- Magic links are single-use and time-limited (Supabase default: 1 hour)
