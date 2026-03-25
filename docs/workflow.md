# TrustEnv Workflow

## Overview
End-to-end orchestration of environmental survey jobs (e.g., Asbestos, Lead) from initial client contact through final remediation — keeping **TrustEnv**, **Technician**, **Client**, and **3rd Party Lab** in sync at every step.

---

## The 7-Step Workflow

```
Step 1  →  Client contacts TrustEnv (email / phone)
Step 2  →  TrustEnv schedules survey, collects availability & job specifics from client
Step 3  →  Technician dispatched to site — collects samples
Step 4  →  Samples sent to 3rd Party Lab for analysis
Step 5  →  Lab returns results → Report is generated
Step 6  →  Report shared with all parties (Client, Company, Technician)
           Report contains:
             - Detected materials (e.g., <1% Chrysotile Asbestos)
             - Cost breakdown
             - Scope of work / how job will be executed
Step 7  →  Client confirms → Technician dispatched to perform remediation
```

---

## Stakeholders & Their Roles

| Stakeholder      | Role                                                                 |
|-----------------|----------------------------------------------------------------------|
| **Client**       | Initiates request, receives updates, confirms scope & cost           |
| **TrustEnv**     | Orchestrates entire workflow, assigns technicians, generates reports  |
| **Technician**   | On-site survey & sample collection; performs final remediation       |
| **3rd Party Lab**| Analyzes samples via PLM, returns certified results (e.g., MicroTest)|

---

## Orchestration Requirements

### Real-Time Status Tracking
Each job moves through defined phases with a status visible to all stakeholders:

```
INTAKE → SCHEDULED → IN_SURVEY → SAMPLES_SENT → LAB_ANALYSIS
      → REPORT_READY → AWAITING_APPROVAL → APPROVED → IN_REMEDIATION → CLOSED
```

### Notifications (per stakeholder)

| Event                          | Client | TrustEnv | Technician | Lab |
|-------------------------------|--------|----------|------------|-----|
| Job created                   | ✓      | ✓        |            |     |
| Survey scheduled              | ✓      | ✓        | ✓          |     |
| Technician en route           | ✓      |          | ✓          |     |
| Samples collected & sent      |        | ✓        | ✓          | ✓   |
| Lab results received          | ✓      | ✓        | ✓          |     |
| Report ready for review       | ✓      | ✓        |            |     |
| Client approved scope & cost  | ✓      | ✓        | ✓          |     |
| Remediation complete          | ✓      | ✓        | ✓          |     |

### Channels
- **Email** — formal notifications, report delivery
- **SMS** — real-time alerts (technician en route, status changes)
- **Portal** — web dashboard for client + TrustEnv staff to track job status
- **Mobile App** — technician workflow (job details, sample submission, status updates)

---

## Sample Report Data Model (derived from WO-19212)

```
Job {
  work_order_id       // e.g., WO-19212
  claim_number        // e.g., 009813123-801
  client_name         // e.g., COMSTOCK, Travis
  site_address        // e.g., 7445 Summer Ave, Citrus Heights, CA
  report_prepared_for // company name, contact, email (e.g., RYTECH)
  survey_date
  report_date
  survey_type         // Limited | Full | Other
  technician          // Jennifer Schiff (CSST# 15-5492)
  consultant          // Victor Ruiz (CAC# 15-5589)
  lab                 // MicroTest Laboratories (NVLAP: 200999-0)
  lab_project_id      // e.g., MT012686974
  status              // enum: workflow phases above
  samples[]           // see below
  result              // DETECTED | NOT_DETECTED
  recommendation      // NAD | trace | <1%CH | >1%CH
}

Sample {
  sample_id           // e.g., 1a, 1b, 1c
  lab_sample_id       // e.g., 86974-1B
  area_tested         // e.g., Kids Bedroom N Wall Closet
  material            // e.g., Tan Joint Compound
  result              // e.g., <1% Chrysotile
  quantity            // e.g., 60sf
  friability          // F | NF
  condition           // G | D | SD
  p5_required         // bool
}
```

---

## Integration Points

| System           | Integration                                              |
|-----------------|----------------------------------------------------------|
| **Lab (MicroTest)** | Email/API — receive results, parse into job record    |
| **Scheduling**   | Calendar sync for technician dispatch                    |
| **Payments**     | Cost estimate approval flow before remediation           |
| **Document Gen** | Auto-generate PDF reports matching current template      |
| **eSignature**   | Client signs off on scope/cost approval                  |

---

## Open Questions for TrustEnv
*(to be answered before system design is finalized)*

1. Does the 3rd party lab provide a structured API/portal, or only email + PDF?
2. How are technicians currently assigned to jobs — manually or any scheduling tool?
3. Is client communication currently phone/email only, or is there an existing portal?
4. Are there multiple lab partners, or exclusively MicroTest?
5. What does "cost detailing" in Step 6 look like — is it auto-calculated or manually entered?
6. Do clients ever reject the scope? What is the revision flow?
7. Is the remediation work subcontracted or done by TrustEnv technicians?
8. Are there multiple report types beyond Asbestos (Lead, Mold, etc.)?
9. What compliance/retention requirements exist for reports and chain-of-custody docs?
10. Current volume: how many jobs per day/week?
