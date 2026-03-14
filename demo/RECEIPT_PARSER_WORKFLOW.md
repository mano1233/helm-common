# 🧾 AI Receipt Parser Workflow

This guide walks you through building an AI-powered receipt parser in n8n that:
1. Monitors a Google Drive folder for new receipt images
2. Uses AI vision to extract data from receipts
3. Automatically adds parsed data to a Google Sheet

## Prerequisites

- n8n instance running (you have this!)
- Google account with Drive and Sheets access
- Gemini API key (free tier works great)

---

## Step 1: Set Up Google Drive

1. **Create a folder** in Google Drive called `Receipts-Inbox`
2. **Create a second folder** called `Receipts-Processed` (for archiving)
3. Note the folder IDs (from the URL when inside each folder)

---

## Step 2: Create the Google Sheet

Create a new Google Sheet called `Expense Tracker` with these columns:

| Column | Description |
|--------|-------------|
| A: Date | Transaction date (YYYY-MM-DD) |
| B: Vendor | Store/company name |
| C: Category | Auto-categorized expense type |
| D: Description | Brief description of items |
| E: Subtotal | Pre-tax amount |
| F: Tax | Tax amount |
| G: Total | Final total |
| H: Payment Method | Card type and last 4 digits |
| I: Receipt File | Link to original receipt |
| J: Processed At | Timestamp when AI parsed it |
| K: Confidence | AI confidence score |

**Header row (copy this):**
```
Date | Vendor | Category | Description | Subtotal | Tax | Total | Payment Method | Receipt File | Processed At | Confidence
```

---

## Step 3: Build the n8n Workflow

### Node 1: Google Drive Trigger

1. Add node: **Google Drive Trigger**
2. Configure:
   - **Trigger On**: File Created
   - **Folder**: Select your `Receipts-Inbox` folder
   - **Poll Time**: Every 1 minute (for demo) or 5 minutes (production)

### Node 2: Download File

1. Add node: **Google Drive** (action node)
2. Configure:
   - **Operation**: Download
   - **File ID**: `{{ $json.id }}`
   - **Options**: Binary Property Name = `receipt`

### Node 3: AI Vision Parser (Gemini)

1. Add node: **HTTP Request** (or Gemini node if available)
2. Configure for Gemini Vision API:

**URL:** `https://generativelanguage.googleapis.com/v1beta/models/gemini-2.0-flash:generateContent`

**Method:** POST

**Headers:**
```
Content-Type: application/json
```

**Query Parameters:**
```
key: YOUR_GEMINI_API_KEY
```

**Body (JSON):**
```json
{
  "contents": [
    {
      "parts": [
        {
          "text": "You are an expert accountant assistant. Analyze this receipt image and extract the following information in JSON format:\n\n{\n  \"date\": \"YYYY-MM-DD format\",\n  \"vendor\": \"Store or company name\",\n  \"category\": \"One of: Office Supplies, Travel, Meals & Entertainment, Tech & Equipment, Vehicle & Fuel, Utilities, Software & Subscriptions, Professional Services, Other\",\n  \"description\": \"Brief 1-line summary of main items purchased\",\n  \"subtotal\": numeric value without currency symbol,\n  \"tax\": numeric value without currency symbol,\n  \"total\": numeric value without currency symbol,\n  \"payment_method\": \"Card type and last 4 digits (e.g., Visa ***4892)\",\n  \"confidence\": \"high/medium/low based on image clarity\"\n}\n\nIMPORTANT:\n- Return ONLY valid JSON, no markdown or explanation\n- If a field cannot be determined, use null\n- For dates, convert any format to YYYY-MM-DD\n- For amounts, extract numeric values only"
        },
        {
          "inline_data": {
            "mime_type": "image/png",
            "data": "{{ $binary.receipt.data }}"
          }
        }
      ]
    }
  ],
  "generationConfig": {
    "temperature": 0.1,
    "topP": 0.8,
    "maxOutputTokens": 1024
  }
}
```

### Node 4: Parse JSON Response

1. Add node: **Code** (JavaScript)
2. Code:
```javascript
// Extract the JSON from Gemini's response
const response = $input.first().json;
const text = response.candidates[0].content.parts[0].text;

// Parse the JSON (handle markdown code blocks if present)
let jsonStr = text;
if (text.includes('```json')) {
  jsonStr = text.split('```json')[1].split('```')[0].trim();
} else if (text.includes('```')) {
  jsonStr = text.split('```')[1].split('```')[0].trim();
}

