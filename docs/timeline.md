# TrustEnv — Phase 1 Timeline

## Overview

Phase 1 delivers a fully functional mobile app (React Native + Expo + Supabase)
for all 4 roles with in-app notifications, real-time job tracking, and template-driven
survey forms across 6 survey types.

**Total duration: 13 weeks**

---

## Sub-phases

```
Sub-phase 1  │  Foundation              │  Weeks 1–2
Sub-phase 2  │  TrustEnv Core           │  Weeks 3–4
Sub-phase 3  │  Client Flow             │  Weeks 5–6
Sub-phase 4  │  Technician Flow         │  Weeks 7–8
Sub-phase 5  │  Lab Flow                │  Week 9
Sub-phase 6  │  Notifications & Realtime│  Weeks 10–11
Sub-phase 7  │  Approval, Polish & Beta │  Weeks 12–13
```

---

## Sub-phase 1 — Foundation
**Weeks 1–2**

Set up the entire project backbone before any features are built.

### Deliverables
- [ ] Supabase project setup
  - Auth configured (email, magic links, JWT)
  - Full database schema created (users, jobs, samples, documents, status_history, notifications, survey_templates, technician_availability)
  - Storage buckets configured with RLS policies
  - Row-level security applied per role
- [ ] React Native + Expo project initialized
  - Navigation structure (role-based router)
  - Supabase client configured
  - Environment config (dev / staging / prod)
- [ ] Onboarding screens
  - Name, email, phone collection
  - Role assignment flow
  - Login + session persistence
- [ ] Basic design system
  - Colors, typography, spacing
  - Shared components (buttons, cards, status badges)

---

## Sub-phase 2 — TrustEnv Core
**Weeks 3–4**

Build the orchestration dashboard — the nerve center of the whole system.

### Deliverables
- [ ] All jobs dashboard
  - List view with status, survey type, date, technician
  - Filter by: status, region, survey type, date
- [ ] Job creation flow
  - Select survey type
  - Enter client + site details
  - Auto-generate work order ID
- [ ] Magic link generation
  - TrustEnv creates client profile
  - Generates + sends Supabase magic link via email
- [ ] Manual technician assignment
  - View available technicians by region
  - Assign to job
- [ ] Job detail view
  - Full status timeline
  - All uploaded documents
  - Sample log summary
  - Status advance controls

---

## Sub-phase 3 — Client Flow
**Weeks 5–6**

Build the client experience — from inquiry submission to report viewing.

### Deliverables
- [ ] Magic link onboarding (Way 1)
  - Auto-authenticate via Supabase magic link
  - Land directly on inquiry form
- [ ] App Store onboarding (Way 2 + 3)
  - Self sign-up with email
  - Opens to inquiry form
- [ ] Inquiry form
  - Property address
  - Survey type selection
  - Availability / preferred dates
  - Any specifics (claim number, notes)
  - Submit → job created → TrustEnv notified
- [ ] Job status view
  - Timeline of completed + upcoming steps
  - Plain-language status descriptions
  - Technician info (name, assigned date)
- [ ] Report viewer
  - PDF viewer in-app (lab uploaded report)
  - Cost estimate display
  - Scope of work summary
- [ ] Approval flow
  - In-app approve button
  - Confirmation screen
  - Notifies TrustEnv + technician

---

## Sub-phase 4 — Technician Flow
**Weeks 7–8**

Build the on-site experience — template-driven, photo-first, offline-aware.

### Deliverables
- [ ] My jobs screen
  - Today + upcoming jobs sorted by date
  - Job cards with address, survey type, status
- [ ] Job detail
  - Site address, scope, survey type
  - Step-by-step checklist
- [ ] Template-driven sample logging
  - Form fields change based on survey type:
    - Asbestos: area, material, friability, condition, P5 required
    - Lead: component, substrate, paint condition, XRF reading
    - Mold: location, visible growth, moisture reading
    - Wildfire: debris type, ash level, area affected
    - CAL 17 Heavy Metals: sample location, soil/surface type
    - Bacteria: surface/water source, sample method
