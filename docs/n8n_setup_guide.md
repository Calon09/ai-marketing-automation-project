# n8n Setup Guide

This guide builds the assignment workflow in a cost-controlled way.

The first prototype should process one customer only. Do not run the full dataset
inside n8n until the workflow is tested and approved.

## Cost Control Rules

- Keep the workflow inactive while building.
- Use a `Manual Trigger`, not a schedule or webhook.
- Use one customer from `workflow/test_customer_input.json` for the first test.
- Do not use AI Workflow Builder.
- Do not connect Gmail or any real email sender.
- Run OpenAI only after the non-AI version works.
- First live AI test should use one customer only, which means two LLM calls:
  one for lead analysis and one for personalization.

n8n Cloud pricing is based on workflow executions, and an execution is one full
workflow run regardless of the number of steps. OpenAI usage may also create API
costs through the configured OpenAI credential.

## Target Workflow

```text
Manual Trigger
  -> Customer Input
  -> Lead Analysis Agent
  -> Parse Lead Analysis
  -> Personalization Agent
  -> Parse Personalization
  -> Google Sheets Output or Screenshot Output
```

## Version 0: No-OpenAI Dry Run

Use this version first to validate the workflow shape without making API calls.

### Node 1: Manual Trigger

Add node:

- Node: `Manual Trigger`
- Purpose: start the workflow only when you manually select `Execute Workflow`.

### Node 2: Customer Input

Add node:

- Node: `Edit Fields (Set)`
- Name: `Customer Input`
- Mode: `JSON Output`
- Include in Output: `All Input Fields`

Do not use `Customer Datastore (n8n training)`. That node belongs to n8n's
training examples and is not needed for this assignment.

Paste the content of:

- `workflow/test_customer_input.json`

This creates one test customer.

### Node 3: Mock Lead Analysis

Add node:

- Node: `Edit Fields (Set)`
- Name: `Mock Lead Analysis`
- Mode: `JSON Output`
- Include in Output: `All Input Fields`

Paste:

```json
{
  "ai_lead_score": 94,
  "intent_segment": "High Intent",
  "purchase_likelihood": "High",
  "recommended_action": "Send a premium personalized offer focused on the customer's preferred category.",
  "recommended_offer_type": "Premium loyalty offer",
  "reasoning_summary": "The customer shows high spend, recent activity, strong purchase volume, and previous campaign acceptance.",
  "confidence_level": "High"
}
```

### Node 4: Mock Personalization

Add node:

- Node: `Edit Fields (Set)`
- Name: `Mock Personalization`
- Mode: `JSON Output`
- Include in Output: `All Input Fields`

Paste:

```json
{
  "email_subject": "A premium wine selection picked for you",
  "email_preview_text": "Explore a tailored offer based on your recent preferences.",
  "email_body": "We noticed your strong interest in wine selections and wanted to share a premium offer tailored to your preferences. This collection highlights quality, variety, and value for customers who enjoy discovering standout bottles. Take a look at the curated selection and choose the wines that fit your next occasion.",
  "call_to_action": "View your personalized wine offer",
  "personalization_strategy": "High-intent customer with strong wine preference and high purchase value.",
  "recommended_channel": "Email",
  "follow_up_timing": "Within 24 hours"
}
```

Run the workflow manually once. If the final node shows one customer plus the
mock fields, the structure is ready.

## Version 1: OpenAI Live Prototype

Only do this after Version 0 works.

### Node 3: Lead Analysis Agent

Replace `Mock Lead Analysis` with:

- Node: `OpenAI`
- Name: `Lead Analysis Agent`
- Operation: `Text -> Generate a Model Response` or `Text -> Generate a Chat Completion`
- Credential: use only the class-approved OpenAI credential.
- Model: use the lowest-cost approved model available in the account.
- Prompt/User message: paste the full content of `prompts/lead_scoring_prompt.txt`.

Keep the instruction requiring valid JSON.

Run this node for one customer. It should return a JSON object with fields such
as `ai_lead_score`, `intent_segment`, and `recommended_action`.

### Node 4: Parse Lead Analysis

Add node:

- Node: `Code`
- Name: `Parse Lead Analysis`
- Language: `JavaScript`
- Mode: `Run Once for All Items`

Paste:

