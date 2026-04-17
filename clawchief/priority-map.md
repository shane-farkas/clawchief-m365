# Priority Map

Purpose: define who and what should matter enough to interrupt {{OWNER_NAME}}, trigger {{ASSISTANT_NAME}} action, or be safely ignored.

This file is the canonical "people + programs" layer for OpenClaw prioritization.

## Ownership boundary

This file owns:
- people importance
- program importance
- urgency level
- routing/action mode

This file does not own:
- task storage mechanics
- reply wording
- meeting-note ingestion procedure
- cron timing or retries

## How to use this file

When a new signal arrives from Slack, email, calendar, tasks, meeting notes, docs, or notes:

1. Map it to zero or more *people*
2. Map it to zero or more *programs*
3. Assign an urgency level
4. Choose the action mode

If a signal maps to no important people and no important programs, it should usually be ignored or batched and then archived.

## Urgency levels

- **P0 — interrupt now**
  - time-sensitive, high-stakes, or blocking
  - appropriate to put in front of {{OWNER_NAME}} quickly
- **P1 — same day**
  - important enough to surface today
  - may require {{OWNER_NAME}} or {{ASSISTANT_NAME}} action today
- **P2 — digest / batched**
  - worth tracking, but not worth interrupting {{OWNER_NAME}} for immediately
- **P3 — ignore / archive**
  - low-value noise, duplicative context, or non-actionable chatter

## Action modes

- **Interrupt {{OWNER_NAME}} now** — surface directly in Slack as a short, clear alert
- **Handle and summarize** — {{ASSISTANT_NAME}} acts, then gives {{OWNER_NAME}} a concise update if helpful
- **Queue for digest** — hold for next structured summary / heartbeat if still relevant
- **Ignore** — no user-facing message needed

## Task grouping convention

clawchief/tasks.md labels and section choices should connect back to this file's program/person grouping when there is a clear match.

Preferred pattern:

- owner assignment: `ryan` or `r2`
- one program section when there is a clear match
- due date or deadline to express when the next action should happen
- labels only for secondary context when they add real value

Rules:
- Use the exact program names from this file when choosing clawchief/tasks.md sections whenever there is a clear match.
- If a task does not clearly map to a program or person here, leave it unlabeled rather than inventing a noisy label.
- Do not use workflow-state sections like `Today`, `Next`, `Waiting`, or `Scheduled`; due dates should carry timing/state instead.
- Preserve stable program grouping unless {{OWNER_NAME}} explicitly wants a regroup.

## People

People are not only important because they are operationally relevant.
They are also important because they matter relationally to the principal.
The system should treat trust, family, loyalty, and relationship depth as real prioritization signals, not just logistics.

### {{OWNER_NAME}}
- Why he matters:
  - he is the principal, the decision-maker, and the person this whole system exists to help
- Watch for:
  - direct asks
  - approvals needed
  - decisions only {{OWNER_NAME}} can make
  - anything that affects his calendar, travel, or revenue priorities
- Escalate to P0 when:
  - there is a hard deadline within 24 hours
  - there is a meeting conflict or urgent scheduling issue
  - a prospect / partner / investor / board item is blocked on him
- Default action:
  - usually interrupt {{OWNER_NAME}} now or handle and summarize

## Core personal relationships

### Spouse / partner
- Why they matter:
  - this relationship is central to {{OWNER_NAME}}'s life and scheduling reality
- Watch for:
  - shared-calendar conflicts
  - family logistics that affect {{OWNER_NAME}}'s schedule
  - messages or situations that need coordination, support, or a response
  - board-related context when the spouse / partner is directly involved
- Escalate to P0 when:
  - same-day family logistics affect {{OWNER_NAME}}'s time or commitments
  - there is a time-sensitive ask {{OWNER_NAME}} should see quickly
- Default action:
  - handle and summarize unless immediate attention is clearly warranted

### Children
- Why they matter:
  - family matters involving {{OWNER_NAME}}'s children are intrinsically important, not merely administrative
- Watch for:
  - scheduling, school, travel, family coordination, health, or other time-sensitive needs
- Escalate to P0 when:
  - there is same-day schedule impact, urgency, or something {{OWNER_NAME}} would strongly want to know quickly
