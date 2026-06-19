# ClinicAI Workflow Diagram

Visual representation of the ClinicAI medical report analysis workflow.

---

## Complete Workflow (Data Flow)

```
┌────────────────────────────────────────────────────────────────────────────┐
│                          CLINICAI WORKFLOW                                  │
└────────────────────────────────────────────────────────────────────────────┘

                        ┌─────────────────────┐
                        │  Medical Report     │
                        │  (REST/Webhook)     │
                        └──────────┬──────────┘
                                   │
                                   │ JSON POST
                                   ▼
                    ┌──────────────────────────────┐
                    │   1. WEBHOOK TRIGGER         │
                    │   ────────────────────       │
                    │   • Receive POST request      │
                    │   • Extract JSON payload      │
                    │   • Validate fields           │
                    │   • Pass to next node         │
                    │   Time: ~0.5 seconds         │
                    └──────────────┬───────────────┘
                                   │
                                   │ Validated data
                                   ▼
                    ┌──────────────────────────────┐
                    │   2. GEMINI CHAT MODEL       │
                    │   ─────────────────────      │
                    │   • LLM analysis engine       │
                    │   • Medical prompt            │
                    │   • Extract clinical data     │
                    │   • Return JSON              │
                    │   Time: 15-20 seconds        │
                    └──────────────┬───────────────┘
                                   │
                                   │ JSON analysis
                                   ▼
                    ┌──────────────────────────────┐
                    │   3. PARSE RESPONSE          │
                    │   ──────────────────         │
                    │   • Parse JSON output         │
                    │   • Map to fields             │
                    │   • Enrich with metadata      │
                    │   • Handle null values        │
                    │   Time: ~0.5 seconds         │
                    └──────────────┬───────────────┘
                                   │
                    ┌──────────────┴──────────────┐
                    │                             │
                    ▼                             ▼
        ┌──────────────────────┐    ┌──────────────────────┐
        │  4. EMAIL DOCTOR     │    │  5. EMAIL PATIENT    │
        │  ────────────────    │    │  ──────────────────  │
        │  • Full analysis      │    │  • Patient summary   │
        │  • Clinical details   │    │  • Plain language    │
        │  • All findings       │    │  • Action items      │
        │  Time: 5-10 sec       │    │  Time: 5-10 sec      │
        └──────────┬────────────┘    └──────────┬───────────┘
                   │                            │
                   │ Email send                 │ Email send
                   ▼                            ▼
        ┌──────────────────────┐    ┌──────────────────────┐
        │   DOCTOR'S INBOX     │    │   PATIENT'S INBOX    │
        │                      │    │                      │
        │  Full Analysis       │    │  Health Summary      │
        │  - Risks: Critical   │    │  - Your Medications  │
        │  - Findings: 7 items │    │  - Next Steps        │
        │  - Labs: Abnormal    │    │  - Follow-up Tests   │
        │  - Tests: Recommended│    │  - Contact Doctor    │
        └──────────────────────┘    └──────────────────────┘

                        Total Processing Time: ~30 seconds
```

---

## Component Interaction Diagram

```
┌─────────────────┐
│  External System │  (EHR, Patient Portal, Form)
│  (REST Client)   │
└────────┬─────────┘
         │ POST /webhook/medical-analyzer
         │ {patientName, patientAge, medicalReport, emails...}
         │
         ▼
    ┌──────────────────────────┐
    │     n8n PLATFORM         │
    │  ══════════════════════  │
    │                          │
    │  ┌────────────────────┐  │
    │  │  Webhook Node      │  │
    │  │  (HTTP Trigger)    │  │
    │  └─────────┬──────────┘  │
    │            │             │
    │            ▼             │
    │  ┌────────────────────┐  │
    │  │ Gemini LLM Node    │  │ ◄──────┐
    │  │ (AI Analysis)      │  │        │
    │  └─────────┬──────────┘  │        │ Google API
    │            │             │        │
    │            ▼             │        │
    │  ┌────────────────────┐  │        │
    │  │ Parse/Map Node     │  │        │
    │  │ (Data Transform)   │  │        │
    │  └─────────┬──────────┘  │        │
    │            │             │        │
    │    ┌───────┴────────┐    │        │
    │    │                │    │        │
    │    ▼                ▼    │        │
    │  ┌──────┐       ┌──────┐ │        │
    │  │Email │       │Email │ │        │
    │  │Doctor│       │Patient      │ ◄──────┐
    │  │Node  │       │Node  │ │         │ Gmail API
    │  └──────┘       └──────┘ │        │
    │                          │        │
    └──────────────────────────┘        │
         │               │              │
         └───────┬───────┘              │
                 │                      │
                 ▼                      │
            ┌────────────┐              │
            │   Gmail    │──────────────┘
            │   Service  │
            └────┬───────┘
                 │
        ┌────────┴─────────┐
        │                  │
        ▼                  ▼
    ┌────────┐         ┌────────┐
    │ Doctor │         │Patient │
    │ Email  │         │ Email  │
    └────────┘         └────────┘
```

