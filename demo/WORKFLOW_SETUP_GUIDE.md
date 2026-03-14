# 🧾 AI Receipt Parser - Setup Guide

## Workflow Overview

```
┌──────────────┐    ┌──────────────┐    ┌──────────────┐    ┌──────────────┐
│ 📥 Watch     │───▶│ 📄 Download  │───▶│ 🤖 AI Parse  │───▶│ ⚙️ Clean     │
│ Receipts     │    │ Image        │    │ (Gemini)     │    │ JSON         │
│ Folder       │    │              │    │              │    │              │
└──────────────┘    └──────────────┘    └──────────────┘    └──────────────┘
                                                                   │
                    ┌──────────────┐    ┌──────────────┐           │
                    │ 💰 High      │◀───│ 📁 Move to   │◀──────────┘
                    │ Value Check  │    │ Processed    │
                    └──────────────┘    └──────────────┘
                           │                   │
                    ┌──────┴──────┐           │
                    ▼             ▼           │
              ┌──────────┐  ┌──────────┐     │
              │ 🔔 Alert │  │ ✅ Done  │◀────┘
              │ (>$200)  │  │          │
              └──────────┘  └──────────┘
```

---

## Step 1: Import the Workflow

1. Open n8n: https://n8n.meerkat-cirius.ts.net
2. Click **"Add workflow"** (+ button)
3. Click the **three dots menu** (⋮) → **Import from file**
4. Select: `/home/bifrostoverseer/n8n-helm/demo/receipt_parser_workflow.json`
5. Click **Import**

---

## Step 2: Get Gemini API Key (Free)

1. Go to: https://aistudio.google.com/apikey
2. Click **"Create API Key"**
3. Copy the key (starts with `AIza...`)

### Add Credential in n8n:

1. Go to **Credentials** in n8n sidebar
2. Click **Add Credential**
3. Search for **"HTTP Query Auth"**
4. Configure:
   - **Name**: `Gemini API`
   - **Parameter Name**: `key`
   - **Value**: `YOUR_API_KEY_HERE`
5. Click **Save**

---

## Step 3: Connect Google Drive

1. In n8n, go to **Credentials**
2. Click **Add Credential**
3. Search for **"Google Drive OAuth2"**
4. Click **Sign in with Google**
5. Grant permissions to:
   - View and manage Google Drive files
   - View and manage Google Sheets

---

## Step 4: Create Google Sheet

### Option A: Create Manually

