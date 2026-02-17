# GA4 Server-Side Tagging Migration Guide (Step-by-Step with Cloud Run Example)

Practical, implementation-focused guide for engineers and analytics teams migrating to Google Analytics 4 (GA4) with Server-Side Tagging using Google Tag Manager and Cloud Run.

This implementation pattern is widely adopted for privacy-focused analytics and first-party tracking setups.

---

## Table of Contents

1. Overview & Benefits  
2. Architecture (High-Level)  
3. Prerequisites  
4. Quick Start (3 Minutes)  
5. Full Step-by-Step  
6. Testing & Validation  
7. Privacy & Best Practices  
8. Troubleshooting  
9. Repository Structure  
10. Contributing & License  
11. Author  

---

## 1) Overview & Benefits

Server-side tagging moves part of your tracking pipeline from the user’s browser into a server you control.

### Benefits

- Higher data quality (reduced ad-blocker loss)
- Improved site performance
- Greater control over data forwarded to third parties
- Easier enrichment with CRM or backend data
- Better privacy and consent enforcement

---

## 2) Architecture (High-Level)

```
Browser (gtag.js or Web GTM)
        ↓
GTM Server Container (Cloud Run / custom server)
        ↓
GA4 (via Measurement Protocol)
        ↓
Optional: Meta CAPI / Other vendors
```

Client-side requests are routed to your first-party server endpoint.  
The server container receives, processes, enriches, and forwards events securely.

---

## 3) Prerequisites

- GA4 property created
- Google Tag Manager account (Server container enabled)
- Google Cloud Project with billing enabled
- gcloud CLI installed (if using Cloud Run)
- First-party domain recommended (e.g., `tagging.yourdomain.com`)

---

## 4) Quick Start (3-Minute Setup)

1. Create GA4 property → note `G-XXXXXXX`
2. Create API Secret in GA4:
   - Admin → Data Streams → Measurement Protocol API secrets
3. Create a GTM Server container
4. Deploy the GTM Server container to Cloud Run
5. Set `transport_url` in your GA4 config
6. Test using Realtime reports

---

## 5) Full Step-by-Step

### A — Create GA4 Property & API Secret

1. Admin → Data Streams → Select Web Stream
2. Create Measurement Protocol API Secret
3. Note:
   - `measurement_id`
   - `api_secret`

Server POSTs require both.

---

### B — Create GTM Server Container

1. Tag Manager → Create Container
2. Choose **Server**
3. Add built-in **GA4 Client**

---

### C — Deploy to Cloud Run

Example deployment:

```bash
PROJECT_ID="your-gcp-project-id"
REGION="us-central1"
SERVICE_NAME="gtm-server"

gcloud config set project $PROJECT_ID

gcloud run deploy $SERVICE_NAME \
  --image=gcr.io/cloud-tagging-10302018/gtm-cloud-image:stable \
  --region=$REGION \
  --platform=managed \
  --allow-unauthenticated \
  --memory=512Mi \
  --port=8080
```

After deployment, note the Cloud Run URL and add it in:

GTM Server Container → Admin → Container URL

For production, map a first-party domain like:

```
tagging.yourdomain.com
```

---

### D — Update Client (transport_url)

If using `gtag.js`:

```html
<script async src="https://www.googletagmanager.com/gtag/js?id=G-XXXXXXX"></script>
<script>
window.dataLayer = window.dataLayer || [];
function gtag(){dataLayer.push(arguments);}
gtag('js', new Date());

gtag('config', 'G-XXXXXXX', {
  'transport_url': 'https://tagging.yourdomain.com',
  'first_party_collection': true
});
</script>
```

If using Web GTM, add `transport_url` inside the GA4 configuration tag.

⚠ Important: Disable any old direct-to-Google GA4 tags to prevent duplicates.

---

## 6) Testing & Validation

- Use GTM Server Preview mode
- Check GA4 Realtime
- Validate:
  - No duplicate events
  - Consent respected
  - Correct parameters (`client_id`, `user_id`)

---

## 7) Privacy & Best Practices

- Respect CMP consent signals
- Do not send PII to GA4
- Monitor Cloud Run usage & costs
- Use first-party domains for better reliability

---

## 8) Troubleshooting

**No events in GA4**
- Verify `measurement_id`
- Verify `api_secret`
- Confirm server tag is firing

**Duplicate events**
- Disable old GA4 web tags

**Missing parameters**
- Ensure `client_id` or `user_id` is included

---

## 9) Repository Structure

```
ga4-server-side-migration/
├── README.md
├── LICENSE
├── examples/
│   ├── curl-test.sh
│   ├── gcloud-deploy.sh
│   └── node-proxy/
│       ├── package.json
│       └── index.js
└── assets/
    └── diagram.png (optional)
```

---

## 10) Contributing & License

- Contributions welcome via Pull Requests
- License: MIT

---

## 11) Author

Zeta Labs (ZetaLabsDXB)  
https://www.zetalabsdxb.com/