const parsed = JSON.parse(jsonStr);

// Add metadata
parsed.receipt_file = $('Google Drive Trigger').first().json.webViewLink;
parsed.processed_at = new Date().toISOString();

return [{ json: parsed }];
```

### Node 5: Add to Google Sheet

1. Add node: **Google Sheets**
2. Configure:
   - **Operation**: Append Row
   - **Document**: Select your `Expense Tracker` sheet
   - **Sheet**: Sheet1
   - **Mapping Mode**: Map Each Column
   
**Column Mapping:**
| Sheet Column | Value |
|--------------|-------|
| Date | `{{ $json.date }}` |
| Vendor | `{{ $json.vendor }}` |
| Category | `{{ $json.category }}` |
| Description | `{{ $json.description }}` |
| Subtotal | `{{ $json.subtotal }}` |
| Tax | `{{ $json.tax }}` |
| Total | `{{ $json.total }}` |
| Payment Method | `{{ $json.payment_method }}` |
| Receipt File | `{{ $json.receipt_file }}` |
| Processed At | `{{ $json.processed_at }}` |
| Confidence | `{{ $json.confidence }}` |

### Node 6: Move to Processed Folder

1. Add node: **Google Drive**
2. Configure:
   - **Operation**: Move
   - **File ID**: `{{ $('Google Drive Trigger').first().json.id }}`
   - **Folder**: Select `Receipts-Processed` folder

### Node 7: (Optional) Send Notification

1. Add node: **Slack** or **Email**
2. Send a summary: "✅ Receipt from {{ $json.vendor }} for {{ $json.total }} processed and categorized as {{ $json.category }}"

---

## Workflow Diagram

```
┌──────────────────┐     ┌──────────────────┐     ┌──────────────────┐
│  Google Drive    │────▶│  Download File   │────▶│  Gemini Vision   │
│  Trigger (new    │     │  (get binary)    │     │  (parse receipt) │
│  file in Inbox)  │     │                  │     │                  │
└──────────────────┘     └──────────────────┘     └──────────────────┘
                                                           │
                                                           ▼
┌──────────────────┐     ┌──────────────────┐     ┌──────────────────┐
│  Send Slack/     │◀────│  Move to         │◀────│  Add Row to      │
│  Email Alert     │     │  Processed       │     │  Google Sheet    │
│  (optional)      │     │  Folder          │     │                  │
└──────────────────┘     └──────────────────┘     └──────────────────┘
```

---

## Testing the Workflow

1. **Activate** the workflow in n8n
2. **Open** `/home/bifrostoverseer/n8n-helm/demo/receipts/index.html` in browser
3. **Screenshot** one of the receipts
4. **Upload** the screenshot to your `Receipts-Inbox` folder in Google Drive
5. **Watch** the magic happen! Check your Google Sheet in ~1 minute

---

## Demo Script for Your Father

### Opening (30 seconds)
> "Let me show you how businesses waste hours every week on expense tracking. 
> Someone gets a receipt, they have to manually type in the date, vendor, amount, 
> categorize it, file it... Now watch this."

### Demo (2 minutes)
1. Show the empty Google Sheet
2. Drop a receipt image into Google Drive
3. Wait 30-60 seconds
4. Refresh the Google Sheet - data appears!
5. Drop 2-3 more receipts quickly
6. Show all data populated automatically

### ROI Pitch (1 minute)
> "A bookkeeper spends 10-15 hours a month just on receipt entry.
> At $25/hour, that's $300-400/month. This AI costs about $5/month
> and does it in seconds with 95%+ accuracy. That's $4,000+ saved per year,
> and the employee can focus on actual accounting work instead of data entry."

---

## Troubleshooting

| Issue | Solution |
|-------|----------|
| Trigger not firing | Check folder permissions, verify folder ID |
| Gemini returns error | Check API key, verify image is base64 encoded |
| Wrong data extracted | Try higher quality images, improve prompt |
| Sheet not updating | Check Google Sheets credentials and permissions |

---

## Enhancements (Future)

- [ ] Add duplicate detection (check if receipt already processed)
- [ ] OCR fallback for low-quality images
- [ ] Auto-categorization learning from corrections
- [ ] Weekly expense summary email
- [ ] Integration with QuickBooks/Xero
