# Research Notes

This file summarizes sources that support the final report. The project uses
these sources to connect the implemented n8n prototype with broader marketing
automation, CRM analytics, personalization, and AI research.

## Davenport, Guha, Grewal, and Bressgott (2020)

Topic:

- Artificial intelligence and the future of marketing.

Key ideas:

- AI can affect marketing strategy, customer behavior, sales processes,
  customer service, and personalization.
- The article frames AI through task types and levels of intelligence.
- It highlights privacy, bias, and ethics as important constraints.
- It argues that AI is often most valuable when it augments human managers
  rather than replacing them.

Use in report:

- Supports the argument that the prototype should be treated as decision
  support for marketers, not as a fully autonomous replacement.
- Supports the discussion of ethical and governance limitations.

## Wedel and Kannan (2016)

Topic:

- Marketing analytics in data-rich environments.

Key ideas:

- Marketing analytics uses structured and unstructured data to support
  marketing decisions.
- Important application areas include marketing-mix decisions, personalization,
  privacy, and data security.
- Organizations need processes and capabilities to turn data into actionable
  marketing decisions.

Use in report:

- Supports the use of customer profile, purchase, channel, and campaign data as
  inputs for scoring and personalization.
- Supports the claim that analytics becomes more useful when it is connected to
  decision workflows.

## Payne and Frow (2005)

Topic:

- Strategic customer relationship management.

Key ideas:

- CRM should be treated as a cross-functional process, not only as software.
- Their framework includes strategy development, value creation, multichannel
  integration, information management, and performance assessment.
- CRM success depends on combining customer data with operational processes.

Use in report:

- Supports the CRM-style framing of the workflow.
- Helps explain why the project connects segmentation, channel preference,
  personalized communication, and potential output storage.

## Huang and Rust (2018)

Topic:

- Artificial intelligence in service.

Key ideas:

- AI can perform mechanical, analytical, intuitive, and empathetic service
  tasks to different degrees.
- AI replacement happens at the task level before the job level.
- Analytical tasks are especially suitable for automation and augmentation.

Use in report:

- Supports the design choice to automate structured analytical tasks such as
  customer scoring and message drafting, while leaving real campaign deployment
  to human review.

## Zhang et al. (2024)

Topic:

- Personalization of large language models.

Key ideas:

- LLM personalization includes both personalized text generation and downstream
  personalization applications such as recommendations.
- The survey highlights challenges around evaluation, granularity of
  personalization, and responsible use.

Use in report:

- Supports the use of an LLM to adapt message content to customer-level context.
- Supports the limitation that personalized LLM output should be evaluated
  before real customer use.

## Kaggle Customer Personality Analysis Dataset

Topic:

- Customer demographics, purchasing behavior, campaign history, and response
  data.

Key ideas:

- Provides realistic customer features for campaign analytics.
- Allows the project to move beyond a fully simulated lead form.
- Includes behavioral and historical response variables that can support
  segmentation and future predictive modeling.

Use in report:

- Supports the methodology section and the explanation of why CRM-style data is
  more realistic than purely simulated form data.

## n8n Documentation

Topic:

- Manual workflows, core nodes, OpenAI node, Code node, and execution control.

Key ideas:

- Manual triggers make it possible to test workflows without schedules.
- Code nodes can parse and transform structured outputs.
- OpenAI nodes can be integrated into automated workflows.

Use in report:

- Supports the technical implementation section and the cost-control decisions.