1. Go to [Google Sheets](https://sheets.google.com)
2. Create new spreadsheet named: **"Expense Tracker"**
3. Add these headers in Row 1:

| A | B | C | D | E | F | G | H | I | J | K | L | M | N |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| Date | Vendor | Category | Description | Items | Subtotal | Tax | Tip | Total | Payment Method | Transaction ID | Receipt Link | Processed At | Confidence |

4. Get the **Spreadsheet ID** from the URL:
   ```
   https://docs.google.com/spreadsheets/d/SPREADSHEET_ID_HERE/edit
   ```

### Option B: Copy Template (Faster)

Copy this template: [Expense Tracker Template](https://docs.google.com/spreadsheets/d/1example/copy)

---

## Step 5: Update Workflow Configuration

Open the workflow in n8n and update these nodes:

### Node: "📥 Watch Receipts Folder"
- **Folder ID**: `1bUOqMH05kr16vuC1og900YWmmxIB00u_` ✅ (Already set)
- Connect your **Google Drive credential**

### Node: "📄 Download Receipt Image"  
- Connect your **Google Drive credential**

### Node: "🤖 AI Parse Receipt (Gemini Vision)"
- Connect your **Gemini API credential** (HTTP Query Auth)

### Node: "📊 Add to Expense Tracker"
- **Spreadsheet ID**: Replace `REPLACE_WITH_SPREADSHEET_ID`
- Connect your **Google Sheets credential**

### Node: "📁 Move to Processed Folder"
- **Folder ID**: Replace `REPLACE_WITH_PROCESSED_FOLDER_ID`
- Connect your **Google Drive credential**

---

## Step 6: Test the Workflow

1. **Activate** the workflow (toggle in top-right)
2. Upload a receipt image to your Google Drive folder
3. Wait ~1 minute for the trigger
4. Check your Google Sheet - data should appear!

### Manual Test:
1. Click on "📥 Watch Receipts Folder" node
2. Click **"Fetch Test Event"**
3. If you have a file in the folder, it will load
4. Click **"Test Workflow"** to run through all nodes

---

## Node Documentation Summary

| Node | Purpose | Credentials Needed |
|------|---------|-------------------|
| 📥 Watch Receipts Folder | Monitors Google Drive for new receipts | Google Drive |
| 📄 Download Receipt Image | Downloads the image file | Google Drive |
| 🤖 AI Parse Receipt | Sends image to Gemini for parsing | HTTP Query Auth (Gemini) |
| ⚙️ Parse & Clean JSON | Extracts and formats the AI response | None |
| 📊 Add to Expense Tracker | Appends row to Google Sheet | Google Sheets |
| 📁 Move to Processed | Archives the receipt | Google Drive |
| 💰 High Value Check | Checks if total > $200 | None |
| 🔔 Send Alert | Optional Slack notification | Slack (disabled) |
| ✅ Done | Workflow endpoint | None |

---

## Expected Output

After processing a receipt, your Google Sheet will have:

| Date | Vendor | Category | Description | Total | ... |
|------|--------|----------|-------------|-------|-----|
| 2026-01-15 | Staples | Office Supplies | Printer paper, pens, folders | 103.32 | ... |
| 2026-01-18 | Uber | Travel | Airport trip | 40.25 | ... |
| 2026-01-20 | The Golden Fork | Meals & Entertainment | Business lunch, 4 guests | 208.41 | ... |

---

## Troubleshooting

### "No data from trigger"
- Ensure file is in the correct folder
- Check folder ID matches
- Verify Google Drive credential has access

### "Gemini API error"
- Verify API key is correct
- Check you haven't exceeded free tier limits
- Ensure image is valid (PNG/JPG/PDF)

### "Failed to parse JSON"
- Check Gemini response in node output
- Image may be too blurry or low quality
- Try a clearer receipt image

### "Sheet not updating"
- Verify spreadsheet ID
- Check column headers match exactly
- Ensure credential has edit access

---

## Customization Ideas

1. **Change categories**: Edit the prompt in the Gemini node
2. **Add approval workflow**: Insert a "Wait" node before Sheet
3. **Email receipts**: Add Gmail trigger alongside Drive trigger
4. **Multiple currencies**: Modify the Code node to handle conversion
5. **Department tracking**: Add a dropdown column in Sheet

---

## Cost Estimate

| Service | Free Tier | Your Usage | Monthly Cost |
|---------|-----------|------------|--------------|
| Gemini 2.0 Flash | 1,500 requests/day | ~150 receipts/mo | **$0** |
| Google Drive | 15 GB | Minimal | **$0** |
| Google Sheets | Unlimited | Minimal | **$0** |
| n8n (self-hosted) | Unlimited | Already running | **$0** |
| **Total** | | | **$0/month** |

---

## Demo Script

### For showing your father:

1. **Show empty sheet**: "Here's our expense tracker - currently empty"

2. **Drop receipt**: "Now I'll just drop this receipt into Google Drive..."

3. **Wait 30-60 seconds**: "The AI is reading the receipt..."

4. **Refresh sheet**: "And look - all the data is extracted automatically!"

5. **Show details**: "It got the vendor, date, total, even categorized it correctly"

6. **ROI pitch**: "This normally takes 3-5 minutes per receipt. With 20 receipts a week, that's over an hour saved. The AI does it in seconds, for free."
