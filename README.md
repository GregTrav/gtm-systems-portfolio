# GTM systems portfolio

I'm Greg Travers, a GTM systems practitioner who turns ambiguous revenue problems into reliable data models, integrations, and AI-assisted workflows. My work starts with stakeholder discovery and continues through implementation, testing, and iteration with the people using it.

## Selected work

### [LinkedIn paid-media intelligence](linkedin-paid-media/README.md)

A system for understanding performance across hundreds of ads: creative tagging, goal-appropriate comparisons, conversational analysis, and downstream conversion delivery.

**My contribution:** Discovery and manual validation through downstream design, implementation, and ongoing stakeholder iteration. Data engineering owned raw ingestion; the paid-media owner made campaign and budget decisions.

**Outcomes:** Reduced manual reporting and informed meaningful improvements in lead volume, cost efficiency, creative decisions, and budget allocation. These program results reflect the system alongside targeting and campaign changes.

**Built with:** Snowflake (Cortex), an integration platform, LinkedIn CAPI, and Slack.

### Google Ads Agent

An agent for exploring Google Ads performance across search and display campaigns, keywords, and search terms. Built with Snowflake Cortex, a semantic model, and verified queries, it lets the stakeholder ask questions and review detailed results directly in Snowflake.

Case study coming soon.

### [Lead eligibility recovery workflow](marketo-dq/README.md)

An agent-assisted workflow that returned hundreds of legitimately eligible records to marketable status. It combines web research, human review, and controlled API writeback in Marketo, with AI-assisted coding used during development and iteration.

### [Closed-won intelligence](closed-won-intelligence/README.md)

A repeatable system that turns sales records and customer conversations into structured evidence about why customers bought—helping marketing ground messaging and campaigns in the problems behind revenue.

## Exploratory labs

Independent prototypes I designed and tested to explore how public evidence, automation, and AI could support GTM work. These were not internal production deployments. They demonstrate the hypotheses, architecture, testing, and trade-offs behind the systems.

### [Account signal monitoring lab](account-signal-monitoring/README.md)

A personal experiment in determining whether public company activity could reveal product-relevant needs across a defined account list.

Rather than ranking companies merely because they were hiring, the system evaluated how the work described in public postings related to problems a product category could address. It retained the underlying evidence and produced a ranked digest for human review.

**Built with:** Python, SQLite, public job and news sources, and AI-assisted coding.

### [Account research to personalized video at scale](personalized-video-outbound/README.md)

An implementation and extension of a personalized-video concept: research accounts at scale, develop an evidence-backed message for each one, and use voice cloning, an AI avatar, and Python-based composition to produce a distinct video of the seller speaking to every account.

**[View the interactive system flow and watch the 54-second demo →](https://gregtrav.github.io/gtm-systems-portfolio/)**

**Built with:** Clay, Codex, ElevenLabs, HeyGen, Python, Chrome DevTools, and ffmpeg.

## Writing

[The Hidden Work Behind Reliable GTM AI](https://www.linkedin.com/pulse/hidden-work-behind-reliable-gtm-ai-greg-travers-b7nmc/)

How business definitions, data preparation, and testing shape useful GTM AI systems.

---

These case studies describe my contribution using generalized architecture and operating details. They exclude source code, underlying records, customer identities, credentials, and internal URLs.
