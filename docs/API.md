# API Reference

Complete API documentation for ClinicAI webhook endpoint.

---

## Base URL

```
Test: https://your-instance.n8n.cloud/webhook-test/medical-analyzer
Production: https://your-instance.n8n.cloud/webhook/medical-analyzer
```

---

## Endpoints

### Submit Medical Report for Analysis

```
POST /webhook/medical-analyzer
```

#### Request Headers

```
Content-Type: application/json
```

#### Request Body

| Field | Type | Required | Max Length | Description |
|-------|------|----------|-----------|-------------|
| `patientName` | string | ✅ | 255 | Patient's full name |
| `patientAge` | string | ✅ | 3 | Patient's age (years) |
| `medicalReport` | string | ✅ | 50,000 | Full medical report text (no formatting required) |
| `doctorEmail` | string | ✅ | 255 | Doctor/clinician email address |
| `patientEmail` | string | ✅ | 255 | Patient email address |

#### Request Example

```bash
curl -X POST https://your-instance.n8n.cloud/webhook/medical-analyzer \
  -H "Content-Type: application/json" \
  -d '{
    "patientName": "John Doe",
    "patientAge": "45",
    "medicalReport": "Chief Complaint: Chest pain. Symptoms: Severe chest pain for 3 hours, shortness of breath...",
    "doctorEmail": "doctor@hospital.com",
    "patientEmail": "john.doe@email.com"
  }'
```

#### Response

**Success Response:**

```
HTTP 200 OK
{
  "success": true
}
```

**Error Response:**

```
HTTP 400 Bad Request
{
  "error": "Missing required field: patientName"
}
```

---

## Response Codes

| Code | Status | Description |
|------|--------|-------------|
| 200 | OK | Report accepted and processing started |
| 400 | Bad Request | Missing or invalid required fields |
| 401 | Unauthorized | Invalid authentication token (if enabled) |
| 429 | Too Many Requests | Rate limit exceeded |
| 500 | Server Error | Internal processing error |
| 503 | Service Unavailable | n8n service temporarily down |

---

## Data Validation

### Patient Name
- **Type:** String
- **Required:** Yes
- **Pattern:** `^[a-zA-Z\s\-']+$` (letters, spaces, hyphens, apostrophes)
- **Examples:**
  - ✅ "John Doe"
  - ✅ "Mary-Jane Smith"
  - ✅ "Jean-Claude O'Brien"
  - ❌ "John123" (numbers not allowed)

### Patient Age
- **Type:** String or Number
- **Required:** Yes
- **Range:** 0-150 years
- **Examples:**
  - ✅ "45"
  - ✅ 45 (numeric)
  - ❌ "-5" (negative not allowed)
  - ❌ "200" (too high)

