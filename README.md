# AI-Powered Job Seeker Automation

Automates job searching on Indeed and LinkedIn using n8n. Scores jobs with AI, generates personalized cover letters, and stores everything in Google Sheets.

![n8n Workflow](docs/images/workflow-overview.png)

## Features

- Scrapes jobs from Indeed and LinkedIn daily using 4 parallel searches (Software Developer + Web Developer on both platforms)
- Removes duplicates and jobs older than 7 days to save on API costs
- AI scores jobs 0-10 based on resume fit, salary, remote work, and benefits
- Recommends which resume version to use (supports 4 versions)
- Auto-generates cover letters for high-scoring jobs (>5/10)
- Stores all data in Google Sheets with application tracking
- Email notifications when new jobs are found

## How It Works

![Workflow Diagram](docs/images/workflow-detailed-1.png)
![Workflow Diagram](docs/images/workflow-detailed-2.png)

```
Schedule Trigger (Daily at 9 AM)
  ├─→ Indeed (Web Dev) → Get Dataset → Add Platform ────────┐
  ├─→ Indeed (Software Dev) → Get Dataset → Add Platform ────┤
  ├─→ LinkedIn (Web Dev) → Get Dataset → Normalize ──────────┤
  └─→ LinkedIn (Software Dev) → Get Dataset → Normalize ─────┤
                                                              ↓
                                                       [MERGE 4 SOURCES]
                                                              ↓
                                               Remove Duplicates
                                                              ↓
                                         Get All Existing Job URLs
                                                              ↓
                              Filter Out Duplicates & Jobs Older Than 7 Days
                                                              ↓
                                         Loop Over Dataset Items
                                                              ↓
                                           Score and Reason (AI)
                                                              ↓
                                         JSONify Score & Reasoning
                                                              ↓
                                               If Score > 5?
                                                   ↓ Yes
                                         Generate Cover Letter (AI)
                                                   ↓
                                         Append to Google Sheets
                                                   ↓
                                            [Loop Completes]
                                                   ↓
                                      Aggregate Results & Send Email
```

## Setup

**Prerequisites:**

- n8n instance
- Apify account
- Google Gemini API key
- Google Sheets
- MailerSend account (or any email service for notifications)

**Installation:**

1. Clone and import the workflow

   ```bash
   git clone https://github.com/majidseifi/JobSeeker.git
   ```

2. Import `Job Seeker.json` into n8n

3. Add credentials in n8n:

   - Apify API
   - Google Gemini API
   - Google Sheets OAuth2

4. Create a Google Sheet with columns: Title, Platform, Date, Company Name, City, Expired, Remote, Link, Rating, Resume Version, Cover Letter, Salary, Job Description, Status

5. Update these nodes:
   - Search parameters in Indeed/LinkedIn actors (4 nodes: Web Dev + Software Dev for each platform)
   - Google Sheets document ID
   - Your resume versions in JSONify node
   - MailerSend API token and email addresses in Send Email node

See `SETUP_GUIDE.md` for detailed steps.

## Customization

**Search parameters** - Edit the 4 actor nodes to change location, keywords, or job type (2 Indeed + 2 LinkedIn)

**Job filtering** - Edit the "Remove Duplicates in Current Batch" node to filter out unwanted job titles

**Scoring criteria** - Modify the "Score and Reason" node:

- Resume fit: 0-4 points
- Salary ($60k+/year): 0-2 points
- Remote: 0-2 points
- Benefits: 0-1 point
- Full-time: 0-1 point

**Cover letter style** - Edit the "Generate Cover Letter" node prompt

## Output

![Google Sheets Output](docs/images/google-sheets-output.png)

All jobs saved with AI rating, recommended resume version, and personalized cover letters for top matches.

## Performance

With 4 parallel searches (200 jobs/day total):

- Removes duplicates across all 4 sources
- Filters jobs older than 7 days
- ~100-150 unique jobs scored after filtering
- ~50-75 cover letters generated for high-scoring jobs
- Email notification sent with results summary

## Author

**YOUR_FULL_NAME**

- Website: [yourdomain.com](https://yourdomain.com)
- LinkedIn: [linkedin.com/in/your-profile](https://linkedin.com/in/your-profile)
- Email: majid@yourdomain.com

## Disclaimer

This tool is for personal use. Be mindful of web scraping terms of service, API rate limits and costs, and data privacy when storing job information. Use responsibly and in accordance with all applicable terms of service and local laws.
