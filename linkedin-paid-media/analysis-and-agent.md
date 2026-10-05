[← LinkedIn overview](README.md) · [Portfolio](../README.md)

# Ad analysis, creative tagging, and conversational access

I built the Snowflake data models, creative classification, and conversational agent that let the paid-media owner compare ad performance by region, audience, and message. A Slack application connected through an integration platform made that analysis accessible through questions and follow-ups.

I turned unstructured ad copy into consistent creative attributes, allowing the team to compare cost per lead by theme, value proposition, tone, and call to action.

## What I built

Data engineering owned raw ingestion. I built the downstream components:

| Component | Purpose |
| --- | --- |
| Curated models and analytical views | Combine performance with creative; derive attributes from naming conventions and URLs. |
| AI creative tagging | Classify theme, value proposition, tone, and call to action so they can be compared across ads. |
| Metric and volume rules | Match comparisons to campaign goals, regional budgets, and available activity. |
| Cortex agent | Answer questions using a semantic model of business definitions and verified example queries. |
| Slack and integration-platform workflow | Receive questions, call Cortex, and return answers with conversational follow-up. |

## Key decisions

**Validate before automating.** I joined one month of performance and creative data in a spreadsheet, added rule-based and AI attributes, and reviewed pivot-table findings with the stakeholder. The taxonomy developed through sample exploration, testing, and review before wider tagging.

**Compare ads against their intended goals.** An early response recommended stopping a retargeting ad for generating no leads. I corrected the metric guidance in verified queries and agent instructions, then reran questions. Regional comparisons reflected how budgets were managed.

**Withhold weak trends.** Volume thresholds determined whether recent performance could be compared with a longer baseline. Thin samples received a cumulative view or no trend label. This could hide emerging signals, but reduced recommendations based on too little activity.

**Prioritize interactive analysis.** A scheduled analyst/strategist workflow struggled to cover every region and ad type within its output limit. We retired it in favor of stakeholder-led questions, giving up the proactive weekly brief.

**Separate acknowledgement from execution.** The integration listener acknowledged Slack requests promptly while a separate process handled the slower Cortex response.

## Validation and results

Stakeholder reviews and repeated test questions guided changes to metrics, comparisons, and output usefulness. These checks did not establish a formal system-wide accuracy score or statistical significance for every trend.

The system substantially reduced recurring reporting work. Analysis informed budget reallocation and helped identify signs of creative fatigue in retargeting ads, prompting spend changes. [See the program results and supporting context](README.md#evidence-and-outcomes).