### Medical Report
- **Type:** String
- **Required:** Yes
- **Min Length:** 50 characters
- **Max Length:** 50,000 characters
- **Format:** Plain text (no HTML/XML required)
- **Examples:**
  - ✅ Structured clinical notes
  - ✅ Unstructured narrative text
  - ✅ Copied from EHR
  - ✅ Handwritten notes (OCR'd to text)

### Email Addresses
- **Type:** String
- **Required:** Yes
- **Pattern:** RFC 5322 standard email format
- **Examples:**
  - ✅ "john.doe@hospital.com"
  - ✅ "dr.smith+clinic@example.com"
  - ❌ "john@" (missing domain)
  - ❌ "john.doe@" (incomplete domain)

---

## Request/Response Examples

### Example 1: Simple Consultation Note

```bash
curl -X POST https://your-instance.n8n.cloud/webhook/medical-analyzer \
  -H "Content-Type: application/json" \
  -d '{
    "patientName": "Jane Smith",
    "patientAge": "38",
    "medicalReport": "Annual physical. Patient reports feeling well. BP: 120/80. HR: 72. All vitals normal. Exam: unremarkable. No acute complaints. Recommend continued healthy lifestyle.",
    "doctorEmail": "dr.wilson@clinic.com",
    "patientEmail": "jane.smith@gmail.com"
  }'
```

### Example 2: Complex Hospital Admission

```bash
curl -X POST https://your-instance.n8n.cloud/webhook/medical-analyzer \
  -H "Content-Type: application/json" \
  -d '{
    "patientName": "Robert Johnson",
    "patientAge": "72",
    "medicalReport": "Admitted to ICU with acute myocardial infarction. Troponin elevated to 3.2. EKG shows ST elevation in inferior leads. Started on dual antiplatelet therapy and heparin. Allergies: Penicillin (anaphylaxis). Current meds: Metoprolol, Lisinopril, Aspirin, Atorvastatin. HR: 92, BP: 145/92, O2 sat 94%. Critical condition - recommend cardiology consult.",
    "doctorEmail": "dr.cardio@hospital.edu",
    "patientEmail": "robert.j.johnson@yahoo.com"
  }'
```

### Example 3: Using Python Requests

```python
import requests
import json

webhook_url = "https://your-instance.n8n.cloud/webhook/medical-analyzer"

payload = {
    "patientName": "Alice Cooper",
    "patientAge": "55",
    "medicalReport": "Patient presents with chronic back pain. MRI shows disc bulge at L4-L5. Referred to physical therapy. Continue current medications.",
    "doctorEmail": "dr.spine@clinic.com",
    "patientEmail": "alice.cooper@outlook.com"
}

response = requests.post(webhook_url, json=payload)

if response.status_code == 200:
    print("✅ Report submitted successfully")
else:
    print(f"❌ Error: {response.status_code}")
    print(response.json())
```

### Example 4: Using JavaScript/Node.js

```javascript
const fetch = require('node-fetch');

const webhookUrl = 'https://your-instance.n8n.cloud/webhook/medical-analyzer';

const payload = {
  patientName: 'David Lee',
  patientAge: '68',
  medicalReport: 'Diabetic patient with HbA1c 8.2%. Blood pressure elevated at 155/95. Recommend adjusting medications.',
  doctorEmail: 'dr.lee@endocrine.com',
  patientEmail: 'david.lee@mail.com'
};

fetch(webhookUrl, {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify(payload)
})
  .then(res => res.json())
  .then(data => console.log('✅ Success:', data))
  .catch(err => console.error('❌ Error:', err));
```

---

## Rate Limiting

### Limits

- **Free Plan:** 100 requests/day
- **Professional:** 10,000 requests/day
- **Enterprise:** Unlimited

### Headers

```
X-RateLimit-Limit: 10000
X-RateLimit-Remaining: 9995
X-RateLimit-Reset: 1687881600
```

### Handling Rate Limits

```python
import requests
import time

def submit_with_retry(payload, max_retries=3):
    for attempt in range(max_retries):
        response = requests.post(webhook_url, json=payload)
        
        if response.status_code == 200:
            return response.json()
        elif response.status_code == 429:
            wait_time = 2 ** attempt  # Exponential backoff
            print(f"Rate limited. Waiting {wait_time}s...")
            time.sleep(wait_time)
        else:
            raise Exception(f"Error: {response.status_code}")
```

---

## Error Handling

### Common Errors

#### Missing Required Field
```
HTTP 400 Bad Request
{
  "error": "Missing required field: doctorEmail"
}
```

**Solution:** Ensure all 5 fields are present in request body.

#### Invalid Email Format
```
HTTP 400 Bad Request
{
  "error": "Invalid email format: doctorEmail"
}
```

**Solution:** Verify email addresses are in valid format (user@domain.com).

#### Report Text Too Short
```
HTTP 400 Bad Request
{
  "error": "Medical report must be at least 50 characters"
}
```

**Solution:** Provide more detailed medical information.

#### API Key Invalid
```
HTTP 401 Unauthorized
{
  "error": "Invalid or expired API key"
}
```

**Solution:** Check your Google Gemini API key is correct.

#### Service Unavailable
```
HTTP 503 Service Unavailable
{
  "error": "Service temporarily unavailable"
}
```

**Solution:** Retry after 5 minutes. Check n8n cloud status page.

---

## Processing Time SLA

| Scenario | Expected Time | Max Time |
|----------|---------------|----------|
| Normal workload | 25-30 seconds | 45 seconds |
| High load (100+ concurrent) | 30-40 seconds | 60 seconds |
| Peak hours | 40-50 seconds | 90 seconds |

---

## Webhook Signature (Optional Security)

If webhook signature verification is enabled:

```
POST /webhook/medical-analyzer
X-Signature: sha256=...
```

**Verification:**
```python
import hmac
import hashlib

secret = "your-webhook-secret"
body = request.body
signature = hmac.new(
    secret.encode(),
    body,
    hashlib.sha256
).hexdigest()

# Verify signature matches X-Signature header
```

---

## Batch Processing

For processing multiple reports:

```python
import requests
import time

def batch_submit(reports, delay=2):
    """Submit multiple reports with rate limiting"""
    
    webhook_url = "https://your-instance.n8n.cloud/webhook/medical-analyzer"
    
    for i, report in enumerate(reports):
        response = requests.post(webhook_url, json=report)
        
        if response.status_code == 200:
            print(f"✅ Report {i+1}/{len(reports)} submitted")
        else:
            print(f"❌ Report {i+1} failed: {response.status_code}")
        
        if i < len(reports) - 1:
            time.sleep(delay)

# Usage
reports = [
    {...patient1...},
    {...patient2...},
    {...patient3...}
]

batch_submit(reports, delay=2)
```

---

## Webhook Logs

Access webhook execution logs:

1. Log in to n8n dashboard
2. Click **"Executions"** tab
3. Filter by date/time
4. Click execution to see:
   - Input data
   - Output data (analysis results)
   - Error messages
   - Execution time

---

## Testing

### Using curl

```bash
# Test endpoint availability
curl -i https://your-instance.n8n.cloud/webhook-test/medical-analyzer

# Submit test report
curl -X POST https://your-instance.n8n.cloud/webhook-test/medical-analyzer \
  -H "Content-Type: application/json" \
  -d @sample-medical-report.json
```

### Using Postman

1. Create new POST request
2. URL: `https://your-instance.n8n.cloud/webhook/medical-analyzer`
3. Headers: `Content-Type: application/json`
4. Body: Paste JSON payload (raw)
5. Click **Send**

### Health Check

```bash
curl -s https://your-instance.n8n.cloud/api/v1/health
```

Expected response:
```json
{
  "status": "ok",
  "version": "1.0.0"
}
```

---

## Support

For API issues:
- Check error message and response code above
- Review [SETUP.md](SETUP.md) for configuration help
- Check n8n logs via dashboard
- Contact support if issues persist

---

**Last Updated:** June 2026 | **Version:** 1.0.0
