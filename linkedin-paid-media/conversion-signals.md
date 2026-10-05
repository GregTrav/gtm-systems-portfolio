[← LinkedIn overview](README.md) · [Portfolio](../README.md)

# Sending downstream conversion data to LinkedIn

I built an integration that prepared conversion data in Snowflake and sent it through Boomi to the LinkedIn Conversions API (CAPI). Events included marketing-qualified leads (MQLs), qualified opportunities (QOs), and closed-won deals.

This gave the team visibility into downstream outcomes in LinkedIn and supplied signals for optimizing campaigns toward qualified leads and revenue, beyond clicks and initial form submissions.

## Data preparation and delivery

| Layer | What I implemented |
| --- | --- |
| Snowflake views | Conversion definitions, email normalization and hashing, timestamp formatting, and event selection. |
| Boomi integration | API payload mapping, delivery, throttling, and retries. |
| Snowflake send ledger | Record confirmed successful events and exclude them from subsequent sends. |

## Key decisions

**Prepare data in Snowflake.** Hashing and timestamp conversion ran in views so I could inspect the prepared values before executing the integration. Boomi handled payload mapping and delivery.

**Keep recovery controls separate.** A lookback window captured late-arriving data. Throttling respected API throughput. Retries helped recover failed attempts during the same run. The ledger controlled repeat sends across runs; it did not replace retries or throttling.

**Record success after confirmation.** Events entered the ledger after the API confirmed successful delivery. This added a database-write dependency, but gave us an independent delivery history. API acceptance and recording success remain separate operations, so the design does not guarantee exactly-once delivery.

## Investigating inflated campaign totals

Despite healthy integration checks, the paid-media owner questioned LinkedIn conversion totals against Salesforce. I investigated both attribution settings and event delivery.

- **Attribution:** Broad settings could credit campaign interactions over a long period. I narrowed settings for relevant conversion types and explained their effect with the stakeholder.
- **Delivery:** I added the ledger to establish which events had been successfully sent, independently of the campaign credit LinkedIn assigned.

Broad attribution was my leading explanation for the discrepancy. Duplicate counting was not established as the cause.

## Results

Send volume fell, and the team gained a historical record of accepted events. We could distinguish delivery questions from attribution questions when investigating discrepancies. The ledger's individual effect on campaign performance was not measured.
