# Prompt Log — Amy Aguaysa

**Project:** Forensic Psychology Risk Assessment Assistant   
**My Component:** AI Core / Integration  
**AI Tools Used:** GitHub Copilot, ChatGPT  

---

## How to Use This Log

Add an entry for each significant AI interaction:
- Copilot Chat conversations where I asked it to generate, explain, or debug something
- Moments where Copilot was wrong and I had to fix it
- Cases where I refined a prompt to get a better result

---

## 2026-05-22 — Created AI Core README with Copilot

**Context:** I was working on the Week 8 assignment and needed to create a README for my AI Core / Integration component. My project uses Flowise LLM chains and n8n to process forensic psychology case notes.

**Prompt:**
> Using the project context from copilot-instructions.md, write a complete README for my AI Core / Integration component. Include what it does, how it connects to other components, setup instructions, how to test it, and known limitations. My component uses Flowise LLM chains and n8n to process forensic psychology case notes. It classifies the risk factor category, analyzes behavioral indicators, and recommends assessment focus areas using structured JSON outputs. Please write it in clear student-friendly language and format it as a README.md file.

**Result:** Copilot generated a README that explained the purpose of my AI Core / Integration component, how the Flowise chains connect to n8n, what setup is needed, how to test the workflow, and what limitations still exist.

**Evaluation:** The result was useful because it gave me a clear structure for my README. However, I still had to review it because Copilot may guess some project details, especially about Airtable and the full team workflow.

**What I changed:** I edited the README to match my actual project setup. I made sure it said that my Week 8 workflow currently uses n8n Edit Fields for testing and that the full Airtable handoff is still in progress.

**What I learned:** I learned that Copilot gives better results when I include specific project context, tools, and limitations in the prompt. I also learned that AI-generated documentation still needs to be checked for accuracy before submitting.

## 2026-05-22 — Created forensic psychology LLM chain prompts

**Context:** I was completing the Week 8 assignment and needed to adapt the required cybersecurity LLM chains to my forensic psychology domain.

**Prompt:**
> Help me adapt the Week 8 LLM chain assignment to forensic psychology. I can only create two Flowise chatflows, but I still need to show three chain functions.

**Result:** The AI helped me create prompts for a Risk Factor Classifier, Behavioral Indicator Analyzer, and Assessment Focus Recommender. It also helped me explain that Chains 2 and 3 were combined into one chatflow because of my Flowise limit.

**Evaluation:** This worked because I was able to test all three chain functions and produce valid JSON outputs. I still had to adjust the prompts to make sure they did not diagnose people or make legal conclusions.

**What I changed:** I changed the topic from cybersecurity alerts to forensic psychology case notes and risk assessment.

**What I learned:** I learned that the same LLM chain structure can be reused for different domains if the role, output format, and constraints are changed clearly.

---

## 2026-05-22 — Generated AI Core README with Copilot

**Context:** I was working on the Week 8 assignment and needed to create a README for my AI Core / Integration component.

**Prompt:**
> Using the project context from copilot-instructions.md, write a complete README for my AI Core / Integration component. Include what it does, how it connects to other components, setup instructions, how to test it, and known limitations.

**Result:** Copilot generated a README explaining the purpose of my component, how Flowise connects to n8n, what setup is needed, how to test the workflow, and what limitations still exist.

**Evaluation:** The structure was helpful, but I had to check the details because Copilot could guess things about Airtable or the team workflow that were not fully true yet.

**What I changed:** I edited the README to say that Airtable integration was still in progress at that point and that the workflow was being tested through n8n.

**What I learned:** Copilot works better when the project instructions file gives it clear context about the actual tools and workflow.

---

## 2026-05-23 — Debugged Airtable Search records in n8n

**Context:** I was working on the Week 9 end-to-end integration test and needed n8n to pull a case note from Airtable.

**Prompt:**
> My Airtable Search records node is not returning my case note. It says no output data returned. How do I fix it?

**Result:** The AI helped me check the Airtable base, table, field names, and filter formula. I learned that leaving the Filter By Formula blank was the fastest way to test if Airtable was connected.

**Evaluation:** This helped because Airtable eventually returned the record with the case_note field. The issue was mainly with the filter formula and making sure the record actually existed in Airtable.

**What I changed:** I removed the filter temporarily and deleted extra blank Airtable records so the workflow only returned the real test case note.

**What I learned:** Before adding filters, it is better to confirm the Airtable node can return records at all.

---

## 2026-05-23 — Fixed Flowise API URL issue in n8n

**Context:** Chain 1 was returning website HTML instead of a Flowise LLM response.

**Prompt:**
> My n8n HTTP Request node is returning HTML code from Flowise instead of the JSON response. What is wrong?

**Result:** The AI explained that I was using the Flowise chatbot URL instead of the API prediction URL. I needed to replace `/chatbot/` with `/api/v1/prediction/`.

**Evaluation:** This solved the issue because Chain 1 started returning Flowise output instead of webpage code.

**What I changed:** I replaced the Flowise chatbot links with API prediction links in the HTTP Request nodes.

**What I learned:** The chatbot link is for testing in the browser, but n8n needs the API prediction endpoint.

---

## 2026-05-23 — Documented Checkpoint 2 end-to-end test results

**Context:** After running the Week 9 workflow, I needed to write `docs/checkpoint2-results.md` and explain what worked and what broke.

**Prompt:**
> Help me fill out the Checkpoint 2 Results template for my forensic psychology workflow. The record went from Airtable to Flowise chains and back to Airtable, but processed_at caused a date error.

**Result:** The AI helped me write a component-by-component report for Ingestion, AI Core, Specialist, and Integration Dashboard. It also helped me list the gaps and create a fix plan.

**Evaluation:** This worked well because it matched what actually happened during my test. The report was honest because it said the end-to-end status was PARTIAL, not perfect.

**What I changed:** I edited the report to match my screenshot names and my actual Airtable fields.

**What I learned:** A partial result is still useful if I clearly document what worked, what failed, and what needs to be fixed next.
