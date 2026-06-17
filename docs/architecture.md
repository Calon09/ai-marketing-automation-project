# Workflow Architecture

## Project Context

This project implements an AI-powered marketing automation workflow using n8n
and Large Language Models. Instead of using a fully simulated website lead form,
the workflow uses a real Kaggle marketing campaign dataset as CRM-style customer
data.

The goal is to show how a marketing team can automate customer scoring,
segmentation, and personalized communication using structured customer data and
AI agents.

## Adapted Research Question

How can AI-powered automation improve marketing efficiency and customer
personalization when processing CRM and campaign response data?

## Input Data

Primary workflow input:

- `data/n8n_sample_customers.csv`

Full cleaned dataset:

- `data/marketing_campaign_clean.csv`

The n8n sample contains 30 customers selected from the clean dataset:

- 10 high-likelihood customers
- 10 medium-likelihood customers
- 10 low-likelihood customers

This allows the workflow to be tested quickly while preserving realistic
customer variety.

## High-Level Workflow

```text
Cleaned Customer Dataset
    -> n8n Manual Trigger or CSV Read Node
    -> Lead Analysis Agent
    -> AI Lead Score and Intent Segment
    -> Personalization Agent
    -> Personalized Email Output
    -> Google Sheets / CSV / CRM-style Storage
```

## n8n Workflow Design

### 1. Trigger

The workflow can start from either:

- A manual trigger for testing.
- A CSV read/import step using `data/n8n_sample_customers.csv`.
- A webhook in a more advanced version.

For the assignment prototype, a manual trigger or file-based workflow is enough.

### 2. Customer Data Input

Each row represents one customer profile with behavioral and campaign features,
including:

- Demographics: age, education, marital status, income, dependents.
- Engagement: recency, web visits, web purchases.
- Purchase behavior: total spend, total purchases, preferred category.
- Campaign history: previous campaign acceptance and historical response.
- Baseline scoring: rule-based lead score and purchase likelihood.

### 3. Lead Analysis Agent

Prompt file:

- `prompts/lead_scoring_prompt.txt`

Responsibilities:

- Analyze each customer profile.
- Refine the baseline rule-based score.
- Estimate customer intent.
- Identify positive and risk signals.
- Recommend a marketing action.

Expected output:

```json
{
  "customer_id": "string",
  "ai_lead_score": 0,
  "intent_segment": "High Intent | Medium Intent | Low Intent",
  "purchase_likelihood": "High | Medium | Low",
  "recommended_action": "string",
  "recommended_offer_type": "string",
  "reasoning_summary": "string",
  "key_positive_signals": ["string"],
  "key_risk_signals": ["string"],
  "confidence_level": "High | Medium | Low"
}
```

### 4. Personalization Agent

Prompt file:

- `prompts/personalization_prompt.txt`

Responsibilities:

- Use customer data and the lead analysis output.
- Generate a personalized marketing email.
- Adapt tone and offer type to the predicted intent.
- Recommend a follow-up channel and timing.

Expected output:

```json
{
  "customer_id": "string",
  "email_subject": "string",
  "email_preview_text": "string",
  "email_body": "string",
  "call_to_action": "string",
  "personalization_strategy": "string",
  "recommended_channel": "Email | Web | Catalog | Store",
  "follow_up_timing": "string"
}
```

### 5. Output Storage

The final workflow output can be stored in:

- Google Sheets
- A CSV file
- A simple CRM-style database table

For the assignment, Google Sheets is the preferred output because it is easy to
show in screenshots and explain in the report.

Suggested output columns:

- `customer_id`
- `baseline_lead_score`
- `ai_lead_score`
- `intent_segment`
- `purchase_likelihood`
- `recommended_action`
- `recommended_offer_type`
- `email_subject`
- `email_body`
- `call_to_action`
- `recommended_channel`
- `follow_up_timing`

## Why This Architecture Is Marketing-Relevant

The workflow demonstrates four practical marketing automation capabilities:

1. Automated customer prioritization.
2. AI-assisted segmentation.
3. Personalized content generation.
4. Scalable campaign execution.

Compared with a traditional manual workflow, the AI workflow can process more
customer records, generate more personalized messaging, and reduce repetitive
marketing operations.

## Evaluation Approach

The project can compare traditional marketing operations with the AI-assisted
workflow using the following criteria:

| Metric | Traditional Workflow | AI Automation Workflow |
| --- | --- | --- |
| Time per customer | Manual and slow | Automated and faster |
| Segmentation | Broad audience groups | Customer-level intent segments |
| Personalization | Generic templates | AI-generated tailored messages |
| Scalability | Limited by human effort | Can process many records |
| Consistency | Varies by marketer | Standardized scoring and prompts |

## Limitations

- The dataset represents historical campaign customers, not live website leads.
- The baseline score is rule-based and should not be interpreted as a trained
  predictive model.
- LLM outputs need validation before being used in real customer communication.
- The workflow prototype focuses on decision support and automation rather than
  advanced machine learning.

## Future Extensions

Possible future improvements include:

- Connecting the workflow to a live website form.
- Sending emails through Gmail, Mailchimp, or another email platform.
- Adding A/B testing for generated email variants.
- Training a predictive response model using the `Response` field.
- Building a dashboard for campaign performance and segment analysis.
