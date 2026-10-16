# AI operations layer: idea

Status: draft, updated 2026-10-16. Research: [market](research/market.md), [software](research/software.md)

## Goal

A new service line for the agency. We connect a business's existing tools and add an AI assistant that answers questions, flags problems, and prepares actions people approve with one click.

**We are client #0.** Everything gets piloted on our own team first.

Why: building websites alone isn't enough anymore. This extends what we sell to existing and new clients, and it brings recurring revenue instead of one-off projects.

## Inspiration: Mission Point "One-Click AI"

https://missionpoint.bg/one-click/

What they sell:

- A thin layer on top of a company's existing systems, plus the modules that are missing.
- One assistant ("Laura AI") for questions, alerts and approvals, by text or voice.
- Agents prepare the email, invoice or task. A person clicks **Confirm**.
- Their split: **software executes, the AI agent processes, the human decides.**
- Two offers: "connect and complete" (keep your systems) or "a new integrated system" (replace them).
- Price by scope, with ongoing support. They claim 1–3 weeks per project.

What we learn from them:

- **They sell process design and integration, not AI.** They're an organizational consultancy (12 years) that turned its method into a product.
- **1–3 week delivery only works with reusable parts** and templates per industry (service company, online store, clinic).
- **They make it easy to start small.** Their demo menu offers the full system, single AI agents, automations, a custom build, or "not sure where to start?".

## Strategy: service first, product later

We build a **kit, not a platform**. We sell projects, and the parts that repeat across them become the kit.

**The kit** (built once, reused for every client):

- **Connector library:** one connector per tool (Gmail, Outlook, Redmine, FreeAgent, Xero, store platforms, couriers…).
- **Assistant** inside the chat tool the client already uses (Slack, Teams, Google Chat, maybe Viber).
- **Approval pattern:** proposal → Confirm / Edit → execute → log.
- **Entity map pattern:** one table that links a client, customer or project across all the tools.
- **Scheduled checks + alerts pattern.**
- **Discovery template:** questions, a scoring sheet, a proposal outline.

**Per client:** discovery, one or two new connectors, workflow setup.

We build a real multi-tenant SaaS only after about 3 paying clients show us what repeats.

## Offer ladder

| Offer | What | Time | Price model |
|---|---|---|---|
| 1. AI audit / workshop | Map processes, find the top 3 time sinks, recommend fixes | 1–2 days | Fixed fee |
| 2. Single agent | One job done well, e.g. email triage, invoice chasing, order status | 1–2 weeks | Setup + monthly |
| 3. Connected system | Several tools + assistant + alerts + approvals | 3–6 weeks | Setup by scope + monthly retainer |

The monthly fee covers support, fixes when a tool's API changes, LLM costs, and small changes.

## Our edge and our gaps

**Edge**
- Existing website clients are warm leads.
- The website is often a natural entry point: contact forms and store orders can feed straight into the system.
- We have developers in-house.

**Gaps**
- **Process discovery.** This is Mission Point's core skill, and we need to build it.
- **Different buyer.** We'd sell to owners and operations managers, not marketing teams.

## Client #0: us

### Stack (we keep all of it)

| Tool | What it holds | API |
|---|---|---|
| FreeAgent | Accounting: contacts, invoices, bills, bank, projects, timeslips | REST + OAuth. No webhooks, so we poll. |
| Redmine | Projects, issues, time entries, wiki | REST + per-user API key. An admin must enable the REST API. |
| Google Workspace | Gmail, Calendar, Drive and Docs | Full APIs. Internal apps in our own domain skip Google's app review. |
| Slack | Team chat | Events API, buttons and modals. The assistant lives here. |

### Known pains

| Pain | Who | Fix |
|---|---|---|
| Email threads that need a reply get lost in Gmail | Several colleagues | **Email agent** (first build, see below) |
| Credentials stored in Redmine are hard to find | Chavdar | **Not an AI job.** Move them to a password manager (Bitwarden / Vaultwarden, or 1Password). Do this *before* connecting Redmine to any AI tool, or the secrets go to the AI provider. |
| Writing and formatting Redmine how-to pages | Chavdar | Small helper: rough notes → formatted wiki page. Later: "ask the wiki". |
| Everything else | Unknown | **Run discovery** (below) |

