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
- **What's working:** Two Flowise chatflows are working. Chain 1 classifies forensic psychology risk factors. Chain 2 is used for both behavioral analysis and assessment recommendations. The n8n pipeline successfully connects the classifier, analyzer, and recommender nodes.
- **What's in progress:** Documentation, Copilot setup, audit report, and prompt log.
- **Known issues:** Flowise account only allowed two chatflows, so Chain 2 and Chain 3 were combined into one chatflow and tested with separate prompts.
- **Next milestone:** Checkpoint 2 — one record flowing through all required components end-to-end.

## Conventions
- Outputs should be valid JSON.
- Field names should use snake_case.
- The system should not diagnose people or make legal conclusions.
- The system should focus on risk category, behavioral indicators, and assessment focus areas.
