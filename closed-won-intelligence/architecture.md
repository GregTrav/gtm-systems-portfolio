[← Case study](README.md)

# Data architecture

The system separates source data, standard preparation rules, AI interpretation, and reviewed results. Each layer has a defined job.

## Flow and design choices

```text
RAW SOURCES
├── Salesforce
│   ├── Structured deal, account and contact fields
│   └── Rep-written notes
└── Gong
    ├── Raw call transcripts, not generated summaries
    └── Call participants
        │
        ▼

ELIGIBILITY                              Which deals get processed?
├── Shared scope rule                    Define scope once.
├── Recorded exclusions                  Keep judgment separate from filtering.
└── Selection by processing step         Select deals missing from that step's output.
        │
        ▼

DEAL RECORD                              What are we counting?
└── One row per deal                     Prepare standard fields without AI.
        │
        ├── OPPORTUNITY CLASSIFICATION   What came before? Why now?
        │   ├── Previous approach        Derived from opportunity data.
        │   └── Reason for urgency       Two separate AI classifications.
        │                                Results join the final deal record.
        │
        ├── REP-WRITTEN NOTES ──────────────────────────┐
        │                                              │
        └── TRANSCRIPT SELECTION                        │
            ├── Date-ordered call sampling              │
            └── Account-call fallback                   │
                │                                      │
                │  Which calls matter?                  │
                │  Select calls before AI extraction.   │
                │  Use account calls only when no       │
                │  opportunity-linked calls exist.      │
                │                                      │
                └──────────────────┬───────────────────┘
                                   ▼

PAIN EXTRACTION                          What does the evidence say?
├── Separate CRM and transcript inputs   Preserve the distinction between sources.
├── Individual pain records              Make pains countable across deals.
└── Speaker, quote and deal ID           Attach the deal ID during extraction.
        │
        ▼

SAVED OUTPUTS                            Can the result be inspected?
├── Original model response              Save before parsing.
├── Parsed pain records                  Parser fixes reuse the saved response.
└── Quote-in-source result               Exclude failing quotes from review.
        │
        ▼

THEME CLASSIFICATION                     How do pains become comparable?
└── Shared label-to-theme mapping        Classify each distinct label once.
                                         Review the mapping before reuse.
                                         One correction updates all matching pains.
        │
        ▼

DEAL SYNTHESIS                           What was the deal as a whole?
├── Narrative and capability bought      Three independent AI passes.
├── Champion and economic buyer          Separate rules and output tables.
└── Competition                          Change one pass without repeating the others.
        │
        ▼

HUMAN REVIEW                             Is the evidence usable?
├── Quote selection and tightening       Retain the original source passage.
└── Ambiguous findings                   Record the reviewer.
        │
        ▼

CURATED CLOSED-WON RECORDS               Can records be combined safely?
├── One main record per deal             Group or filter supporting records
└── Linked pains and evidence            before joining them to the deal.
        │
        ▼

ANALYSIS AND QUOTE LIBRARY                Where does AI stop?
└── Standard reporting queries           No AI inside the analysis views.


CHECKS THROUGHOUT AI PROCESSING
├── After each model step: operator checks for missing responses and keys.
├── When prompts change: test known deals with known answers.
└── Failed rows: investigate and rerun manually.
```

The diagram shows the main processing stages. Opportunity classification reads Salesforce opportunity data. Pain extraction reads both rep-written notes and selected transcripts; those inputs remain distinguishable.

## Workflow design and processing controls

- **Consistent business rules:** One shared scope rule keeps every processing step aligned. This resolved inconsistent deal selections caused by separate filters.
- **Visible exceptions:** Judgment-based exclusions retain a reason and author, keeping business decisions traceable.
- **Repeatable processing:** Each step selects eligible deals missing from its own output. New deals are added while previously reviewed work is preserved.
- **Operational visibility:** A progress view shows deals awaiting curation but does not control processing. Failed rows are investigated and rerun manually.

## Data preparation and reliable reporting

- **Controlled inputs:** Calls are ordered by date and divided into ten groups by call count. The call with the most transcript segments is selected from each group; deals with fewer than ten calls retain all calls.
- **Defined fallback:** Account-linked calls are used when no opportunity-linked calls exist.
- **Consistent deal counts:** Each deal has one main record. Calls, pains, and people are grouped or filtered before joining it, preventing supporting records from inflating reporting totals.

## AI workflow design and evidence quality

- **Inspectable outputs:** Original extraction responses are saved before parsing. Parser fixes reuse those responses without another model call.
- **Source-based safeguards:** Extracted quotes are checked against their source, and failing quotes are excluded from review. Human review still checks meaning and speaker attribution.
- **Reusable classifications:** Each distinct pain label is mapped once to a shared theme list. Correcting the mapping updates every matching pain record.
- **Maintainable AI steps:** Narrative and capability, stakeholders, and competition use separate scripts and output tables. One pass can change without repeating correct work in the others.

## Evaluation and ongoing maintenance

- **Clear responsibility for checks:** Parsing and quote matching are automatic. The operator checks for missing responses and keys after each model step.
- **Controlled prompt changes:** Updated prompts are tested against known deals with known answers. Previous AI output is not treated as the answer key because responses can vary between runs.
- **Durable corrections:** The curated deal record stores copied values. Corrections update both that record and the saved synthesis result so a later rebuild preserves the fix.
