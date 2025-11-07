# Setup Guide - n8n Job Seeker Automation

Complete step-by-step guide to set up your Job Seeker automation from scratch.

## Table of Contents
1. [Prerequisites](#prerequisites)
2. [n8n Installation](#n8n-installation)
3. [Credential Setup](#credential-setup)
4. [Google Sheets Setup](#google-sheets-setup)
5. [Workflow Import](#workflow-import)
6. [Configuration](#configuration)
7. [Testing](#testing)
8. [Troubleshooting](#troubleshooting)

---

## Prerequisites

Before starting, ensure you have:
- [ ] Computer with internet connection
- [ ] Node.js 16+ installed
- [ ] Google account
- [ ] Credit card (for API services - most have free tiers)

---

## n8n Installation

### Option 1: Desktop App (Recommended for Beginners)
1. Download n8n Desktop from [n8n.io/download](https://n8n.io/download)
2. Install and launch the application
3. n8n will open in your browser at `http://localhost:5678`

### Option 2: npm Installation
```bash
npm install n8n -g
n8n start
```

### Option 3: Docker
```bash
docker run -it --rm \
  --name n8n \
  -p 5678:5678 \
  -v ~/.n8n:/home/node/.n8n \
  n8nio/n8n
```

### Option 4: n8n Cloud
1. Go to [n8n.cloud](https://n8n.cloud)
2. Sign up for an account
3. Create a new workspace

---

## Credential Setup

### 1. Apify API

**Step-by-step:**
1. Go to [apify.com](https://apify.com)
2. Sign up for a free account
3. Navigate to Settings → Integrations
4. Copy your API token
5. In n8n:
   - Click "Credentials" in the left menu
   - Click "Add Credential"
   - Search for "Apify"
   - Paste your API token
   - Name it "Apify account"
   - Click "Save"

**Free Tier:** $5 free credit monthly

### 2. Google Gemini API

**Step-by-step:**
1. Go to [aistudio.google.com](https://aistudio.google.com)
2. Sign in with your Google account
3. Click "Get API Key"
4. Create a new API key
5. Copy the key
6. In n8n:
   - Click "Credentials" in the left menu
   - Click "Add Credential"
   - Search for "Google PaLM API" or "Google AI"
   - Paste your API key
   - Name it "Project -> JobSeeker"
   - Click "Save"

**Free Tier:** 1,500 requests/day for Gemini 1.5 Flash

### 3. Google Sheets OAuth2

**Step-by-step:**
1. Go to [console.cloud.google.com](https://console.cloud.google.com)
2. Create a new project (or select existing)
3. Enable Google Sheets API:
   - Search "Google Sheets API"
   - Click "Enable"
4. Create OAuth credentials:
   - Go to "APIs & Services" → "Credentials"
   - Click "Create Credentials" → "OAuth client ID"
   - Choose "Web application"
   - Add authorized redirect URI: `http://localhost:5678/rest/oauth2-credential/callback`
   - Download JSON
5. In n8n:
   - Click "Credentials" in the left menu
   - Click "Add Credential"
   - Search for "Google Sheets OAuth2 API"
   - Upload the JSON file (or paste Client ID and Secret)
   - Click "Connect my account"
   - Authorize in Google
   - Name it "Google Sheets account"
   - Click "Save"

---

## Google Sheets Setup

### Create Job Database Spreadsheet

1. Go to [sheets.google.com](https://sheets.google.com)
2. Click "+ Blank" to create new spreadsheet
3. Name it "Job Listings Database"
4. Create the following columns (in order):

| Column Name | Description |
|-------------|-------------|
| Title | Job title |
| Platform | Indeed or LinkedIn |
| Date | Date posted |
| Company Name | Company name |
| City | Location city |
| Expired | Job expired status |
| Remote | Remote work availability |
| Link | Job URL (unique identifier) |
| Rating | AI score (0-10) |
| Resume Version | Recommended version (1-4) |
| Cover Letter | Generated cover letter |
| Salary | Salary information |
| Job Description | Full job description |
| Status | Application status (To-Do, Applied, etc.) |

5. Copy the spreadsheet ID from the URL:
   ```
   https://docs.google.com/spreadsheets/d/[THIS_IS_THE_ID]/edit
   ```

---

## Workflow Import

### Import the Workflow

1. Download `Job Seeker.json` from this repository
2. In n8n:
   - Click "Workflows" in the left menu
   - Click "Add Workflow" → "Import from File"
   - Select `Job Seeker.json`
   - Click "Import"

### Configure Workflow Credentials

After importing, you need to assign credentials to nodes:

**Nodes requiring Apify credentials:**
- Run Indeed Actor
- Get Indeed Dataset
- Run Linkedin Actor
- Get Linkedin Dataset

**Nodes requiring Google Gemini credentials:**
- Score and Reason
- Generate Cover Letter

**Nodes requiring Google Sheets credentials:**
- Get All Existing Job URLs
- Append or update job in sheet

**How to assign:**
1. Click on each node
2. In the right panel, find "Credential to connect with"
3. Select the credential you created earlier
4. Click outside the node to save

---

## Configuration

### 1. Update Google Sheets Document ID

**Nodes to update:**
- Get All Existing Job URLs
- Append or update job in sheet

**How:**
1. Click on the node
2. Find "Document ID" field
3. Select "From list" → Click "Refresh"
4. Select "Job Listings Database"
5. Select Sheet: "Sheet1"

### 2. Configure Job Search Parameters

**Indeed (Run Indeed Actor node):**
```json
{
  "country": "ca",              // Change to your country code
  "location": "North York",     // Change to your location
  "maxRows": 50,                // Max jobs per run
  "query": "web developer",     // Change to your job search
  "jobType": "fulltime",        // fulltime, parttime, contract
  "remote": "remote",           // remote, hybrid, onsite
  "fromDays": "1"               // Jobs from last N days
}
```

**LinkedIn (Run Linkedin Actor node):**
```json
{
  "count": 100,                 // Max jobs to scrape
  "scrapeCompany": true,        // Get company details
  "urls": [
    "YOUR_LINKEDIN_SEARCH_URL"  // Update with your search URL
  ]
}
```

**To get LinkedIn search URL:**
1. Go to linkedin.com/jobs
2. Enter your search criteria
3. Copy the URL from browser address bar
4. Paste in the workflow configuration

### 3. Add Your Resume Data

Edit the `JSONify score and reasoning` node:
1. Click on the node
2. Find the `resumeVersions` object in the code
3. Replace the resume data with your own information
4. Keep the structure the same
5. Update all 4 versions

Refer to `Resumes/` folder for sample format.

### 4. Adjust Scoring Criteria (Optional)

Edit `Score and Reason` node to change:
- Salary thresholds
- Scoring weights
- Resume version descriptions

### 5. Customize Cover Letter Style (Optional)

Edit `Generate Cover Letter` node prompt to adjust:
- Tone (formal vs casual)
- Length
- Structure
- Focus areas

---

## Testing

### Test Individual Nodes

1. **Test Indeed Scraping**
   - Click "Run Indeed Actor" node
   - Click "Execute node" (play button at top)
   - Verify you see job data in output

2. **Test LinkedIn Scraping**
   - Click "Run Linkedin Actor" node
   - Click "Execute node"
   - Verify job data appears

3. **Test Google Sheets Connection**
   - Click "Get All Existing Job URLs" node
   - Click "Execute node"
   - Should succeed (even if no data yet)

### Test Full Workflow

1. Click "Execute Workflow" at top right
2. Watch the execution flow
3. Check for errors (red nodes)
4. Verify final output in Google Sheets

### Verify Results

Check your Google Sheets:
- [ ] New rows added
- [ ] Platform column shows "Indeed" or "LinkedIn"
- [ ] Rating column has scores (0-10)
- [ ] Resume Version column has numbers (1-4)
- [ ] Cover Letter column has text for high-scoring jobs
- [ ] No duplicate URLs in Link column

---

## Troubleshooting

### Common Issues

**"Apify authentication failed"**
- Verify API token is correct
- Check Apify account has available credits
- Try regenerating API token

**"Google Gemini error"**
- Verify API key is active
- Check daily quota (1,500 requests)
- Try creating new API key

**"Google Sheets permission denied"**
- Re-authenticate OAuth connection
- Verify spreadsheet is accessible
- Check redirect URI in Google Cloud Console

**"No jobs found"**
- Verify search parameters
- Check Apify actor is running
- Try broader search terms

**"Duplicate jobs appearing"**
- Verify "Link" column in Google Sheets matches job URLs
- Check filter node is using correct node reference
- Clear test data and re-run

**"Cover letters not generating"**
- Check if jobs score > 5
- Verify Gemini API credentials
- Check API quota

**"Loop exits without processing"**
- Ensure Merge node is before deduplication
- Check both actor branches connect to Merge
- Verify loop configuration

### Getting Help

If you encounter issues:
1. Check workflow execution logs
2. Look for error messages in node outputs
3. Review this documentation
4. Check [GitHub Issues](https://github.com/yourusername/n8n-job-seeker/issues)
5. Open a new issue with:
   - Error message
   - Steps to reproduce
   - Screenshots
   - n8n version

---

## Next Steps

After successful setup:
1. ✅ Run workflow manually to test
2. ✅ Verify Google Sheets output
3. ✅ Enable schedule trigger (daily at 9 AM)
4. ✅ Monitor for errors
5. ✅ Adjust search parameters as needed
6. ✅ Customize scoring criteria
7. ✅ Fine-tune cover letter prompts

---

## Estimated Setup Time

- Basic setup: 30-45 minutes
- Full configuration: 1-2 hours
- Resume customization: 1-2 hours

---

## Cost Estimate (Monthly)

- Apify: Free tier ($5 credit)
- Google Gemini: Free tier (1,500 requests/day)
- Google Sheets: Free
- n8n Desktop: Free
- n8n Cloud: $20/month (optional)

**Total: $0-20/month depending on usage and n8n hosting choice**

---

Need help? Open an issue on GitHub!
