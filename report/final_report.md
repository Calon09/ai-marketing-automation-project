**Big Data & AI for Marketing - Individual Assignment - Álvaro Calonge**

**Selected course topic:** AI-powered marketing automation and customer personalization  
**Project approach:** Option 2 - Hands-On Applied Project  
**Core tools:** n8n, OpenAI API, CRM-style campaign data, prompt-based AI agents  
**Date:** June 2026

# AI-Powered Marketing Automation System with n8n and Large Language Models

## 1. Introduction

Marketing teams today sit on more customer data than they can realistically act on. The gap is not analytical — most CRM platforms already surface useful signals — but operational: deciding which customers deserve attention, choosing the right message, and adapting to individual preferences still largely falls to human judgment, applied manually and inconsistently at scale.

This project explores whether AI-powered automation can close that gap, specifically asking: *how can AI-powered automation improve marketing efficiency and customer personalization when processing CRM and campaign response data?*

To answer that question, a working prototype was built using n8n and OpenAI's API. The workflow takes a customer record from a real marketing campaign dataset, passes it through an AI agent that assesses purchase intent, and then hands the output to a second agent that generates a personalized email. The goal was deliberately modest — not to build a predictive model or a production-grade system, but to test whether a marketing team with no ML infrastructure could get meaningful, personalized output from structured CRM data combined with workflow automation and a large language model.

This submission follows the hands-on applied project route. It does not simply summarize AI marketing theory; it applies course concepts around automation, analytics, customer segmentation, and generative AI to a concrete marketing workflow, then reflects on what the prototype demonstrates and where its limitations are.

## 2. Literature Review

The relevance of AI to marketing is well documented, though the details matter. Davenport et al. (2020) offer a useful taxonomy: AI can affect marketing strategy, sales processes, customer behavior, personalization, and service. What makes their framing relevant here is the distinction between AI as a replacement and AI as augmentation. The prototype sits firmly in the latter category — the agents assist with scoring and content generation, but a human would still review the outputs before they reach a customer.

On the analytics side, Wedel and Kannan (2016) describe marketing analytics as a bridge between structured and unstructured data and actual marketing decisions, including personalization and privacy-aware design. The dataset used in this project is almost entirely structured — demographics, spending, recency, channel preference, campaign history — and those variables only become useful when connected to a specific action. The n8n workflow creates that connection.

The CRM framing draws on Payne and Frow (2005), who argue that customer relationship management is best understood as a cross-functional strategic process involving value creation, multichannel integration, information management, and performance assessment. This matters because the prototype is not simply a content generator bolted onto a dataset. It tries to replicate the logic of a CRM workflow: take in customer signals, assess intent, select a channel, and produce a relevant output. The structure is more important than the scale.

Huang and Rust (2018) add a useful lens from service research, distinguishing between AI tasks by intelligence level and suggesting that analytical and communicative tasks are especially well suited to automation and augmentation. That roughly describes what both agents are doing here: scoring intent based on behavioral signals, and drafting communication adapted to individual context.

Finally, Zhang et al. (2024) provide a framing for why LLMs specifically are relevant to marketing communication. Their work on personalized LLM use covers both the generation of adapted text and broader downstream personalization applications — and it also reinforces the point that evaluation and governance matter before any generated content goes near a real campaign.

## 3. Methodology

The dataset underpinning the project is the Kaggle Customer Personality Analysis dataset, a publicly available marketing campaign dataset that includes demographic variables, purchase behavior across product categories and channels, campaign response history, and web activity. Using a real dataset rather than a simulated lead form made the prototype more grounded — the customer signals are messy in the way actual CRM data is messy.

The raw data required a cleaning pass before it was usable. Records with missing income, implausible age values, non-positive income, and extreme outliers above 200,000 were removed. The cleaned dataset contains 2,212 rows and 31 columns, saved as `data/marketing_campaign_clean.csv`.

From the cleaned data, a set of derived variables was constructed to make the customer profile more interpretable for the AI agents:

- `age_at_campaign`, `dependents`, `total_spend`, `total_purchases`
- `accepted_campaigns_total`, `previous_campaign_accepted`
- `preferred_category`, `preferred_channel`
- `intent_segment`, `marketing_segment`
- `baseline_lead_score`, `purchase_likelihood`, `suggested_action`

The baseline lead score deserves a note. It is a rule-based score from 1 to 100, derived from income, spending, recency, purchase volume, and prior campaign acceptance. It is not a trained predictive model and is not presented as one. Its purpose is twofold: it gives the LLM a structured starting point for its assessment, and it creates a natural comparison — a transparent, rule-based score versus an AI-assisted interpretation of the same data.

