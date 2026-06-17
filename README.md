# AI-Powered Marketing Automation System with n8n and Large Language Models

## Project Overview

This project demonstrates an AI-powered marketing automation workflow built with
n8n and OpenAI. The workflow processes CRM-style customer campaign data, scores
customer intent, recommends a marketing action, and generates a personalized
marketing email.

The prototype was designed for an individual university assignment on Big Data
and AI for Marketing. It focuses on practical marketing value rather than
building a complex machine learning model from scratch.

## Research Question

How can AI-powered automation improve marketing efficiency and customer
personalization when processing CRM and campaign response data?

## What The Prototype Does

The system takes one customer record and runs it through two AI-assisted steps:

1. Lead/customer intent analysis
2. Personalized marketing email generation

The workflow combines a transparent rule-based baseline score with an LLM-based
assessment. The final output includes customer profile data, an AI lead score,
intent segment, purchase likelihood, recommended action, personalized email
copy, call to action, recommended channel, and follow-up timing.

## Repository Structure

```text
data/
  marketing_campaign_raw.csv
  marketing_campaign_clean.csv
  n8n_sample_customers.csv

docs/
  architecture.md
  data_cleaning.md
  n8n_setup_guide.md

prompts/
  lead_scoring_prompt.txt
  personalization_prompt.txt

workflow/
  n8n_workflow.json
  test_customer_input.json
  lead_analysis_agent_output.json
  parse_lead_analysis_output.json
  personalization_agent_output.json
  final_personalized_email_output.json
  workflow_diagram.png

screenshots/
  01_n8n_dry_run_success.png
  02_lead_analysis_agent_output.png
  03_parse_lead_analysis_output.png
  04_personalization_agent_output.png
  05_final_personalized_email_output.png

research/
  papers.md
  references.md

report/
  final_report.md
```

## Dataset

The project uses the Kaggle Customer Personality Analysis / marketing campaign
dataset as realistic CRM-style customer data. The raw file was cleaned and
converted into:

- `data/marketing_campaign_clean.csv`: full cleaned dataset with 2,212 rows.
- `data/n8n_sample_customers.csv`: small 30-customer sample for controlled
  n8n testing.
- `workflow/test_customer_input.json`: one-customer input used for the live
  prototype.

Cleaning removed rows with missing income, unrealistic age values, and extreme
income values. The cleaned dataset also includes derived fields such as
`total_spend`, `total_purchases`, `preferred_category`, `preferred_channel`,
`baseline_lead_score`, and `purchase_likelihood`.

See `docs/data_cleaning.md` for the full cleaning notes.

## Workflow Architecture

The exported n8n workflow follows this main branch:

```text
Manual Trigger
  -> Customer Input
  -> Lead Analysis Agent
  -> Parse Lead Analysis
  -> Personalization Agent
  -> Parse Personalization
```

The workflow diagram is saved at:

- `workflow/workflow_diagram.png`

The workflow export is saved at:

- `workflow/n8n_workflow.json`

## AI Agent Design

### Lead Analysis Agent

Prompt:

- `prompts/lead_scoring_prompt.txt`

Purpose:

- Analyze customer profile, purchase behavior, recency, campaign history, and
  channel preferences.
- Refine the baseline rule-based score.
- Return a structured JSON output with AI lead score, intent segment, purchase
  likelihood, recommended action, offer type, reasoning summary, positive
  signals, risk signals, and confidence level.

### Personalization Agent

Prompt:

- `prompts/personalization_prompt.txt`

Purpose:

- Use the merged customer and lead analysis output.
- Generate a concise personalized marketing email.
- Recommend a call to action, personalization strategy, channel, and follow-up
  timing.

## Example Result

The live prototype processed customer `10133`.

Key input signals:

- Age at campaign: 44
- Income: 93,790
- Total spend: 2,349
- Total purchases: 25
- Preferred category: Wine
- Preferred channel: Store
- Previous campaigns accepted: Yes
- Baseline lead score: 95

AI output summary:

- AI lead score: 88
- Intent segment: Medium Intent
- Purchase likelihood: High
- Recommended action: Engage with personalized offers via store visits.
- Recommended channel: Store
- Email subject: Exclusive Discounts Await You at Our Wine Store!
- Call to action: Visit Us In-Store for Exclusive Offers!

The complete final output is saved at:

- `workflow/final_personalized_email_output.json`

## Cost And Safety Controls

The implementation was deliberately tested in a cost-controlled way:

- The workflow was kept inactive.
- A Manual Trigger was used.
- No schedule was enabled.
- No active webhook was used.
- No real email sender was connected.
- Only one customer was processed in the live AI prototype.
- Only two OpenAI calls were made: one for lead analysis and one for email
  personalization.
- The 30-row n8n sample and the full 2,212-row cleaned dataset were not run
  through OpenAI.

## Evidence Files

Screenshots in `screenshots/` show:

- Successful no-OpenAI dry run.
- Lead Analysis Agent output.
- Parsed lead analysis output.
- Personalization Agent output.
- Final personalized email output.

JSON output files in `workflow/` preserve the structured result from each
important workflow stage.

## Notes Before Public Upload

The exported workflow has been anonymized for sharing. It does not include an
OpenAI secret key, n8n credential name, n8n credential ID, workflow instance ID,
workflow version ID, or exported workflow ID.

If the workflow is imported back into n8n, the OpenAI credential must be
selected again in the `Lead Analysis Agent` and `Personalization Agent` nodes
before running the AI branch.

## Future Extensions

Possible future improvements include:

- Connecting the workflow to a real CRM or website lead form.
- Adding Google Sheets or database storage.
- Sending emails through a marketing platform after human approval.
- Testing multiple customers from the 30-row sample.
- Training a supervised response prediction model using the historical
  `Response` field.
- Adding A/B testing for generated email variants.
