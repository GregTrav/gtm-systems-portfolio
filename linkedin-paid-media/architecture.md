[← Case study](README.md)

# Data architecture

The system separates source data, deterministic preparation rules, AI interpretation, and delivery workflows. Snowflake holds the data models and saved outputs; an integration platform connects the agent to Slack and sends conversion events to LinkedIn.

My work covers the SQL models, creative tagging, conversational agent, and API integrations. Data engineering owned upstream ingestion. The paid-media owner reviewed the analysis and made campaign and budget decisions.

## Performance analysis

```text
SOURCE DATA                           What did LinkedIn report?
├── Daily ad metrics
├── Creative text and landing pages
└── Campaign metadata
        │
        ▼
CURATED AD BASE                       What can fixed rules establish?
└── Deterministic, rule-based attributes
    Region, segment, ad type, targeting,
    copy length, content type and scope.
        │
        ├─────────────────────────────────────┐
        │                                     ▼
        │                          AI CREATIVE CLASSIFICATION
        │                          What does the creative communicate?
        │                          ├── Theme, tone and value proposition
        │                          └── Hook, CTA and headline style
        │                                     │
        │                                     ▼
        │                          STAGING AND MANUAL CHECKS
        │                          Is the output ready to save?
        │                          └── Check output and record counts.
        │                                     │
        │                                     ▼
        │                          SAVED CREATIVE TAGS
        │                          What can we inspect and reuse?
        │                          └── Labels, rationale and run details
        │                                     │
        ▼                                     │
AD-BY-DAY ANALYST VIEW ◄───────────────────────┘
What did each ad do, with its attributes?
└── Join daily metrics, rule-based attributes and saved AI tags.
        │
        ├── Section rollups       How is each comparison group doing?
        └── Ranked-ad rollups     Which ads warrant investigation?

Ad-by-day view + both rollups
        │
        ▼
SEMANTIC MODEL                        How should the data be queried?
├── Metric and grain definitions      Keep comparisons consistent.
└── Human-verified query examples     Guide the agent's queries.
        │
        ▼
CONVERSATIONAL AGENT                  What does the evidence suggest?
└── Query and explain results         Recommendations remain provisional.
        │
        ▼
PAID-MEDIA OWNER                      What action should we take?
└── Review through Slack              Campaign and budget decisions stay human.
```

## Data preparation and reliable reporting

- **Rules before AI:** Campaign metadata, word counts, and URL patterns establish standard attributes. AI classifies qualitative creative features that require interpretation.
- **Defined reporting grain:** The analyst view has one row per ad per day. Section rollups group by region, segment, and ad type; ranked-ad rollups identify ads within those sections. The semantic model guides the agent to keep these grains distinct.
- **Objective-aware comparisons:** Metrics follow campaign goals, so lead-generation and conversion-focused ads are not judged by the same outcome. Regional comparison boundaries are preserved, with a defined global grouping for sponsored content.
- **Volume-aware signals:** Recent and baseline windows support comparisons. Volume rules govern selected performance flags and rankings; a trend label alone does not establish sufficient evidence.
- **Consistent calculations:** Aggregate rates use summed results and their summed denominators rather than averages of daily rates. Cost per lead and cost per conversion remain separate metrics.

## AI classification and saved outputs

- **Bounded interpretation:** The tagging prompt defines label lists, descriptions, and tie-break rules for six creative dimensions. These guide classification; they are not an enforced vocabulary validation gate.
- **Reuse within a run:** One representative per unique creative text is classified, then its tags are copied to matching ads in that run. This reduces repeated model calls; it does not guarantee identical labels across separate runs.
- **Inspectable results:** Saved tags include a rationale, taxonomy version, model reference, text fingerprint, and timestamp. They remain separate from source data and join the analyst view by ad.
- **Operator checks:** Tagging is run manually, with output and count checks around staging and insertion. These checks assess processing completeness, not classification accuracy.

## Slack delivery and conversation state

