# TrustEnv Workflow

## Overview
TrustEnv orchestrates end-to-end environmental survey jobs (Asbestos, Lead, etc.)
across 4 roles — keeping everyone in sync with no web portal required in Phase 1.

**TrustEnv is the single orchestrator.** Every job flows through them.

---

## Roles

| Role           | How They Access              | Responsibility                                      |
|---------------|------------------------------|-----------------------------------------------------|
| **TrustEnv**   | Mobile app                   | Create jobs, assign technicians, orchestrate flow   |
| **Technician** | Mobile app                   | On-site survey, sample collection, photos           |
| **Client**     | Deep link → app download     | Submit inquiry, track job, approve report & cost    |
| **Lab**        | Mobile app                   | Receive sample manifest, upload final report PDF    |

---

## 2-Phase Approach

### Phase 1 — Mobile Only
```
React Native + Expo     → mobile app (all 4 roles)
Node.js API             → single backend
PostgreSQL              → jobs, users, samples, statuses
S3                      → PDFs, photos
Twilio + SendGrid       → SMS + email notifications
Branch.io               → deep link / deferred deep linking
```

### Phase 2 — Web Portal Added
```
Next.js                 → web portal (TrustEnv + admin)
Everything from Phase 1 stays
```

> AI-based report generation → future phase (not in scope now)

---

## How a Client Enters the System

### Way 1 — TrustEnv sends a deep link
```
Client calls TrustEnv (phone/email)
        ↓
TrustEnv generates a unique deep link in app
        ↓
Link sent to client via SMS or email
        ↓
Client taps link
        ↓
   App installed?
   ↙           ↘
  YES            NO
   ↓              ↓
Opens app     App Store / Play Store
to pre-filled      ↓
inquiry form  Downloads app → opens to same form
        ↓
Client fills: availability, property details, job type
        ↓
Submits → job created in system
```

### Way 2 — Client downloads app independently
```
Client finds TrustEnv on App Store / Play Store
        ↓
Signs up → fills inquiry form → job created
```

---

## Workflow — Step by Step

### Step 1: Intake
```
Actor:   Client
Action:  Fills inquiry form (via deep link or app)
         - Property address
         - Type of survey needed
         - Availability / preferred dates
         - Contact info
Output:  Job created with status → INTAKE
Notify:  TrustEnv (SMS + in-app)
```

### Step 2: Scheduling
```
Actor:   TrustEnv
Action:  Reviews inquiry, assigns technician, confirms date
Output:  Job status → SCHEDULED
Notify:  Client (date + technician details)
         Technician (job assigned, address, scope)
```

### Step 3: On-Site Survey
```
Actor:   Technician
Action:  Travels to site, collects samples
         Logs in app:
           - Sample locations (area tested)
           - Material type
           - Photos
           - Site diagram
         Marks survey complete
Output:  Job status → IN_SURVEY → SAMPLES_COLLECTED
Notify:  TrustEnv (samples collected)
         Lab (incoming samples — manifest sent)
```

### Step 4: Samples to Lab
```
Actor:   TrustEnv
Action:  Confirms samples dispatched to lab
Output:  Job status → SAMPLES_SENT
Notify:  Lab (sample manifest with job details)
         Client (samples at lab, results coming)
```

### Step 5: Lab Analysis
```
Actor:   Lab (e.g., MicroTest)
Action:  Receives samples
         Runs PLM analysis
         Creates report PDF (manually, as today)
         Uploads PDF to app
Output:  Job status → LAB_COMPLETE
Notify:  TrustEnv (results uploaded)
         Technician (FYI)
```

### Step 6: Report Review & Dispatch
```
Actor:   TrustEnv
Action:  Reviews uploaded lab PDF
         Adds cost estimate + scope of work
         Sends to client via app
Output:  Job status → REPORT_SENT
Notify:  Client (report ready — view + approve in app)
```

### Step 7: Client Approval
```
Actor:   Client
Action:  Opens report in app
         Reviews findings, cost, scope
         Signs off / approves
Output:  Job status → APPROVED
Notify:  TrustEnv (approved, ready to schedule remediation)
         Technician (prepare for job)
```

### Step 8: Remediation
```
Actor:   Technician
Action:  Goes on-site, performs remediation work
         Marks job complete in app
Output:  Job status → CLOSED
Notify:  Client (job complete)
         TrustEnv (job closed)
```

---

## Job Status Phases

```
INTAKE → SCHEDULED → IN_SURVEY → SAMPLES_COLLECTED → SAMPLES_SENT
      → LAB_COMPLETE → REPORT_SENT → APPROVED → IN_REMEDIATION → CLOSED
```

---

## Notification Matrix

| Event                        | TrustEnv | Technician | Client | Lab |
|-----------------------------|----------|------------|--------|-----|
| Inquiry submitted            | ✓        |            |        |     |
| Job scheduled                |          | ✓          | ✓      |     |
| Technician en route          |          |            | ✓      |     |
| Samples collected            | ✓        |            |        | ✓   |
| Samples dispatched to lab    |          |            | ✓      | ✓   |
| Lab results uploaded         | ✓        | ✓          |        |     |
| Report sent to client        |          |            | ✓      |     |
| Client approved              | ✓        | ✓          |        |     |
| Remediation complete         |          |            | ✓      |     |

**Channels:** SMS (Twilio) + Push notification (Expo) + Email (SendGrid)

---

## What Each Role Sees in the App

### TrustEnv
```
- All jobs dashboard (filter by status, date, technician)
- Job detail: full timeline, all uploads, sample logs
- Assign / reassign technician
- Generate & send deep link to client
- Add cost estimate + scope before sending report
- Advance job status manually when needed
```

### Technician
```
- My jobs (today + upcoming)
- Job detail: address, scope, what to sample
- Sample logging: area, material, photo, condition
- Digital chain-of-custody form
- Mark steps complete (drives status forward)
```

### Client
```
- Job status timeline (what's done, what's next)
- Technician info (name, arrival time)
- Report PDF viewer
- Approve + sign off on scope & cost
- Notifications at every step
```

### Lab
```
- Incoming sample manifest (job ID, sample count, locations)
- Upload final report PDF
- Job reference details (address, technician, work order)
```

---

## Data Model (core)

```
Job {
  id
  work_order_id         // e.g., WO-19212
  claim_number
  status                // enum: phases above
  survey_type           // Limited | Full | Other
  site_address
  client_id
  technician_id
  lab_id
  created_at
  updated_at
  samples[]
  documents[]           // lab PDF, chain-of-custody
  status_history[]      // full audit trail
  cost_estimate
  scope_of_work
}

Sample {
  id
  job_id
  area_tested
  material
  result
  quantity
  friability            // F | NF
  condition             // G | D | SD
  photos[]
}

User {
  id
  role                  // trustenv | technician | client | lab
  name
  email
  phone
  deep_link_token       // for client magic link
}
```

---

## Open Questions for TrustEnv

1. How many technicians are currently on staff?
2. Are there multiple lab partners or exclusively MicroTest?
3. What survey types beyond Asbestos are offered? (Lead, Mold, etc.)
4. How is cost estimate currently calculated — manual or formula-based?
5. Does client approval require a legal e-signature or a simple in-app confirm?
6. What's the current average turnaround from survey to report delivery?
7. Any compliance/retention requirement for how long job records must be kept?