For cost and control reasons, only one customer was processed through the live OpenAI nodes: customer `10133`. The full dataset and a 30-row sample were prepared but not sent through the API.

## 4. System Implementation

The n8n workflow was built as a manual, inactive workflow — meaning it only runs when triggered explicitly, with no scheduled executions. This was partly a practical decision to avoid unintended API calls, and partly a design choice: in a real marketing context, human-in-the-loop review before sending anything to customers is preferable to fully automatic execution.

The main branch runs as follows:

```
Manual Trigger → Customer Input → Lead Analysis Agent → Parse Lead Analysis → Personalization Agent → Parse Personalization
```

The workflow export is at `workflow/n8n_workflow.json`, with a diagram at `workflow/workflow_diagram.png`.

The Customer Input node holds a single customer record in JSON format (`workflow/test_customer_input.json`). The Lead Analysis Agent uses `gpt-4o-mini` with a prompt saved at `prompts/lead_scoring_prompt.txt`. The agent receives a structured set of customer variables — income, recency, total spend, total purchases, campaign acceptance history, preferred category and channel, baseline score, and historical response — and returns a JSON object containing an AI lead score, intent segment, purchase likelihood, recommended action, offer type, reasoning summary, positive and risk signals, and a confidence level.

The Parse Lead Analysis node is a JavaScript code node that extracts the JSON from the OpenAI response and merges it with the original customer input, creating a unified data object for the next stage.

The Personalization Agent also runs on `gpt-4o-mini`, using a prompt at `prompts/personalization_prompt.txt`. It receives the merged customer and lead analysis data, and generates a personalized email subject, preview text, body copy, call to action, personalization rationale, recommended channel, and follow-up timing. The final structured output is saved at `workflow/final_personalized_email_output.json`.

Mock versions of both agent nodes remain on the canvas as evidence of a dry run without API calls, but they are disconnected from the live branch.

The main implementation evidence is documented through the exported workflow, the saved JSON outputs, and screenshots:

- `screenshots/01_n8n_dry_run_success.png`
- `screenshots/02_lead_analysis_agent_output.png`
- `screenshots/03_parse_lead_analysis_output.png`
- `screenshots/04_personalization_agent_output.png`
- `screenshots/05_final_personalized_email_output.png`
- `workflow/final_personalized_email_output.json`

## 5. Results

Customer `10133` is a 44-year-old single graduate with no dependents, an income of 93,790, and a total spend of 2,349 across 25 purchases. Store purchases dominate (12), with catalog (7) and web (6) as secondary channels. Recency is low at 16 days. The customer accepted a previous campaign and has a historical response of 1. The rule-based baseline score came out at 95.

The Lead Analysis Agent returned an AI lead score of 88, a Medium Intent classification, a purchase likelihood of High, and a confidence level of Medium. This is an important result because the model did not simply copy the rule-based baseline score of 95. It still recognized the customer as commercially valuable, but it softened the intent classification because recent web engagement was low. It recommended engaging through personalized in-store offers, identifying high total spend, strong in-store behavior, and previous campaign acceptance as positive signals. Low recent web engagement appeared as a risk signal — not enough to downgrade the customer to low priority, but a relevant nuance the rule-based score would not have surfaced.

The Personalization Agent output:

- **Subject:** "Exclusive Discounts Await You at Our Wine Store!"
- **Preview text:** "Enjoy special offers tailored just for you on our finest wines."
- **Call to action:** "Visit Us In-Store for Exclusive Offers!"
- **Recommended channel:** Store
- **Follow-up timing:** 1 week from send

The subject line is functional rather than remarkable, but the channel selection and category alignment — wine, in-store — follow directly from the customer data. Compared to a generic campaign blast, the output reflects actual behavioral patterns.

## 6. Marketing Implications

The prototype points to four practical benefits for a marketing team that would adopt something like this.

The first is customer prioritization. The AI lead score and reasoning summary give a marketer a fast, structured starting point for deciding which customers to act on and why — without requiring them to read through raw CRM records.

The second is more genuine personalization. The generated email reflects the customer's preferred product category and preferred channel, rather than applying a generic template across a segment. That distinction matters less at small scale but compounds quickly across thousands of records.

The third is operational efficiency. Once the workflow is configured and tested, rerunning it for additional customers does not require rewriting the analysis or the email drafts from scratch. The structure scales; the content adapts.

The fourth is auditability. Because both agents return structured JSON, the outputs are parseable, storable, and comparable. That makes the system easier to integrate with existing CRM tools and reporting workflows, and easier to audit when something goes wrong.

The following comparison summarizes the difference between the traditional workflow and the prototype:

