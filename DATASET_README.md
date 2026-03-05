# ZenTrades AI — Test Dataset Documentation

*A guided tour of the synthetic data powering this project, written for the people evaluating it.*

---

## What Is This Folder and Why Does It Exist?

The assignment asked me to build two automated pipelines that process call transcripts and generate structured outputs. Before plugging in real client data, I needed a controlled dataset to prove the pipelines actually work.

So I built one from scratch.

This folder contains **10 synthetic call transcripts** — 5 demo calls and 5 onboarding calls — written to mimic real conversations between a Clara AI sales rep and a trade business owner. Each transcript covers a different industry (HVAC, plumbing, fire protection, electrical, pest control) and each onboarding call introduces deliberate changes to the demo call data, so the versioning and diff logic in Pipeline B gets a proper workout.

The expected outputs are pre-computed and stored here too. You can delete them, run the pipeline yourself, and compare. If the outputs match, the pipeline is working correctly.

---

## Folder Layout

```
ZenTrades AI/
├── inputs/
│   ├── demo/
│   │   ├── demo_001_arctic_hvac.txt                 → ACC-001
│   │   ├── demo_002_riverstone_plumbing.txt         → ACC-002
│   │   ├── demo_003_fireguard.txt                   → ACC-003
│   │   ├── demo_004_volt_masters.txt                → ACC-004
│   │   └── demo_005_shieldpest.txt                  → ACC-005
│   └── onboarding/
│       ├── onboarding_001_arctic_hvac.txt           → ACC-001
│       ├── onboarding_002_riverstone_plumbing.txt   → ACC-002
│       ├── onboarding_003_fireguard.txt             → ACC-003
│       ├── onboarding_004_volt_masters.txt          → ACC-004
│       └── onboarding_005_shieldpest.txt            → ACC-005
└── outputs/
    ├── accounts/
    │   ├── ACC-001_memo_v1.json
    │   ├── ACC-001_memo_v2.json
    │   └── ... (through ACC-005, both versions)
    └── agent_spec/
        ├── 001_arctic_hvac_agent_spec_v1.json
        └── ... (through 005)
```

The outputs folder is pre-filled with expected results. To test the pipeline from scratch, delete the outputs and hit run. They should regenerate identically.

---

## The Five Accounts

Each account was designed with a specific complexity so the pipeline has to handle more than just the easy cases.

| Account | Company | Industry | What Makes It Interesting |
|---|---|---|---|
| ACC-001 | Arctic Comfort HVAC Solutions | HVAC | Rotating on-call schedule, seasonal staffing changes |
| ACC-002 | Riverstone Plumbing and Drain | Plumbing | Spanish-speaking caller handling, multiple emergency triggers |
| ACC-003 | FireGuard Protection Services | Fire Protection | Legal constraints, authority jurisdiction escalations, hard-banned words |
| ACC-004 | Volt Masters Electrical | Electrical | VIP client routing, a company rebrand mid-project, routing order swap |
| ACC-005 | ShieldPest Control Group | Pest Control | Zone-based dispatch by zip code, full platform migration, weekend shutdown |

---

## What Pipeline A Should Produce (Demo Calls)

Pipeline A reads each demo transcript and extracts a structured account memo in JSON. The memo is the foundation everything else builds on. A correctly functioning pipeline will pull out all of the following without making anything up:

- Company name, address, and business hours with timezone
- Full list of services offered
- Every emergency trigger the client described
- Emergency routing rules including primary contact, secondary contact, timeout durations, and what to do if both fail
- Non-emergency after-hours instructions
- Business hours transfer rules
- Software the agent must never interact with
- Phrases the agent must never say
- The exact greeting the client requested
- A summary of both the office hours call flow and the after-hours call flow
- Any information that was genuinely missing from the transcript, flagged honestly rather than guessed

It then takes that memo and generates a full agent configuration spec, which includes a complete system prompt the Retell agent will use when it goes live.

---

## What Pipeline B Should Produce (Onboarding Calls)

