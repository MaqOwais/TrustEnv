# TrustEnv Workflow

## Overview
TrustEnv orchestrates end-to-end environmental survey jobs across 4 roles —
keeping everyone in sync through a single mobile app with no web portal in Phase 1.

**TrustEnv is the single orchestrator.** Every job flows through them.

---

## Roles

| Role           | How They Access                  | Responsibility                                      |
|---------------|----------------------------------|-----------------------------------------------------|
| **TrustEnv**   | Mobile app                       | Create jobs, assign technicians, orchestrate flow   |
| **Technician** | Mobile app                       | On-site survey, sample collection, photos           |
| **Client**     | Supabase magic link → app        | Submit inquiry, track job, approve report & cost    |
| **Lab**        | Mobile app                       | Receive sample manifest, upload final report PDF    |

**Staff:** 20 technicians across different locations (region-based assignment)

---

## Onboarding

Every user (all roles) goes through onboarding when they first open the app:
```
Name
Email                ← used as login identifier
Phone number
Role assigned        ← by TrustEnv (technician, lab) or self (client)
```
Supabase Auth handles login via email. No separate username — email is the identity.

---

## Notifications

**All notifications are in-app only.** No SMS at any phase.
Every stakeholder tracks everything end-to-end through the app itself.

```
Push notification (Expo)  →  alerts when status changes
In-app feed               →  full history of all job updates
Real-time status          →  Supabase Realtime keeps job view live
```

No need to leave the app to know what's happening at any stage.

---

## Survey Types & Templates

Each survey type has its own **sample collection form** (technician) and **report template** (lab).
The app is template-driven — survey type selected at intake determines all downstream forms.

| Survey Type        | Notes                                      |
|-------------------|--------------------------------------------|
| Asbestos           | PLM analysis, friability, condition rating |
| Lead               | Lead Inspector/Assessor certified          |
| Mold               | Visual + air/surface sampling              |
| Wildfire           | Post-fire debris, ash, particulate         |
| CAL 17 Heavy Metals| California-specific heavy metals panel     |
| Bacteria           | Surface/water sampling                     |

---

## 2-Phase Approach

### Phase 1 — Mobile Only
```
React Native + Expo          →  mobile app (all 4 roles)
Supabase                     →  everything backend
  ├── Auth                      magic links, JWT, role-based access
  ├── Database                  Postgres — jobs, users, samples, statuses
  ├── Storage                   PDFs, sample photos
  ├── Realtime                  live job status updates
  └── Edge Functions            business logic
Expo Push Notifications      →  in-app push only
```

### Phase 2 — Web Portal + External Notifications
```
Next.js                      →  web portal (TrustEnv + admin)
SendGrid                     →  email notifications
Everything from Phase 1 stays
```

> AI-based report generation → future phase (not in scope now)
> Lab creates report manually → uploads PDF via app

---

## How a Client Enters the System

### Way 1 — TrustEnv sends a magic link (Supabase Auth)
```
Client calls TrustEnv (phone/email)
        ↓
TrustEnv creates client profile + generates magic link in app
        ↓
Supabase sends link to client via email
        ↓
Client taps link
        ↓
   App installed?
   ↙              ↘
  YES               NO
   ↓                 ↓
Authenticated     App Store / Play Store
opens directly    Downloads app
to inquiry form   Magic link auto-authenticates
        ↓
Client fills: property address, survey type needed,
              availability, any specifics
        ↓
Submits → job created → TrustEnv notified
```

### Way 2 — Client downloads app independently
```
Client finds TrustEnv on App Store / Play Store
        ↓
Signs up → fills inquiry form → job created
```

### Way 3 — TrustEnv sends app store link
```
Client calls TrustEnv (phone/email)
        ↓
TrustEnv sends app download link via in-app message
        ↓
Client taps link
        ↓
   App installed?
   ↙              ↘
  YES               NO
   ↓                 ↓
Opens app         App Store / Play Store
to inquiry form   Downloads app → opens to inquiry form
        ↓
Client fills: property address, survey type needed,
              availability, any specifics
        ↓
Submits → job created → TrustEnv notified
```

> Way 1 and Way 3 differ: Way 1 uses a Supabase magic link (auto-authenticates client).
> Way 3 sends a plain app store link — client creates their own account after downloading.

---

## Workflow — Step by Step

### Step 1: Intake
```
Actor:   Client
Action:  Fills inquiry form (via magic link or app)
         - Property address
         - Survey type (Asbestos / Lead / Mold / Wildfire / Heavy Metals / Bacteria)
         - Availability / preferred dates
         - Contact info
         - Any specifics (insurance claim number, referring company, etc.)
Output:  Job created → status: INTAKE
Notify:  TrustEnv (push)
```

### Step 2: Scheduling
```
Actor:   TrustEnv + System
Action:  TrustEnv reviews inquiry
         System auto-assigns nearest available technician
           based on: region match + availability
         TrustEnv admin can override assignment if needed
         Confirms survey date
Output:  Job status → SCHEDULED
Notify:  Client     (confirmed date + technician name)
         Technician  (job assigned, site address, survey type, scope)
```

