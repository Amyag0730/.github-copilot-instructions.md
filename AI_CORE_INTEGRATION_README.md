# AI Core / Integration Component README

## Overview

The AI Core / Integration component is the **heart of the Forensic Psychology Risk Assessment Assistant**. It takes raw case notes from the Ingestion component, runs them through AI-powered analysis chains using Flowise, and produces structured risk assessments. It classifies risk categories, analyzes behavioral indicators, and recommends assessment focus areas—all in clean JSON format. This component bridges Flowise (the AI logic) and n8n (the workflow orchestration), making sure data flows reliably from input to output.

## What This Component Does

1. **Receives case notes** from the Ingestion component via Airtable (status="new")
2. **Classifies risk categories** using a Flowise LLM chain (e.g., "self-harm," "substance-use," "low-risk")
3. **Analyzes behavioral indicators** by extracting patterns from the case note text
4. **Recommends assessment focus areas** for the Specialist component to review
5. **Outputs structured JSON** with all analysis results back to Airtable
6. **Updates workflow status** to signal handoff to the next component

## Architecture & Data Flow

```
┌─────────────────────────────────────────────────────────────┐
│                    AIRTABLE (Case Notes Table)               │
│                                                              │
│  ┌────────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐ │
│  │   status   │  │case_note │  │risk_cat  │  │behavior  │ │
│  │ (new→...)  │  │(input)   │  │(output)  │  │(output)  │ │
│  └────────────┘  └──────────┘  └──────────┘  └──────────┘ │
│         ↑              ↑              ↓              ↓       │
└─────────────┼──────────┼──────────────┼──────────────┼──────┘
              │          │              │              │
              │          │         ┌────────────────┐  │
        ┌──────────────┐ │         │   N8N WORKFLOW  │  │
        │  INGESTION   │─┤         │                 │  │
        │  (reads)     │ │         │ ┌────────────┐ │  │
        └──────────────┘ │         │ │Airtable    │ │  │
                         │         │ │Trigger:   │ │  │
                    ┌─────────────┐│ │status=new │ │  │
                    │ AI CORE /   ││ └────────────┘ │  │
                    │ INTEGRATION ││                │  │
                    │ (reads from │├─┐┌────────────┐│  │
                    │ Airtable,   ││ ││Flowise     ││  │
                    │ writes back)││ ││Chain 1:    ││  │
                    └─────────────┘│ ││Classifier  ││  │
                         │         │ └────────────┘│  │
                         │         │                │  │
                         │         │ ┌────────────┐│  │
                         │         │ │Flowise     ││  │
                         │         │ │Chain 2:    ││  │
                         │         │ │Analyzer    ││  │
                         │         │ └────────────┘│  │
                         │         │                │  │
                         │         │ ┌────────────┐│  │
                         │         │ │Flowise     ││  │
                         │         │ │Chain 3:    ││  │
                         │         │ │Recommender ││  │
                         │         │ └────────────┘│  │
                         │         │                │  │
                         │         │ ┌────────────┐│  │
                         │         │ │Airtable    ││  │
                         │         │ │Update:    ││  │
                         │         │ │Write all   ││  │
                         │         │ │fields & set││  │
                         │         │ │status=...  ││  │
                         │         │ └────────────┘│  │
                         │         └────────────────┘  │
                         └──────────────────────────────┘
                              ↓
              ┌──────────────────────────────────┐
              │   SPECIALIST COMPONENT           │
              │   (reads status="analyzed")      │
              └──────────────────────────────────┘
```

## Component Inputs & Outputs

### Inputs (from Airtable)
- **case_note** (text): Raw forensic psychology case note
- **status** (select): Must be "new" to trigger processing
- Other fields: case_id, date, clinician_name (optional, for context)

### Outputs (written to Airtable)
```json
{
  "risk_category": "self-harm",
  "behavioral_indicators": ["suicidal ideation", "isolation", "hopelessness"],
  "protective_factors": ["supportive family", "ongoing therapy"],
  "follow_up_questions": ["Any recent suicide attempts?", "Current medication?"],
  "records_to_review": ["psychiatric evaluations", "family history"],
  "urgent_review_needed": true
}
```

Also updates **status** field:
- `status="new"` → triggers processing
- `status="classified"` → classifier done
- `status="analyzed"` → analyzer done  
- `status="recommended"` → recommender done
- `status="error"` → something failed (with error_log)

## Setup Instructions

### Prerequisites

You'll need accounts and API keys for:

1. **Flowise** (local or cloud)
2. **n8n** (local or cloud)
3. **Groq or HuggingFace API** (for LLM inference)
4. **Airtable** (with Case Notes table created)