```javascript
function findJsonString(value) {
  if (typeof value === 'string') {
    const trimmed = value.trim();
    if (trimmed.startsWith('{') && trimmed.endsWith('}')) {
      return trimmed;
    }
    const start = trimmed.indexOf('{');
    const end = trimmed.lastIndexOf('}');
    if (start !== -1 && end !== -1 && end > start) {
      return trimmed.slice(start, end + 1);
    }
  }

  if (Array.isArray(value)) {
    for (const item of value) {
      const found = findJsonString(item);
      if (found) return found;
    }
  }

  if (value && typeof value === 'object') {
    for (const item of Object.values(value)) {
      const found = findJsonString(item);
      if (found) return found;
    }
  }

  return null;
}

const customer = $("Customer Input").all()[0].json;
const openAiOutput = $("Lead Analysis Agent").all()[0].json;

let leadAnalysis;
if (openAiOutput.ai_lead_score) {
  leadAnalysis = openAiOutput;
} else {
  const rawJson = findJsonString(openAiOutput);
  if (!rawJson) {
    throw new Error('Could not find JSON in Lead Analysis Agent output.');
  }
  leadAnalysis = JSON.parse(rawJson);
}

return [
  {
    json: {
      ...customer,
      ...leadAnalysis
    }
  }
];
```

This merges the original customer data with the first AI agent output.

### Node 5: Personalization Agent

Add node:

- Node: `OpenAI`
- Name: `Personalization Agent`
- Operation: `Text -> Generate a Model Response` or `Text -> Generate a Chat Completion`
- Credential: same approved OpenAI credential.
- Model: same low-cost approved model.
- Prompt/User message: paste the full content of `prompts/personalization_prompt.txt`.

Because this node receives the merged output from `Parse Lead Analysis`, the
prompt variables such as `{{$json.ai_lead_score}}` and `{{$json.preferred_category}}`
should resolve correctly.

### Node 6: Parse Personalization

Add node:

- Node: `Code`
- Name: `Parse Personalization`
- Language: `JavaScript`
- Mode: `Run Once for All Items`

Paste:

```javascript
function findJsonString(value) {
  if (typeof value === 'string') {
    const trimmed = value.trim();
    if (trimmed.startsWith('{') && trimmed.endsWith('}')) {
      return trimmed;
    }
    const start = trimmed.indexOf('{');
    const end = trimmed.lastIndexOf('}');
    if (start !== -1 && end !== -1 && end > start) {
      return trimmed.slice(start, end + 1);
    }
  }

  if (Array.isArray(value)) {
    for (const item of value) {
      const found = findJsonString(item);
      if (found) return found;
    }
  }

  if (value && typeof value === 'object') {
    for (const item of Object.values(value)) {
      const found = findJsonString(item);
      if (found) return found;
    }
  }

  return null;
}

const mergedCustomer = $("Parse Lead Analysis").all()[0].json;
const openAiOutput = $("Personalization Agent").all()[0].json;

let personalization;
if (openAiOutput.email_subject) {
  personalization = openAiOutput;
} else {
  const rawJson = findJsonString(openAiOutput);
  if (!rawJson) {
    throw new Error('Could not find JSON in Personalization Agent output.');
  }
  personalization = JSON.parse(rawJson);
}

return [
  {
    json: {
      ...mergedCustomer,
      ...personalization
    }
  }
];
```

This final output is the main result to screenshot for the assignment.

## Optional Output: Google Sheets

Add this only after the AI prototype works with one customer.

Add node:

- Node: `Google Sheets`
- Name: `Save Output`
- Operation: `Sheet Within Document -> Append Row`

Suggested output columns:

- `customer_id`
- `baseline_lead_score`
- `ai_lead_score`
- `intent_segment`
- `purchase_likelihood`
- `recommended_action`
- `recommended_offer_type`
- `email_subject`
- `email_preview_text`
- `email_body`
- `call_to_action`
- `recommended_channel`
- `follow_up_timing`
- `confidence_level`

If Google Sheets credentials are inconvenient, skip this node and use screenshots
of `Parse Personalization` as the final evidence.

## Screenshots To Capture

Save screenshots in `screenshots/`:

- Full n8n workflow canvas.
- `Customer Input` node output.
- `Lead Analysis Agent` output.
- `Parse Lead Analysis` merged output.
- `Personalization Agent` output.
- Final email output from `Parse Personalization`.
- Google Sheets output, if used.

## Scaling Later

After the one-customer test works:

1. Test 3 customers.
2. Test 10 customers.
3. Test the full 30-row sample only if needed.

Do not run `data/marketing_campaign_clean.csv` with 2212 rows in n8n for this
assignment. It is not necessary and could create unnecessary execution and API
usage.

## Official References

- n8n pricing and executions: https://n8n.io/pricing/
- Manual Trigger node: https://docs.n8n.io/integrations/builtin/core-nodes/n8n-nodes-base.manualworkflowtrigger/
- Manual and production executions: https://docs.n8n.io/workflows/executions/manual-partial-and-production-executions/
- Edit Fields node: https://docs.n8n.io/integrations/builtin/core-nodes/n8n-nodes-base.set/
- OpenAI node: https://docs.n8n.io/integrations/builtin/app-nodes/n8n-nodes-langchain.openai/
- Code node: https://docs.n8n.io/code/code-node/
- Google Sheets node: https://docs.n8n.io/integrations/builtin/app-nodes/n8n-nodes-base.googlesheets/