### Step 3: On-Site Survey
```
Actor:   Technician
Action:  Travels to site
         App loads the correct sample form for the survey type
         Logs on-site:
           - Sample locations (area tested)
           - Material type + condition
           - Friability (where applicable)
           - Photos of each sample location
           - Site diagram sketch
         Marks survey complete
Output:  Job status → IN_SURVEY → SAMPLES_COLLECTED
Notify:  TrustEnv  (samples collected, ready to dispatch)
         Lab        (incoming samples — manifest sent automatically)
```

### Step 4: Samples to Lab
```
Actor:   TrustEnv
Action:  Confirms physical samples dispatched to lab
Output:  Job status → SAMPLES_SENT
Notify:  Lab     (sample manifest: job ID, count, locations, survey type)
         Client  (samples at lab, results coming)
```

### Step 5: Lab Analysis
```
Actor:   Lab (e.g., MicroTest)
Action:  Receives physical samples
         Runs analysis (PLM, XRF, etc. depending on survey type)
         Creates report PDF manually (per survey template)
         Uploads PDF to app
Output:  Job status → LAB_COMPLETE
Notify:  TrustEnv  (results uploaded — review needed)
         Technician (FYI)
```

### Step 6: Report Review & Dispatch
```
Actor:   TrustEnv
Action:  Reviews uploaded lab PDF
         Adds cost estimate + scope of work for remediation
         Sends report package to client via app
Output:  Job status → REPORT_SENT
Notify:  Client (report ready — view + approve in app)
```

### Step 7: Client Approval
```
Actor:   Client
Action:  Opens report in app
         Reviews findings, cost, scope of work
         Approves in-app
Output:  Job status → APPROVED
Notify:  TrustEnv  (approved — schedule remediation)
         Technician (prepare for remediation job)
```

### Step 8: Remediation
```
Actor:   Technician
Action:  Dispatched to site
         Performs remediation work per approved scope
         Marks job complete in app
Output:  Job status → CLOSED
Notify:  Client     (job complete)
         TrustEnv   (job closed)
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

**Phase 1 Channels:** Push notifications (Expo) — in-app only
**Phase 2 Channels:** + Email (SendGrid)

---

## What Each Role Sees in the App

### TrustEnv
```
- All jobs dashboard
    Filter by: status, date, technician, location, survey type
- Job detail: full timeline, sample logs, uploaded PDFs
- Technician auto-assigned by system (region + availability)
- Admin can override assignment manually
- Generate & send magic link to client
- Add cost estimate + scope before dispatching report
- Manually advance job status when needed
```

### Technician
```
- My jobs (today + upcoming), sorted by location
- Job detail: site address, survey type, scope
- Template-driven sample log (form changes per survey type)
    Asbestos → area, material, friability, condition, P5 required
    Lead     → component, substrate, paint condition, XRF reading
    Mold     → location, visible growth, moisture reading
    ...etc
- Photo capture per sample location
- Digital chain-of-custody form
- Mark steps complete → drives status forward
```

### Client
```
- Job status timeline (what's done, what's next, plain language)
- Technician info (name, eta when en route)
- Report PDF viewer (lab results)
- Cost + scope review
- In-app approval
- Notifications at every step
```

### Lab
```
- Incoming sample manifest
    Job ID, work order, survey type, sample count, collection locations
- Upload final report PDF (per survey type template)
- Job reference: site address, technician, dates
```

---

## Data Model (core)

```
Job {
  id
  work_order_id           // e.g., WO-19212
  claim_number
  status                  // enum: INTAKE → CLOSED
  survey_type             // asbestos | lead | mold | wildfire | heavy_metals | bacteria
  site_address
  region                  // for technician assignment
  client_id
  technician_id
  lab_id
  cost_estimate
  scope_of_work
  created_at
  updated_at
  samples[]
  documents[]             // lab PDF, chain-of-custody
  status_history[]        // full audit trail with timestamps + actor
}

Sample {
  id
  job_id
  survey_type             // determines which fields are relevant
  area_tested
  material
  result
  quantity
  // Asbestos-specific
  friability              // F | NF
  condition               // G | D | SD
  p5_required             // bool
  // Lead-specific
  xrf_reading
  paint_condition
  // Mold-specific
  moisture_reading
  visible_growth
  // ...extensible per survey type
  photos[]
}

User {
  id
  role                    // trustenv | technician | client | lab
  name
  email
  phone
  region                  // technician only
  magic_link_token        // client onboarding (Supabase Auth)
}

SurveyTemplate {
  id
  survey_type
  sample_fields[]         // defines what technician sees on-site
  report_schema           // defines expected lab report structure
}
```

---

## Open Questions for TrustEnv

| #  | Question                                                                 | Status      |
|----|-------------------------------------------------------------------------|-------------|
| 1  | How many technicians on staff?                                           | ✓ 20, multi-location |
| 2  | Multiple lab partners or exclusively MicroTest?                          | Pending     |
| 3  | Survey types offered?                                                    | ✓ 6 types   |
| 4  | How is cost estimate calculated — manual or formula-based?               | Pending     |
| 5  | Client approval — legal e-signature or simple in-app confirm?            | Pending     |
| 6  | Average turnaround from survey to report delivery?                       | Pending     |
| 7  | Compliance/retention requirement for job records?                        | Pending     |