```text
SLACK QUESTION                       Where should the answer return?
└── Capture question, channel, thread Preserve the reply destination.
        │
        ▼
LISTENER                             How do we accept work promptly?
├── Publish request to event queue   Separate receipt from analysis.
└── Return HTTP acknowledgement      Confirm receipt to Slack.
        │
        ▼
WORKER                               Which conversation should continue?
├── Look up saved thread mapping     Reuse the linked agent conversation.
└── Create conversation if missing   Establish context for a new thread.
        │
        ▼
AGENT API CALL                       What does the data suggest?
└── Run analysis in that conversation Support follow-up questions.
        │
        ▼
SLACK REPLY AND SAVED STATE           How do we keep the exchange connected?
├── Format and post to Slack thread  Return the answer where it was asked.
└── Save or update thread mapping    Link later questions to the conversation.
```

- **Separate processing responsibilities:** The listener accepts requests and acknowledges receipt; a separate worker handles the longer agent call. The acknowledgement is an HTTP response to Slack, not a visible progress message.
- **Explicit conversation state:** A mapping stored in Snowflake links each Slack thread to its agent conversation, allowing follow-up questions to reuse context.
- **API coordination:** The integration carries the question and reply destination between Slack and the agent, formats the response, and posts it in the originating thread.
- **Visible failure handling:** A failed agent call returns a “try again” message. Automatic agent retries and queue redelivery are not established by the available evidence.

## LinkedIn conversion API process

```text
CRM OUTCOMES                          Which outcomes qualify?
└── Eligibility rules in Snowflake    Apply business scope before delivery.
        │
        ▼
PREPARED EVENTS                       What will we send?
├── Event identity                    Assign a consistent event ID.
└── Hashed contact identity           Prepare matching fields.
        │
        ▼
EVENT SELECTION                       Which events still need delivery?
├── Watermark with lookback overlap   Revisit recent events.
└── Successful-send exclusion         Remove events already recorded as sent.
        │
        ▼
SCHEDULED INTEGRATION                  How do we control delivery?
├── Payload mapping                   Translate events to the API format.
├── Throttling                        Keep delivery below the API rate limit.
├── Retries and error notifications   Handle failures and surface exceptions.
└── Response check                    Save only successful sends.
        │
        ▼
LINKEDIN CONVERSION API               Was the send successful?
        │
        ▼
SUCCESSFUL-SEND TABLE                 What can we verify from our end?
└── Record successful API responses   Query delivered events in Snowflake.
        │
        └──► Exclude these events from later runs.
```

- **Controlled delivery:** Throttling keeps sends below the API rate limit, with retries, error handling, and notification steps on failure paths.
- **Next-run recovery:** Unsuccessful sends are not recorded as sent. The overlapping lookback makes them eligible for the following daily run while they remain within that window.
- **Queryable delivery records:** Successful API responses are recorded in Snowflake, providing a record of events sent successfully from our end.
- **Reconciliation:** These records support comparison with conversion counts shown in the LinkedIn UI when investigating reporting differences.

## Evaluation and ongoing maintenance

- **Query guidance:** Human-verified query examples guide the agent's use of the semantic model. They are not scheduled regression tests, and prompted instructions are not equivalent to SQL enforcement.
- **Human judgment:** The paid-media owner evaluates answers and recommendations. The agent supports investigation; it does not change campaigns or budgets.
- **Maintenance needs:** Naming conventions, URL patterns, taxonomy definitions, and agent instructions require updates as campaigns and stakeholder needs change. Manual tagging checks remain part of operation.
- **Human evaluation during development:** Initial testing and ramp-up included substantial human review, judgment, and accuracy testing of creative classifications. This evaluation informed refinement before ongoing use.
- **Further evaluation controls:** The development testing is distinct from a formal, repeatable regression set. Automated vocabulary validation and a classification regression set are improvement opportunities, not implemented controls.
- **Reporting boundary:** The analysis uses LinkedIn-reported performance. It does not establish pipeline or revenue attribution by ad.