| Marketing task | Traditional workflow | AI-assisted workflow |
| --- | --- | --- |
| Customer review | Manual inspection of CRM records | Automated structured analysis |
| Lead prioritization | Rule of thumb or static score | Baseline score plus LLM interpretation |
| Message creation | Generic template or manual copywriting | Customer-specific email draft |
| Channel decision | Broad campaign assumption | Preferred-channel recommendation |
| Scalability | Limited by marketer time | Repeatable workflow structure |
| Governance need | Human review still required | Human review still required |

## 7. Limitations

Several limitations are worth being direct about.

The dataset is historical, not live. It represents customers from past campaigns, which makes the project credible as a CRM analytics use case but limits its relevance to real-time acquisition contexts where the data would look different.

The baseline score is rule-based and transparent, which is a deliberate choice, but it also means it should not be taken as a predictive benchmark. It is a starting point, not a ground truth.

Only one customer was run through the live AI pipeline. That was the right call given the scope of the project and cost considerations, but it means there is no systematic evaluation of output consistency across different customer profiles or segments.

The LLM outputs may vary between runs. In a production environment this would require additional controls — brand voice guidelines, compliance checks, fairness monitoring, hallucination prevention, and probably human review before any content reaches a customer.

Finally, the workflow does not currently send emails or write results to Google Sheets. The final output is stored as JSON and documented through screenshots. That is appropriate for a prototype, less so for anything approaching deployment.

## 8. Future Work

A natural next step would be connecting the workflow to a storage layer — Google Sheets, a CRM, or a simple database table — so that output records accumulate automatically and can be compared over time.

Running the workflow against the 30-customer sample in `data/n8n_sample_customers.csv` would also be revealing. The more interesting question is not whether the system works on a single high-value customer, but whether the outputs remain coherent and appropriately differentiated across high-, medium-, and low-likelihood profiles.

On the modeling side, the historical `Response` field in the dataset makes it possible to train a supervised prediction model. Comparing that against the rule-based baseline score and the LLM-assisted interpretation would give a clearer picture of where each approach adds value and where they fail.

The personalization layer could also be hardened with human approval steps, brand voice rules, compliance filters, and A/B testing infrastructure before anything close to production use.

## Conclusion

The prototype works in the sense that matters for this type of project: it takes structured customer data, produces a defensible intent assessment, and generates a personalized email output that reflects actual behavioral signals rather than demographic averages. That is not a small thing, even if the scale is limited.

The broader argument is not that AI replaces the marketer, but that the combination of workflow automation and LLM-based decision support can reduce the gap between available data and marketable action. A marketer using this system would spend less time writing and more time reviewing, adjusting, and deciding — which is probably a better use of their judgment.

The limits are real and the governance requirements are significant. AI-generated content sent to real customers without review and controls is a liability, not a feature. But within a well-designed process, the prototype demonstrates a practical direction for AI-assisted CRM and marketing automation that does not require a dedicated ML team to build.

## References

Davenport, T., Guha, A., Grewal, D., & Bressgott, T. (2020). How artificial intelligence will change the future of marketing. *Journal of the Academy of Marketing Science, 48*, 24-42. https://doi.org/10.1007/s11747-019-00696-0

Huang, M.-H., & Rust, R. T. (2018). Artificial intelligence in service. *Journal of Service Research, 21*(2), 155-172. https://doi.org/10.1177/1094670517752459

Kaggle. (n.d.). *Customer Personality Analysis*. https://www.kaggle.com/datasets/imakash3011/customer-personality-analysis

n8n. (n.d.). *Code node documentation*. https://docs.n8n.io/code/code-node/

n8n. (n.d.). *Manual Trigger node documentation*. https://docs.n8n.io/integrations/builtin/core-nodes/n8n-nodes-base.manualworkflowtrigger/

n8n. (n.d.). *OpenAI node documentation*. https://docs.n8n.io/integrations/builtin/app-nodes/n8n-nodes-langchain.openai/

Payne, A., & Frow, P. (2005). A strategic framework for customer relationship management. *Journal of Marketing, 69*(4), 167-176. https://doi.org/10.1509/jmkg.2005.69.4.167

Wedel, M., & Kannan, P. K. (2016). Marketing analytics for data-rich environments. *Journal of Marketing, 80*(6), 97-121. https://doi.org/10.1509/jm.15.0413

Zhang, Z., Rossi, R. A., Kveton, B., Shao, Y., Yang, D., Zamani, H., Dernoncourt, F., Barrow, J., Yu, T., Kim, S., Zhang, R., Gu, J., Derr, T., Chen, H., Wu, J., Wang, Z., Mitra, S., Lipka, N., Ahmed, N., & Wang, Y. (2024). *Personalization of large language models: A survey*. arXiv. https://arxiv.org/abs/2411.00027
