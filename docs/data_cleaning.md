# Data Cleaning Notes

Source file:

- `data/marketing_campaign_raw.csv`

Generated files:

- `data/marketing_campaign_clean.csv`
- `data/n8n_sample_customers.csv`

## Cleaning Rules

The original Kaggle dataset was tab-separated, even though it used a `.csv`
extension. The cleaned version was exported as a standard comma-separated CSV.

Rows were removed when:

- `Income` was missing.
- `age_at_campaign` was below 18 or above 90.
- `income` was below or equal to 0, or above 200,000.

The campaign reference year is 2014, based on the dataset period.

## Standardization

- `Education = 2n Cycle` was renamed to `Second Cycle`.
- Rare marital statuses (`Alone`, `Absurd`, `YOLO`) were grouped as `Other`.
- Dates were converted from `dd-MM-yyyy` to `yyyy-MM-dd`.

## Derived Variables

The clean dataset adds:

- `age_at_campaign`
- `dependents`
- `total_spend`
- `total_purchases`
- `accepted_campaigns_total`
- `previous_campaign_accepted`
- `preferred_category`
- `preferred_channel`
- `intent_segment`
- `marketing_segment`
- `baseline_lead_score`
- `purchase_likelihood`
- `suggested_action`

## Baseline Scoring Logic

`baseline_lead_score` is a rule-based score from 1 to 100. It combines:

- Income strength
- Total spending
- Recent activity
- Purchase volume
- Previous campaign acceptance

This score is not presented as a machine learning model. It is a transparent
baseline that the n8n and LLM workflow can use as structured context for AI
lead analysis and personalized messaging.

## n8n Sample

`data/n8n_sample_customers.csv` contains 30 customers:

- 10 high-likelihood customers
- 10 medium-likelihood customers
- 10 low-likelihood customers

This smaller file is intended for early workflow testing in n8n.
