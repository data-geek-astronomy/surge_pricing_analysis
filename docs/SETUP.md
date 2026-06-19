# Setup & Deployment Guide

Complete step-by-step instructions to deploy ClinicAI in your environment.

---

## Prerequisites

- ✅ n8n cloud account (free tier: https://n8n.cloud)
- ✅ Google Generative AI API key (free: https://aistudio.google.com/app/apikey)
- ✅ Gmail account for sending emails
- ✅ Basic understanding of webhooks and REST APIs

---

## Step 1: Get API Keys

### 1.1 Google Gemini API Key

1. Go to [Google AI Studio](https://aistudio.google.com/app/apikey)
2. Click **"Create API Key"**
3. Copy the key (keep it safe!)
4. Store it somewhere secure (you'll need it in Step 3)

### 1.2 Gmail Setup

1. Enable Gmail API in Google Cloud Console
2. Create OAuth2 credentials for n8n
3. Grant n8n permission to send emails

*Note: n8n will handle OAuth2 authentication with Gmail*

---

## Step 2: Create n8n Workflow

### 2.1 Import Workflow

1. Log in to your n8n cloud instance
2. Click **"+ New"** → **"Workflow"**
3. Go to **"Import from URL"** or paste the workflow JSON from `workflows/medical-report-analyzer.json`
4. Click **"Import"**

### 2.2 Workflow Structure

Your imported workflow should have these nodes:
```
✓ Webhook Trigger
✓ Gemini Chat Model (LangChain)
✓ Analyze Medical Report (Agent)
✓ Parse Analysis (Set node)
✓ Email Doctor (Gmail)
✓ Email Patient (Gmail)
```

---

## Step 3: Configure Credentials

### 3.1 Google Gemini API

1. Click on **"Analyze Medical Report"** node (the Agent node)
2. Expand **"Model"** section
3. Click **"+ Add credential"** for "Google Generative AI"
4. Paste your API key from Step 1.1
5. Click **"Save"**

### 3.2 Gmail Authentication

1. Click on **"Email Doctor"** node
2. Click **"+ Add credential"** for "Gmail OAuth2 API"
3. Click **"Authenticate with Google"**
4. Log in with your Gmail account
5. Grant n8n permission to send emails
6. Click **"Save"**
7. Repeat for **"Email Patient"** node

---

## Step 4: Configure Webhook

### 4.1 Webhook Trigger Setup

1. Click on **"Webhook Trigger"** node
2. Note the **Test URL** (copy it)
3. Verify the path is set to: `medical-analyzer`
4. HTTP Method: `POST`
5. Click **"Save"**

Your webhook URL will look like:
```
https://your-instance.n8n.cloud/webhook-test/medical-analyzer
```

---

## Step 5: Test the Workflow

### 5.1 Manual Test

1. Click **"Execute step"** on the Webhook Trigger node
2. Or use curl (see Step 5.2)

### 5.2 Curl Test

Run this command in your terminal:

```bash
curl -X POST https://your-instance.n8n.cloud/webhook-test/medical-analyzer \
  -H "Content-Type: application/json" \
  -d '{
    "patientName": "John Doe",
    "patientAge": "45",
    "medicalReport": "Chief Complaint: Chest pain. Symptoms: Severe chest pain for 3 hours, shortness of breath, dizziness. History: Type 2 Diabetes, Hypertension. Medications: Metformin 500mg twice daily, Lisinopril 10mg once daily. Lab Results: Troponin 0.08 (elevated), ECG shows ST depression. Allergies: Penicillin (rash).",
    "doctorEmail": "your-email@gmail.com",
    "patientEmail": "patient@example.com"
  }'
```

### 5.3 Verify Results

1. Check your Gmail inbox for 2 emails:
   - **To doctor:** Full medical analysis with all findings
   - **To patient:** Health summary (simplified language)
2. Both emails should be populated with actual data from the medical report

---

## Step 6: Publish Workflow (Production)

### 6.1 Activate Workflow

Once testing is complete:

1. Click **"Publish"** button (top right)
2. Confirm the workflow status: **"Active"**
3. Your workflow is now live!

### 6.2 Production Webhook URL

Once published, use this URL (without `/test`):
```
https://your-instance.n8n.cloud/webhook/medical-analyzer
```

**Important:** The production webhook requires authentication by default. Configure it in Webhook Trigger settings if needed.

---

## Troubleshooting

### Problem: "API Key Invalid"
**Solution:**
- Verify your Google API key is correct
- Ensure it's for "Generative AI" not just any Google API
- Regenerate the key if needed

### Problem: Emails Not Sending
**Solution:**
- Check Gmail is authenticated (green checkmark on Email nodes)
- Verify recipient email addresses are correct
- Check n8n error logs for detailed error message
- Ensure Gmail account isn't blocking "less secure apps"

### Problem: Gemini Analysis Returning Empty
**Solution:**
- Verify medical report text is in the webhook payload
- Check Gemini model is set to `gemini-1.5-flash`
- Check API key quota isn't exceeded

### Problem: Webhook Not Triggering
**Solution:**
- Verify webhook path matches exactly: `medical-analyzer`
- Check HTTP method is `POST`
- Ensure Content-Type header is `application/json`
- Check n8n cloud instance is running

---

## Performance Tuning

### Optimize for Speed
- Use `gemini-1.5-flash` (faster than pro)
- Reduce email payload by removing unnecessary HTML

### Optimize for Accuracy
- Use `gemini-1.5-pro` (more accurate analysis)
- Provide more context in the system prompt
- Add validation step before sending emails

### Scale for Volume
- Increase n8n worker count (paid plan feature)
- Add caching layer for similar reports
- Implement rate limiting if needed

---

## Production Checklist

Before going live:

- [ ] Google Gemini API key added and tested
- [ ] Gmail authentication working
- [ ] Webhook tested with sample medical report
- [ ] Doctor email verified (test email sent)
- [ ] Patient email verified (test email sent)
- [ ] Error handling configured (optional)
- [ ] Logging/audit enabled (optional)
- [ ] Workflow published (not in test mode)
- [ ] Email templates reviewed for accuracy
- [ ] API documentation updated with your webhook URL

---

## Integration Examples

### Integrate with Your EHR System

Send reports from your EHR to the webhook:

```python
import requests
import json

webhook_url = "https://your-instance.n8n.cloud/webhook/medical-analyzer"

payload = {
    "patientName": "Jane Smith",
    "patientAge": "38",
    "medicalReport": "...", # from your EHR
    "doctorEmail": "doctor@hospital.com",
    "patientEmail": "jane@email.com"
}

response = requests.post(webhook_url, json=payload)
print(f"Status: {response.status_code}")
```

### Queue Multiple Reports

```python
import requests
import json
import time

webhook_url = "https://your-instance.n8n.cloud/webhook/medical-analyzer"
reports = [...]  # list of medical reports

for report in reports:
    payload = {
        "patientName": report['name'],
        "patientAge": report['age'],
        "medicalReport": report['text'],
        "doctorEmail": report['doctor_email'],
        "patientEmail": report['patient_email']
    }
    
    response = requests.post(webhook_url, json=payload)
    print(f"Submitted {report['name']}: {response.status_code}")
    
    time.sleep(2)  # Rate limit: 2 seconds between requests
```

---

## Monitoring & Logging

### View Workflow Executions

1. Go to **"Executions"** tab in n8n
2. See all webhook submissions and their status
3. Click each execution to view:
   - Input data
   - Output data
   - Error messages
   - Execution time

### Enable Audit Logging

1. Go to **Settings** → **Audit**
2. Enable "Log all changes"
3. Configure log retention (compliance requirement)

---

## Security Best Practices

1. **Rotate API keys regularly** (every 90 days)
2. **Use environment variables** for secrets (not hardcoded)
3. **Enable webhook authentication** (add API key requirement)
4. **Monitor API usage** (check for abuse)
5. **Encrypt sensitive fields** in emails
6. **Add IP whitelisting** (if integrated with internal systems)
7. **Enable HIPAA audit logging** (if handling PHI)

---

## Support & Troubleshooting

- **n8n Docs:** https://docs.n8n.io
- **Google Generative AI Docs:** https://ai.google.dev/docs
- **Gmail API Docs:** https://developers.google.com/gmail/api

---

## Next Steps

1. ✅ Deploy the workflow
2. ✅ Test with sample data
3. ✅ Configure for your hospital/clinic
4. ✅ Integrate with your EHR system
5. ✅ Train staff on how to use it
6. ✅ Monitor performance and iterate

---

**Last Updated:** June 2026 | **Version:** 1.0.0