- [ ] Photo capture
  - Camera integration per sample location
  - Upload to Supabase Storage
- [ ] Digital chain-of-custody form
  - Auto-populated from job data
  - Technician sign-off
- [ ] Status controls
  - Mark survey started
  - Mark samples collected
  - Drives job status forward
- [ ] Offline support
  - Queue sample logs + photos locally when no signal
  - Auto-sync when connection restores

---

## Sub-phase 5 — Lab Flow
**Week 9**

Build the lab experience — simple, focused on receiving and uploading.

### Deliverables
- [ ] Incoming jobs screen
  - List of jobs with samples en route
  - Sample manifest per job (count, locations, survey type)
- [ ] Job detail for lab
  - Work order ID, site address, technician name
  - Sample breakdown (what was collected, where)
- [ ] PDF report upload
  - File picker → upload to Supabase Storage
  - Upload triggers status → LAB_COMPLETE
  - TrustEnv notified automatically
- [ ] Upload history
  - All uploaded documents per job

---

## Sub-phase 6 — Notifications & Realtime
**Weeks 10–11**

Wire up the nervous system — push notifications and live status updates.

### Deliverables
- [ ] Expo push notification setup
  - Register push tokens on login
  - Store in `users.push_token`
- [ ] Supabase Edge Functions
  - `on-job-created` → notify TrustEnv
  - `on-status-change` → notify correct roles per notification matrix
  - `auto-assign-technician` → assignment algorithm on SCHEDULED
  - `on-document-uploaded` → notify TrustEnv, advance status
  - `generate-magic-link` → client onboarding
- [ ] Auto-assignment
  - Region match → availability check → least loaded first
  - TrustEnv admin override in job detail
- [ ] In-app notification feed
  - Bell icon with unread count
  - Full history of notifications per user
  - Mark as read
- [ ] Supabase Realtime
  - Job status view updates live without refresh
  - New sample logs appear in real time for TrustEnv

---

## Sub-phase 7 — Approval, Polish & Beta
**Weeks 12–13**

Complete the approval loop, fix gaps, and ship to real users.

### Deliverables
- [ ] End-to-end approval flow
  - Full flow test: intake → closed for all survey types
  - Client approval triggers technician + TrustEnv notification
- [ ] Status history view
  - Full audit trail on every job
  - Timestamps + actor for each transition
- [ ] Error states + empty states
  - No jobs yet, no samples, upload failed, offline
- [ ] Performance + reliability
  - Image compression before upload
  - Pagination on job lists
  - Supabase query optimization
- [ ] Internal beta (TrustEnv staff + 1-2 technicians)
  - Real job run-through end to end
  - Collect feedback
- [ ] Bug fixes from beta
- [ ] App Store submission prep (iOS + Android)

---

## Milestone Summary

| Milestone                        | Week | What's Live                                          |
|---------------------------------|------|------------------------------------------------------|
| Foundation ready                 | 2    | Auth, DB, navigation, onboarding                     |
| TrustEnv can create + assign jobs| 4    | Dashboard, job creation, magic link, manual assign   |
| Clients can submit inquiries     | 6    | All 3 client entry flows, inquiry form, status view  |
| Technicians can log on-site      | 8    | Sample forms (all 6 types), photos, offline support  |
| Labs can upload results          | 9    | Lab view, PDF upload, status advance                 |
| Full notifications live          | 11   | Push, realtime, auto-assignment, in-app feed         |
| Beta-ready                       | 13   | End-to-end tested, polished, submitted to App Store  |

---

## Risk & Dependencies

| Risk                                      | Mitigation                                          |
|------------------------------------------|-----------------------------------------------------|
| Offline sync complexity (Sub-phase 4)    | Use Expo + MMKV for local queue, test early         |
| 6 survey templates take longer than expected | Build template engine in Sub-phase 1, templates in Sub-phase 4 |
| App Store review delays                  | Submit TestFlight (iOS) early in Week 12            |
| Supabase Edge Function cold starts       | Keep functions lightweight, test under load         |
| Lab adoption (uploading PDF in app)      | Simple UX, 1-on-1 onboarding with lab contact       |