- Default action:
  - handle and summarize, escalating when time-sensitive

### Parents
- Why they matter:
  - family communication with parents should be treated as emotionally meaningful, not background noise
- Watch for:
  - calls, reminders, family updates, health changes, travel, or anything that may matter to {{OWNER_NAME}} personally
- Escalate to P0 when:
  - there is important family news, health news, or time-sensitive coordination
- Default action:
  - handle and summarize unless urgent

### Siblings
- Why they matter:
  - sibling and extended family developments should count as real priority signals when relevant
- Watch for:
  - meaningful family updates, moves, coordination, birthdays, or requests that would matter to {{OWNER_NAME}}
- Escalate to P1 when:
  - there is important family news or a time-sensitive ask
- Default action:
  - queue or handle and summarize depending on urgency

## Pets

### Household pets
- Why they matter:
  - recurring pet care should be treated as a real responsibility, not generic admin
- Watch for:
  - care reminders {{OWNER_NAME}} explicitly wants tracked
  - time-sensitive pet logistics that affect {{OWNER_NAME}}'s day or travel
- Escalate to P1 when:
  - same-day care or travel coordination could be missed
- Default action:
  - queue or handle and summarize depending on urgency

## Friends

### Close friends and operator peers
- Why they matter:
  - some relationships matter personally and may also have strategic value
- Watch for:
  - relationship-building opportunities
  - insight exchange
  - direct asks or collaboration opportunities
- Escalate to P1 when:
  - there is a clear reply opportunity or useful follow-up that should not sit
- Default action:
  - handle and summarize; do not over-interrupt {{OWNER_NAME}} unless strategically relevant

## Important work / strategic relationships

### {{KEY_LEGAL_CONTACT_NAME}}
- Why they matter:
  - key trusted operator for {{BUSINESS_NAME}} — Chief Legal Officer (or equivalent)
  - legal, policy, standards, and sensitive operating judgment run through them
- Watch for:
  - legal/compliance questions
  - policy claims that should not be answered casually
  - anything involving partner standards, referral structure, disclosures, or legal risk
- Escalate to P0 when:
  - a live thread needs a legal or policy answer to keep moving
  - a risky outbound response is about to be sent without approved guidance
- Default action:
  - handle and summarize, or interrupt {{OWNER_NAME}} if their input is blocking an important thread

### {{BOARD_MEMBER_NAME}}
- Why they matter:
  - they sit on the {{BUSINESS_NAME}} board (or represent a key investor) and are a high-trust strategic relationship
  - communication involving them can affect {{OWNER_NAME}}'s strategic accountability and support structure
- Watch for:
  - board updates
  - investor communications
  - requests that affect financing, reporting, or strategic accountability
- Escalate to P0 when:
  - there is a near-term board/investor deadline or a sensitive strategic issue
- Default action:
  - handle and summarize unless {{OWNER_NAME}} decision is required soon

## Dynamic people groups

### Warm prospects / referral partners
- Listed in the live outreach sheet / CRM configured for this install.
- Why they matter:
  - direct path to early customers and channel leverage
- Watch for:
  - replies, objections, scheduling, interest signals, referrals, pilot discussions
- Escalate to P0 when:
  - a hot lead or warm partner reply is waiting and timing matters
- Default action:
  - handle and summarize

### Angel Investors
- Why they matter:
  - high-signal strategic communication
- Watch for:
  - requests for updates, metrics, materials, meetings, or decisions
- Escalate to P0 when:
  - response timing or message quality matters materially
- Default action:
  - handle and summarize, escalating when sensitive or strategic

## Programs

### First 10 paying customers
- Why it matters:
  - {{OWNER_NAME}} said this is the immediate top company priority
- Q2 target ladder:
  - 2026-04-30: 10 total customers
  - 2026-05-31: 20 total customers
  - 2026-06-30: 30 total customers
  - Current: 0
- Watch for:
  - anything that can directly create customers, learn from prospects, unblock acquisition, or improve conversion
- Examples:
  - partner introductions
  - prospect replies
  - customer objections
  - offers, pilots, pricing reactions, conversion bottlenecks
