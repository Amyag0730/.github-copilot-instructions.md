 # Capstone Project Context

## Project
- **Name:** Forensic Psychology Risk Assessment Assistant
- **What it does:** This project helps organize and analyze forensic psychology case notes. It takes case notes as input, classifies the main risk factor category, analyzes behavioral indicators, and recommends assessment focus areas for further review.
- **Project type:** Forensic Psychology AI Workflow

## Architecture
- **Ingestion:** n8n receives or creates sample forensic psychology case notes and sends them through the workflow.
- **AI Core:** Flowise LLM chains use Groq to classify risk categories, analyze behavioral indicators, and recommend assessment focus areas.
- **Specialist:** The system produces structured JSON outputs that can be reviewed by a student, evaluator, or project team member.
- **Integration:** n8n connects the Flowise chains together so one case note flows through classification, analysis, and recommendation.

## Tech Stack
- n8n Cloud
- Flowise Cloud
- Groq API
- GitHub
- VS Code
- GitHub Copilot

## Current State

- **What's working:** The Flowise LLM chains are working for the forensic psychology workflow. Chain 1 classifies the risk factor category, Chain 2 analyzes behavioral indicators, and Chain 3 recommends assessment focus areas. The n8n workflow can pull a case note from Airtable, send it through the Flowise chains, and write the final recommendation back into Airtable. The test record successfully reached the final Airtable view with the status updated to complete.

- **What's in progress:** The workflow still needs cleaner JSON field mapping. Right now, the final Chain 3 JSON response is being saved into the assessment_focus_areas field instead of splitting every value into separate Airtable fields. A more polished dashboard view also still needs to be created.

- **Known issues:** The processed_at field caused an Airtable update error because Airtable did not accept the date value sent from n8n. This field was removed from the update step for now and needs to be fixed later using the correct date format. Error handling is also not fully complete yet, so failed steps do not automatically update the Airtable status to error.

- **Schema updates:** The Case Notes table now includes fields for case_note, risk_category, behavioral_indicators, assessment_focus_areas, urgent_review_needed, status, created_at, and processed_at. The processed_at field needs to be checked because it caused a date-format issue during Checkpoint 2 testing.

- **Next milestone:** Improve the workflow by fixing processed_at, mapping JSON outputs into separate Airtable fields, adding error handling, creating more test records, and building a cleaner dashboard view.

## Conventions
- Outputs should be valid JSON.
- Field names should use snake_case.
- The system should not diagnose people or make legal conclusions.
- The system should focus on risk category, behavioral indicators, and assessment focus areas.