Pipeline B reads each onboarding transcript, identifies what changed since the demo call, and updates the account accordingly. The result is a v2 memo, a v2 agent spec, and a changelog explaining every modification.

Here is a summary of the changes each onboarding call introduces, so you can verify the pipeline caught all of them:

| Account | Changes From v1 to v2 |
|---|---|
| ACC-001 | New primary on-call technician; two new services added; commercial boiler added as emergency type; VIP routing for maintenance contract customers; website URL added to after-hours close |
| ACC-002 | Saturday hours added; emergency line number updated; gas smell added as an emergency with a specific evacuation instruction; two new services; greeting reworded for legal safety |
| ACC-003 | Secondary on-call number corrected; AHJ violation added as emergency with a direct escalation path that bypasses normal dispatch; kitchen hood suppression added to services; Friday hours extended; the word "guarantee" hard-banned; greeting updated |
| ACC-004 | Emergency call order swapped (Aaron first, then Linda); Saturday hours added; VIP routing created for a specific commercial client; solar and battery services added; company name shortened following a rebrand; Aaron's title updated to co-owner |
| ACC-005 | All weekend hours removed entirely; zone dispatch structure rebuilt with a new technician; hospital and elder care facility added as a high-priority emergency type; wildlife removal added to services; integration platform migrated from FieldRoutes to ServiceTitan; greeting updated |

---

## How to Verify the Pipeline Is Working Correctly

### For Pipeline A, check that each output:

- Contains the correct company name, address, and hours with no invented details
- Lists all emergency triggers mentioned in the transcript (no more, no less)
- Has the correct phone numbers and timeout values in the routing rules
- Names the correct software in the integration constraints
- Does not include any of the forbidden phrases in the generated agent prompt
- Uses exactly the greeting the client asked for
- Flags any missing information instead of hallucinating a plausible answer

### For Pipeline B, check that each v2 output:

- Correctly links to the right account ID
- Captures every change listed in the table above (the change counts per account are: ACC-001 has 5 changes, ACC-002 has 5, ACC-003 has 6, ACC-004 has 6, ACC-005 has 7)
- Leaves unchanged fields exactly as they were in v1
- Produces a changelog that records what the old value was, what the new value is, and why it changed
- Produces an updated agent spec with a system prompt that reflects all the v2 changes
- Running the pipeline twice on the same onboarding file produces the same v2 output both times

---

## The Trickier Cases Worth Checking Specifically

A few scenarios were written to catch pipelines that handle the easy stuff fine but fall apart on edge cases. These are worth examining directly:

**ACC-001** — The primary on-call technician was replaced. The old phone number (Mike Salazar's) must not appear anywhere in v2. This tests whether the pipeline overwrites correctly or accidentally appends.

**ACC-002** — The gas smell emergency has a specific instruction attached to it: the caller should be told to evacuate the building and call their gas utility before a technician is dispatched. A pipeline that just routes the call without capturing this instruction has missed something important.

**ACC-003** — The word "guarantee" or "guaranteed" is a hard-banned term due to a prior legal dispute. The generated agent prompt must never contain this word in any form, even in a positive context like "we guarantee a callback." This tests how well the pipeline respects explicit constraints.

**ACC-004** — The emergency routing order was intentionally swapped. In v1, Linda is called first and Aaron second. In v2, Aaron is called first and Linda second. A pipeline that merges rather than replaces will get this backwards.

**ACC-005** — The zone dispatch structure was not just updated, it was rebuilt. Old zones were reorganised and a new technician was added. A pipeline that appends to the existing zone list instead of replacing it will produce duplicate and incorrect routing rules.

---

## A Note on the Data Itself

Everything in these transcripts is fictional. The phone numbers use the 555 prefix, which does not exist in real US phone networks. The company names, addresses, and people are invented. Nothing here is real client data, and nothing should be treated as such.

The transcripts were written to be realistic enough that an LLM will extract them correctly, but controlled enough that the expected outputs can be verified precisely.

---

*Dataset built and documented by Aryaman for the ZenTrades AI assignment.*