---

## Decision Points & Error Handling

```
         ┌─────────────┐
         │ START       │
         └──────┬──────┘
                │
                ▼
         ┌──────────────────┐
         │ Receive Report?  │
         └──────┬───────────┘
                │
         ┌──────┴──────┐
         │ YES    NO   │
         ▼             ▼
    PROCESS      REJECT
         │        (400)
         ▼
    ┌──────────────────┐
    │ Validate Fields? │
    └──────┬───────────┘
           │
      ┌────┴────┐
      │Y    NO  │
      ▼         ▼
   PROCESS   ERROR
      │      (400)
      ▼
    ┌──────────────────┐
    │ Call Gemini API? │
    └──────┬───────────┘
           │
      ┌────┴────────┐
      │Y      NO    │
      ▼             ▼
   ANALYZE      TIMEOUT
      │          (retry)
      ▼
    ┌──────────────────┐
    │ Parse Success?   │
    └──────┬───────────┘
           │
      ┌────┴────┐
      │Y    NO  │
      ▼         ▼
   CONTINUE  FALLBACK
      │        JSON
      ▼
    ┌──────────────────┐
    │ Send Emails?     │
    └──────┬───────────┘
           │
      ┌────┴────┐
      │Y    NO  │
      ▼         ▼
   SUCCESS    QUEUE
      │      (retry)
      │
      ▼
    ┌─────────┐
    │ END     │
    └─────────┘
```

---

## Data Transformation Flow

```
STAGE 1: INPUT
┌────────────────────────────────────┐
│ {                                  │
│   "patientName": "John Doe",       │
│   "patientAge": "45",              │
│   "medicalReport": "...",          │
│   "doctorEmail": "...",            │
│   "patientEmail": "..."            │
│ }                                  │
└────────────────────────────────────┘
        │
        │ Webhook receives
        │
        ▼
STAGE 2: ANALYSIS REQUEST
┌────────────────────────────────────┐
│ {                                  │
│   "model": "gemini-1.5-flash",     │
│   "messages": [{                   │
│     "role": "system",              │
│     "content": "You are..."        │
│   }, {                             │
│     "role": "user",                │
│     "content": "Analyze: ..."      │
│   }]                               │
│ }                                  │
└────────────────────────────────────┘
        │
        │ Gemini API processes
        │
        ▼
STAGE 3: ANALYSIS RESPONSE
┌────────────────────────────────────┐
│ {                                  │
│   "keyFindings": [...],            │
│   "medications": [...],            │
│   "allergies": [...],              │
│   "drugInteractions": [...],       │
│   "criticalRisks": [...],          │
│   "abnormalLabs": [...],           │
│   "followUpTests": [...],          │
│   "recommendations": [...],        │
│   "patientSummary": "..."          │
│ }                                  │
└────────────────────────────────────┘
        │
        │ Parse and map
        │
        ▼
STAGE 4: EMAIL PAYLOAD
┌──────────────────────┐  ┌──────────────────────┐
│ DOCTOR EMAIL         │  │ PATIENT EMAIL        │
├──────────────────────┤  ├──────────────────────┤
│ To: doctor@...       │  │ To: patient@...      │
│ Subject: Analysis    │  │ Subject: Summary     │
│ Body: Full HTML      │  │ Body: Simple HTML    │
│ - Critical Risks     │  │ - Health Summary     │
│ - All Findings       │  │ - Medications        │
│ - Labs               │  │ - Next Steps         │
│ - Tests              │  │                      │
│ - Recommendations    │  │                      │
└──────────────────────┘  └──────────────────────┘
        │                          │
        └──────────┬───────────────┘
                   │
                   │ Gmail API sends
                   │
                   ▼
            EMAILS DELIVERED
            ✓ Doctor receives full analysis
            ✓ Patient receives summary
```