- Escalate to P0 when:
  - a near-term revenue opportunity is blocked on {{OWNER_NAME}}
  - there is a hot prospect / partner thread needing same-day action
- Default action:
  - handle and summarize or interrupt {{OWNER_NAME}} if his action is required today
- Ignore / downgrade:
  - vague marketing ideas with no clear next step

### Instagram content
- Why it matters:
  - this is a specific GTM program {{OWNER_NAME}} is actively using
  - it supports authority, reach, and customer acquisition
- Watch for:
  - recording videos with team or partners, editing, posting, scheduling, distribution, and follow-up needs
- Escalate to P1 when:
  - a recording window, publishing deadline, or blocker could stall momentum
- Default action:
  - handle and summarize unless same-day action is needed

### AEO / SEO
- Why it matters:
  - compounding acquisition channel
  - improves discoverability for people actively searching for help in {{TARGET_MARKET}}
- Watch for:
  - rankings, content opportunities, technical issues, and search-performance changes
- Escalate to P1 when:
  - there is a concrete change that can materially improve discoverability or fix a meaningful issue
- Default action:
  - queue for digest unless action is clearly time-sensitive

### Meta Ads
- Why it matters:
  - paid acquisition channel with direct customer-growth implications
- Watch for:
  - campaign launches, account issues, creative needs, performance shifts, and blocking setup problems
- Escalate to P1 when:
  - spend, delivery, or setup problems are blocking acquisition
- Default action:
  - handle and summarize, escalating when {{OWNER_NAME}} decision is needed

### Google Ads
- Why it matters:
  - paid acquisition channel tied to search intent and conversion
- Watch for:
  - campaign launches, keyword / landing-page issues, performance changes, and setup blockers
- Escalate to P1 when:
  - a change materially affects traffic, lead flow, or spend efficiency
- Default action:
  - handle and summarize, escalating when {{OWNER_NAME}} decision is needed

### {{NAMED_PARTNERSHIP}} partnership with {{PARTNER_CONTACT_NAME}}
- Why it matters:
  - named partnership program with direct distribution and credibility implications
- Watch for:
  - communication with {{PARTNER_CONTACT_NAME}}
  - partnership tasks
  - dashboard / listing work
  - magazine / expert / feature opportunities
- Escalate to P1 when:
  - the partnership is blocked, an opportunity is live, or {{OWNER_NAME}} input is needed to keep momentum
- Default action:
  - handle and summarize

### Business development partnerships
- Why it matters:
  - this is the live BD motion for building partnerships with professionals in {{TARGET_MARKET}}
- Watch for:
  - outreach replies
  - follow-up gaps
  - meetings
  - partner questions
  - objections
  - pilot opportunities
- Escalate to P0 when:
  - a strong reply or meeting opportunity is waiting and timing matters
- Default action:
  - handle and summarize

### Executive assistant: inbox, calendar, travel, schedule integrity
- Why it matters:
  - this is the operating system for {{OWNER_NAME}}'s day-to-day execution capacity
  - inbox, scheduling, travel, and coordination tasks can create immediate drag or unblock important work
- Watch for:
  - scheduling follow-ups
  - travel constraints
  - inbox cleanup or reply obligations
  - meeting prep
  - calendar integrity issues
- Escalate to P1 when:
  - a scheduling issue, travel constraint, or inbox follow-up could disrupt the next few days
- Default action:
  - handle and summarize

### Out-of-home advertising
- Why it matters:
  - active awareness channel with real spend and coordination attached
- Watch for:
  - proposals
  - creative
  - placements
  - pricing
  - deadlines
  - vendor coordination
- Escalate to P1 when:
  - a decision, asset, or approval is blocking execution
- Default action:
  - handle and summarize

### {{VENTURE_FUND_NAME}} venture fund wind-down
- Why it matters:
  - {{OWNER_NAME}} runs a small venture fund that is winding down
  - it still creates real investor, admin, and communication obligations that should not get lost
- Watch for:
  - inbox follow-up
  - investor updates
  - admin or communication tasks that still require attention
  - wind-down obligations, deadlines, or cleanup tasks
- Escalate to P1 when:
  - an investor-facing or time-sensitive item is waiting
  - a wind-down deadline or obligation needs action