### Discovery (doubles as our client template)

A 30-minute talk with each colleague, using the same questions every time:

1. What did you do by hand this week that felt like a waste?
2. What do you copy from one tool into another?
3. What do you search for and can't find?
4. What do you lose track of or forget?
5. If you had an assistant tomorrow, what would you give it first?

Score each pain by **how often it happens × time it costs × number of people affected**. The top one is the next build.

What we learn here becomes our audit offer (offer 1).

### First build: team follow-up (email agent)

Why first: it's a real pain here, and nearly every business has the same problem. It's the easiest piece of the kit to sell.

**Positioning:** AI that drafts replies in one inbox is already a commodity (Gmail, Outlook, Fyxer, Superhuman; see research/market.md §6). So we sell **"nothing falls through the cracks across the team"**: follow-up tracked across everyone's inbox without switching tools, with drafts that use live data from the CRM, tickets and invoices. It's workload routing, never scoring people (EU AI Act).

What it does:

- Tracks threads where someone outside is waiting on us.
- Sends each person a daily Slack digest, e.g. "5 threads waiting more than 24 h", with links.
- Drafts replies using context from past emails, Redmine and Drive.
- Labels and sorts threads in Gmail.
- **Never sends on its own.** The person reviews, edits and sends.

Tech: Gmail API (push via Pub/Sub, or polling), Claude to sort threads and write drafts, a Slack bot for the digest and buttons, Postgres for thread state.

To sell it, the kit later needs an Outlook / Microsoft 365 version too.

How we measure it: number of threads waiting more than 48 h, and time to first reply. Measure both before and after.

### Later for us

- Month-end invoicing: Redmine hours → draft FreeAgent invoice → Confirm in Slack.
- Overdue invoices → draft reminder email.
- Project budget alerts (hours spent vs estimate).
- Cross-tool questions: "Who last talked to this client?" "What's open, and what's been billed?"

## Architecture

```
People ──► Chat tool (Slack for us): questions, alerts, [Confirm] [Edit]
              │
        Agent: Claude + tools
        (reads freely; every write is a proposal a human confirms)
              │
   Workflows + scheduled checks  ───  audit log of every action
              │
   Data layer: Postgres (synced records + entity map) + docs index
              │
   Connectors: one per tool, swappable
```

## Rules

- **Numbers come from data, never from the model.** Every figure comes from a query or tool result, with the source shown.
- **Permissions are enforced in code, not in the prompt.**
- **Nothing financial or outgoing happens without a human confirming it.**
- **Log every action:** who asked, what the agent proposed, who confirmed it.
- **Secrets never go through the AI.** Keep credentials in a password manager.
- **Data:** host in the EU. Sign data processing agreements with clients and with the LLM provider.

## Rollout

Times are rough guesses.

| Phase | What | Rough time |
|---|---|---|
| 0 | Move credentials out of Redmine. Run discovery with the team. Try Claude + read-only connectors. | 1–2 weeks |
| 1 | Email agent for our team | 2–3 weeks |
| 2 | Package it as offer 2. Pilot with 1–2 friendly existing clients at a discount. | 1–2 months |
| 3 | Connected layer for us (Redmine, FreeAgent, entity map) and for the pilot clients | 1–2 months |
| 4 | After ~3 paying clients, decide what to turn into a product | later |

## Risks

- **Building a platform before anyone pays.** Sell first, then build.
- **Liability:** agents acting inside client systems. Covered by approvals, the audit log and contract terms.
- **Support load:** integrations break when tools change their APIs. The retainer has to cover this.
- **Skills gap:** discovery and process consulting are different from web development.
- **Crowded market:** many "AI agencies" exist. We stand out through an industry niche and our existing client relationships.

## Open questions

1. Which market do we sell to: Bulgaria, UK, or both? This decides which connectors come first (e.g. Microinvest, Econt, Speedy vs Xero, QuickBooks).
2. Which industries are our current clients in? The first niche should come from there.
3. Who on the team runs discovery and sales conversations?
4. Which 2–3 existing clients would try a pilot?
5. What chat tool do target clients use: Slack, Teams, Google Chat, Viber?
6. Is our Redmine self-hosted? Is the REST API enabled?