---

## Performance Timeline

```
0s          ┌─ Webhook receives request
            │
0.5s        ├─ Validation complete
            │
1s          ├─ Gemini API call initiated
            │   ├─ System message sent: "You are a medical expert..."
            │   ├─ Report text sent: "Chief complaint: Chest pain..."
            │   └─ Awaiting AI analysis
            │
5s          ├─ Still waiting for Gemini...
            │
10s         ├─ Still analyzing (normal for complex reports)
            │
15-20s      ├─ Gemini response received
            │   └─ Returns JSON with medical insights
            │
20.5s       ├─ Parse/map complete
            │
21s         ├─ Gmail requests queued
            │   ├─ Doctor email queued
            │   └─ Patient email queued
            │
25-30s      ├─ Emails sent successfully
            │
30s         └─ WORKFLOW COMPLETE ✓

            Timeline by step:
            ├─ Webhook: 0.5s (1-2%)
            ├─ Gemini: 15-20s (50-65%)
            ├─ Parse: 0.5s (1-2%)
            └─ Email: 5-10s (15-30%)
```

---

## Scaling Diagram (Multiple Reports)

```
Single Report (Current):
┌──────┐    ┌────────┐    ┌──────┐    ┌──────┐
│Report│ -> │Analyze │ -> │Parse │ -> │Email │ = 30 seconds
└──────┘    └────────┘    └──────┘    └──────┘


Multiple Reports (Sequential):
Report 1 ──┐
Report 2 ──┤─── [WORKFLOW] ──┬─── Email 1
Report 3 ──┤─── [WORKFLOW] ──┼─── Email 2
Report 4 ──┤─── [WORKFLOW] ──┼─── Email 3
Report 5 ──┘─── [WORKFLOW] ──┴─── Email 4
           (All sequential, ~150 seconds for 5 reports)


Multiple Reports (Parallel - with multiple workers):
Report 1 ──┬─── [WORKER 1] ──┬─── Email 1
Report 2 ──┼─── [WORKER 2] ──┼─── Email 2
Report 3 ──┼─── [WORKER 3] ──┼─── Email 3
Report 4 ──┼─── [WORKER 4] ──┼─── Email 4
Report 5 ──┴─── [WORKER 5] ──┴─── Email 5
           (All parallel, ~30 seconds for 5 reports)
```

---

## Security & Data Flow

```
INBOUND:
Doctor/System ──HTTPS──> Webhook (encrypted in transit)
               ──POST──> n8n (validated input)
                         │
                         ├─ Check for malicious content
                         ├─ Validate email format
                         └─ Sanitize report text

PROCESSING:
n8n (memory) ──HTTPS──> Google Gemini API (API key auth)
             ──TLS 1.2─> Encrypted connection
                        │
                        └─ Prompt injection checks

OUTBOUND:
n8n (memory) ──HTTPS──> Gmail API (OAuth2)
             ──TLS 1.2─> Encrypted connection
                        │
                        ├─ Doctor email (full analysis)
                        └─ Patient email (summary)

AUDIT:
All workflow executions logged in n8n
├─ Input data
├─ Output data
├─ Execution time
├─ Error messages
└─ User/system actions
```

---

## Integration Points

```
                    ┌─────────────────────┐
                    │   CLINICAI CORE     │
                    │   (n8n Workflow)    │
                    └────────┬────────────┘
                             │
            ┌────────────────┼────────────────┐
            │                │                │
            ▼                ▼                ▼
    ┌──────────────┐  ┌──────────────┐  ┌──────────┐
    │   EHR         │  │  Patient     │  │ Google   │
    │   Systems     │  │  Portals     │  │ API      │
    │               │  │              │  │          │
    │ • Epic        │  │ • MyChart    │  │ • Gemini │
    │ • Cerner      │  │ • Patient    │  │ • Gmail  │
    │ • HL7/FHIR    │  │   Sky        │  │          │
    │ • REST API    │  │ • Custom     │  │          │
    └──────────────┘  └──────────────┘  └──────────┘
```

---

**Last Updated:** June 2026 | **Version:** 1.0.0