### Step 1: Airtable Setup

1. **Create the Case Notes table** (or use existing):
   
   | Field Name | Type | Description |
   |-----------|------|-------------|
   | case_id | Single line text | Unique identifier |
   | case_note | Long text | Raw input from Ingestion |
   | status | Single select | `new`, `classified`, `analyzed`, `recommended`, `complete`, `error` |
   | risk_category | Single select | Values like "self-harm", "substance-use", "violence", "low-risk" |
   | behavioral_indicators | Long text | JSON array of indicators |
   | protective_factors | Long text | JSON array of protective factors |
   | follow_up_questions | Long text | JSON array of follow-up questions |
   | records_to_review | Long text | JSON array of records to review |
   | urgent_review_needed | Checkbox | True/false flag |
   | error_log | Long text | Error messages if status="error" |
   | date_created | Created time | Auto-filled |
   | date_updated | Last modified time | Auto-filled |

2. **Get your Airtable API key**:
   - Go to [airtable.com/account](https://airtable.com/account)
   - Generate a Personal Access Token
   - Copy and save it (you'll need it in n8n)

3. **Get your Base ID and Table ID**:
   - Open your base in Airtable
   - Look at the URL: `https://airtable.com/[BASE_ID]/[TABLE_ID]`
   - Copy both IDs

### Step 2: Flowise Setup

1. **Install Flowise** (if not already done):
   ```bash
   npm install -g flowise
   flowise start
   ```
   Flowise runs on `http://localhost:3000` by default.

2. **Create Three LLM Chains** in Flowise:

   #### Chain 1: Risk Classifier
   - **Input**: case_note (string)
   - **Output**: risk_category (string, one of: "self-harm", "substance-use", "violence", "other")
   - **Prompt**: "Analyze this forensic psychology case note and classify the primary risk category. Return ONLY the category name, no explanation."
   - **LLM Model**: Groq or HuggingFace (see Step 3)

   #### Chain 2: Behavioral Analyzer
   - **Input**: case_note (string)
   - **Output**: behavioral_indicators (JSON array), protective_factors (JSON array)
   - **Prompt**: "Extract behavioral indicators and protective factors from this case note. Return as JSON: {\"behavioral_indicators\": [...], \"protective_factors\": [...]}"
   - **LLM Model**: Same as Chain 1

   #### Chain 3: Assessment Recommender
   - **Input**: case_note (string), risk_category (string), behavioral_indicators (array)
   - **Output**: assessment_focus_areas (array), follow_up_questions (array), records_to_review (array)
   - **Prompt**: "Based on this risk category and indicators, recommend assessment focus areas, follow-up questions, and records to review. Return as JSON: {\"assessment_focus_areas\": [...], \"follow_up_questions\": [...], \"records_to_review\": [...]}"
   - **LLM Model**: Same as Chains 1 & 2

3. **Get your Flowise API Endpoint URLs**:
   - For each chain, click "Deploy"
   - Copy the API endpoint (e.g., `http://localhost:3000/api/v1/prediction/[CHAIN_ID]`)
   - Save all three endpoints

### Step 3: LLM API Setup (Groq or HuggingFace)

Choose one:

#### Option A: Groq (Recommended - Faster)
1. Sign up at [console.groq.com](https://console.groq.com)
2. Create an API key
3. In Flowise, add Groq as your LLM provider:
   - Model: `mixtral-8x7b-32768` or `llama-2-70b-chat`
   - API Key: Paste your Groq API key

#### Option B: HuggingFace
1. Sign up at [huggingface.co](https://huggingface.co)
2. Create an API token
3. In Flowise, add HuggingFace as your LLM provider:
   - Model: `meta-llama/Llama-2-7b-chat-hf`
   - API Token: Paste your HuggingFace token

### Step 4: n8n Workflow Setup

1. **Install n8n** (if not already done):
   ```bash
   npm install -g n8n
   n8n start
   ```
   n8n runs on `http://localhost:5678` by default.

2. **Create n8n Credentials**:
   - Go to **Settings → Credentials**
   - Add **Airtable Credentials**:
     - Name: "Airtable API"
     - API Key: Paste your Airtable API key from Step 2
   - Add **HTTP Credentials** for Flowise (if needed for authentication)

3. **Create the Main Workflow** in n8n:

   **Workflow Name**: `Forensic_AI_Core_Processor`

   **Steps**:

   1. **Airtable Watch Records** (Trigger)
      - Credentials: Airtable API
      - Base ID: Paste from Step 2
      - Table: Case Notes
      - Trigger: Watch for new records OR when field status changes
      - Filter: status = "new"
      - Poll interval: Every 5 minutes (or real-time if available)

   2. **Extract case_note**
      - Use "Set" node or "Function" node
      - Extract `case_note` field from Airtable record

   3. **Flowise Chain 1: Classifier**
      - Use HTTP POST request
      - URL: Flowise classifier endpoint from Step 2
      - Method: POST
      - Body:
        ```json
        {
          "question": "{{ $node['Extract case_note'].json.case_note }}"
        }
        ```
      - Headers: Content-Type: application/json
      - Save output as `risk_category`

   4. **Flowise Chain 2: Analyzer**
      - HTTP POST request to analyzer endpoint
      - Body:
        ```json
        {
          "question": "{{ $node['Extract case_note'].json.case_note }}"
        }
        ```
      - Parse JSON response
      - Extract `behavioral_indicators` and `protective_factors`

   5. **Flowise Chain 3: Recommender**
      - HTTP POST request to recommender endpoint
      - Body:
        ```json
        {
          "question": "{{ $node['Extract case_note'].json.case_note }}",
          "risk_category": "{{ $node['Flowise Chain 1'].json.text }}",
          "behavioral_indicators": {{ $node['Flowise Chain 2'].json.behavioral_indicators }}
        }
        ```
      - Parse JSON response
      - Extract all outputs

   6. **Airtable Update Record**
      - Credentials: Airtable API
      - Base ID: Same as Step 1
      - Table: Case Notes
      - Record ID: From trigger
      - Update fields:
        ```
        risk_category: {{ $node['Flowise Chain 1'].json.risk_category }}
        behavioral_indicators: {{ JSON.stringify($node['Flowise Chain 2'].json.behavioral_indicators) }}
        protective_factors: {{ JSON.stringify($node['Flowise Chain 2'].json.protective_factors) }}
        follow_up_questions: {{ JSON.stringify($node['Flowise Chain 3'].json.follow_up_questions) }}
        records_to_review: {{ JSON.stringify($node['Flowise Chain 3'].json.records_to_review) }}
        urgent_review_needed: {{ $node['Flowise Chain 3'].json.urgent_review_needed }}
        status: "recommended"
        ```

   7. **Error Handler**
      - Add error handling after each Flowise call
      - On error: Update Airtable with `status="error"` and `error_log` with error message

4. **Test the Workflow**:
   - Click "Activate" to turn on the workflow
   - See "Test Workflow" section below

## Testing Instructions

### Test 1: Manual Trigger (Fastest)

1. **Create a test record in Airtable**:
   - Add a new row to Case Notes table
   - case_id: "TEST_001"
   - case_note: "John is 24-year-old male presenting with suicidal ideation. Reports recent breakup and job loss. Lives alone. No current support system. Previously hospitalized for depression."
   - status: "new"
   - Leave other fields blank
   - **Save**

2. **Trigger the n8n workflow**:
   - Go to n8n dashboard
   - Open `Forensic_AI_Core_Processor` workflow
   - Click "Execute Workflow" (play button)
   - Wait 30-60 seconds

3. **Check Airtable for results**:
   - Refresh the Case Notes table
   - Look at TEST_001 record
   - Verify these fields are now filled:
     - ✅ risk_category: "self-harm" (or similar)
     - ✅ behavioral_indicators: JSON array with values like ["suicidal ideation", "isolation"]
     - ✅ protective_factors: JSON array
     - ✅ status: "recommended"

4. **Success Criteria**:
   - All output fields populated
   - No "error" status
   - JSON is valid (can be parsed)

### Test 2: Automated Trigger

1. **Create a test record with status="new"**:
   - Add to Airtable as above
   - The workflow should automatically trigger within 5-15 minutes
   - Check results same as Test 1

### Test 3: Edge Cases (Test Error Handling)

Create test records with problematic data:

#### Edge Case 3a: Very Short Note
- case_note: "Patient sad."
- Expected: Should still classify and analyze (may give generic results)
- Verify: Status updates (doesn't hang or error)

#### Edge Case 3b: Missing Data
- case_note: ""
- Expected: Should error gracefully
- Verify: status="error", error_log contains explanation

#### Edge Case 3c: Non-English Text
- case_note: "患者很伤心" (Chinese for "patient is sad")
- Expected: LLM may fail or give unexpected results
- Verify: Handled gracefully (error or generic output)

#### Edge Case 3d: Very Long Note
- case_note: 5000+ words of case details
- Expected: May timeout or hit token limit
- Verify: Error handling works

### Test 4: Full End-to-End Validation

1. **Verify all three Flowise chains are working**:
   - Open Flowise
   - Test each chain with sample case note
   - Check JSON output is valid

2. **Verify n8n workflow steps**:
   - Check each HTTP request succeeds (green checkmark)
   - Check data flows correctly between steps
   - Verify Airtable updates happen last

3. **Create 5 diverse test records**:
   - Normal low-risk case
   - Self-harm case
   - Substance-use case
   - Vague/short case
   - Missing/bad data case

4. **Run workflow on all 5**:
   - Check success rate (aim for 4/5 or 5/5)
   - Document any failures
   - Fix before Checkpoint 2

## Logging & Debugging

### Where to Check for Errors

1. **n8n Workflow Logs**:
   - Open workflow
   - Look at "Execution" history
   - Click on any red-marked execution
   - See detailed error messages

2. **Airtable error_log field**:
   - If status="error", error message is here
   - Contains API response or timeout info

3. **Flowise Logs**:
   - Terminal where Flowise is running
   - Shows LLM API calls and responses
   - Look for rate limit or authentication errors

4. **Browser Console** (if using web version):
   - Press F12
   - Check for JavaScript errors
   - Check Network tab for failed HTTP requests

### Common Issues & Fixes

| Issue | Cause | Fix |
|-------|-------|-----|
| "Airtable API Key invalid" | Wrong or expired key | Regenerate in Airtable account settings |
| Flowise endpoint 404 | Chain not deployed or wrong URL | Redeploy chain in Flowise, copy URL again |
| "LLM request timed out" | API overloaded or slow internet | Increase timeout in n8n HTTP node (30s → 60s) |
| Blank output fields | JSON parsing failed | Check Flowise output format matches expected JSON |
| Workflow never triggers | Airtable Watch trigger misconfigured | Verify "status = new" filter and polling interval |
| "Unexpected token in JSON" | Arrays not converted to strings | Wrap arrays in `JSON.stringify()` in n8n |

## Known Limitations

1. **LLM Accuracy**
   - The risk classifications and behavioral analyses are only as good as your Flowise prompts and LLM model
   - Groq's Mixtral and LLaMA 2 are good but not perfect; consider human review for high-risk cases
   - **Workaround**: Specialist component is designed for human review before final decision

2. **API Rate Limits**
   - Groq, HuggingFace, and Airtable all have rate limits
   - Processing >100 records/day may hit limits
   - **Workaround**: Stagger requests; upgrade API plans if needed

3. **No Token Counting**
   - Very long case notes (>4000 words) may exceed LLM token limits
   - Will fail silently or timeout
   - **Workaround**: Pre-processing in Ingestion component to summarize long notes

4. **No Caching**
   - If same case note is processed twice, it's re-analyzed each time
   - Wastes API calls
   - **Workaround**: Add deduplication check in n8n (check if case_note already processed)

5. **JSON Output Format Not Guaranteed**
   - LLM may not return perfect JSON even with prompting
   - May require JSON repair/fallback logic
   - **Workaround**: Add validation in n8n to catch malformed JSON; set default values

6. **No Historical Tracking**
   - Updates overwrite previous analysis
   - Can't see audit trail of how assessment changed
   - **Workaround**: Add version history field or separate analysis log table

7. **Flowise Chain Changes Require Re-Deployment**
   - If you edit prompts in Flowise, must click "Deploy" again
   - n8n will use old version until re-deployed
   - **Workaround**: Document when chains were last updated; re-deploy before major runs

## Handoff Protocol

### What the Specialist Component Expects

When status="analyzed" (final output from AI Core):
- All output fields populated with valid JSON or text
- `urgent_review_needed` set to true/false
- `status="analyzed"` (exact match)

### What Integration Component Expects to Receive

From Ingestion:
- status="new" record
- case_note field populated
- case_id for tracking

## Contributing & Improvements

If you modify this component:
1. Document any new Flowise chains or n8n steps
2. Test with all 5 edge cases before committing
3. Update this README with changes
4. Notify the Integration lead if you change field names or output format

## Questions?

- **Flowise help**: See [Flowise docs](https://docs.flowiseai.com)
- **n8n help**: See [n8n docs](https://docs.n8n.io)
- **Airtable help**: See [Airtable API docs](https://airtable.com/api)
- **Team coordination**: Check GitHub Issues or team Slack

---

**Last Updated**: Checkpoint 2 (Week 8)  
**Status**: In Development  
**Owner**: AI Core / Integration Team