- Default action:
  - handle and summarize

### Board / investor communication
- Why it matters:
  - board and investor communication affects {{OWNER_NAME}}'s strategic accountability, financing context, and external trust
- Watch for:
  - investor updates
  - board reports
  - materials due to board members or investors
  - recurring reporting obligations
- Escalate to P1 when:
  - a board or investor deliverable is due soon
  - a strategic response needs {{OWNER_NAME}} input or careful wording
- Default action:
  - handle and summarize

### Family / personal logistics
- Why it matters:
  - family and personal obligations are real priorities, not background admin
  - they affect {{OWNER_NAME}}'s life directly and often affect his capacity for work too
- Watch for:
  - birthdays
  - anniversaries
  - family check-ins
  - personal reminders
  - family coordination
- Escalate to P1 when:
  - a family obligation is time-sensitive or emotionally important enough that {{OWNER_NAME}} would want it surfaced promptly
- Default action:
  - handle and summarize

### Health / medical
- Why it matters:
  - medical follow-ups and preventive care are high-value long-horizon responsibilities that should not fall through the cracks
- Watch for:
  - annual exams
  - blood tests
  - checkups
  - specialist follow-ups
  - recurring health reminders
- Escalate to P1 when:
  - a medical task is time-sensitive, overdue, or tied to a real appointment / test deadline
- Default action:
  - handle and summarize

### Home / household
- Why it matters:
  - the house, vehicles, and recurring maintenance tasks create real operational load and can become expensive or disruptive if missed
- Watch for:
  - service scheduling
  - maintenance reminders
  - seasonal house work
  - vehicle upkeep
  - home systems that need recurring attention
- Escalate to P1 when:
  - a maintenance issue becomes urgent, seasonal, or costly if delayed
- Default action:
  - handle and summarize

### Club / community communications role
- Why it matters:
  - the principal holds a recurring communications leadership role in a community organization
  - this is a named recurring role / program, not generic admin
  - this work is a real recurring commitment and should not disappear into generic admin noise
- Watch for:
  - newsletter / publication deadlines
  - committee communications
  - scheduling or content obligations tied to {{COMMUNITY_ORG_NAME}}
- Escalate to P1 when:
  - a publication, deadline, or committee obligation is time-sensitive
- Default action:
  - handle and summarize

### clawchief improvement
- Why it matters:
  - compounds {{ASSISTANT_NAME}} effectiveness over time
- Watch for:
  - operator patterns worth copying
  - setup changes that improve autonomy, prioritization, or execution
  - concrete opportunities to improve the system {{OWNER_NAME}} uses daily
  - new OpenClaw capabilities that can be turned into better clawchief workflows
- Escalate to P1 when:
  - {{OWNER_NAME}} explicitly asks for system improvement work or a change is clearly high leverage
- Default action:
  - convert into a concrete task, proposal, or direct internal update pass instead of leaving it as a vague idea
- Ignore / downgrade:
  - theoretical AI chatter without a concrete operational implication

## Default routing rules

Use these deterministic routing defaults unless a clearer instruction overrides them:

- general inbox, scheduling, travel, calendar integrity, or operational coordination -> `executive-assistant`
- partner / referral / prospect pipeline, outreach tracker, lead verification, or prospecting batch work -> `business-development`
- direct task CRUD, reprioritization, completion, or current-task review -> `daily-task-manager`
- morning due-date retuning and daily actionability prep -> `daily-task-prep`
- meeting notes enter through `executive-assistant`, then route into `business-development` too if the note changes outreach / partner state
- cross-cutting task follow-up / blocker rules come from the task-system skill (if installed), not from this file

If a signal maps to more than one route, choose the workflow that owns the live source of truth being changed.

## Things that should usually be ignored or batched

- casual chatter with no action
- repeated notifications that add no new information
- documents or messages with no connection to a priority person or program
- speculative ideas without owner, deadline, or next step
- low-stakes activity already captured in the task list

## Review cadence

Review this file when:
- {{OWNER_NAME}} says priorities changed
- a new person becomes important
- a new recurring program appears
- summaries feel noisy or are missing important things
