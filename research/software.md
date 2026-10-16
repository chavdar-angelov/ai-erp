# Software research: how to build the AI operations layer

Date: 2026-10-16. Companion to [idea.md](../idea.md) and [market.md](market.md).

The scope has three parts. How competitors build their software, and the good and bad sides of each approach. What building blocks exist in late 2026. What we should build, what we should reuse, and how to keep it practical and easy to roll out.

Effort and cost figures are estimates unless a source is given. **(uncertain)** marks claims we couldn't verify.

---

## Summary

- **Approach: "assemble, thin".** We build a small core of our own and reuse everything else.
  - **Our own core:** approvals, an audit log, the entity map, a policy layer and an agent runner.
  - **Reused:** connectors (MCP), open-source workflow and job tools, the client's own chat tool (Slack or Teams), and open-source modules where a client is missing one.
  - **Not like Mission Point:** we don't build our own CRM or ticketing.
  - **Not white-label:** no platform does internal-ops agents plus white-label plus approvals plus local systems.
- **Stack:**
  - TypeScript core and Postgres, one copy per client, hosted in the EU.
  - Claude models: **Sonnet 5.5** for drafts and questions, **Haiku 5.5** for sorting emails.
  - Models run through **AWS Bedrock (EU regions) or Google Vertex (EU)**. Anthropic's own API can't keep processing in the EU.
  - The whole stack is MIT or Apache licensed, so it's safe to white-label through UK partners.
- **The core design rule:**
  - The AI only *proposes*.
  - A person approves the **exact** action (recipients, text, amounts).
  - **Plain code** carries it out.
  - Every incoming email counts as possibly hostile.
- **Cost per client:**
  - Hosting: about €20–35 a month.
  - AI usage for a 10-person email agent: about $60–120 a month.
- **The big warning:** Anthropic launched **Claude for Small Business** in May 2026. It's an assistant over QuickBooks, HubSpot, Google and Microsoft 365, with owner approval, at no cost beyond Claude seats.
  - So "an AI assistant over your tools" is becoming free.
  - We have to sell **local connectors, packaged results, and running it for the client.**

---

## 1. How competitors build, and what's good and bad about each

| Who | Approach | Good | Bad |
|---|---|---|---|
| **Mission Point** | **Full custom app.** Next.js on Claude, self-hosted with Dokploy, their own CRM, projects, quotes and tickets modules, and a Meet transcription bot. | A polished single UI and full control. | Heavy to build. They own every bug. They rebuild modules that open source already covers. Each client deployment is a one-off. |
| **Digital Agent, FlowexAI** | **No-code first:** n8n or Make plus an LLM | Fast first demo (2–4 weeks) | Silent failures (expired logins, changed APIs). The n8n licence blocks white-label and shared multi-client hosting without a paid deal. Clients start asking "why pay to maintain this?" |
| **Picase** | n8n + EspoCRM + local open-weight models (Ollama, Qwen, Llama) | A strong privacy and on-premises story | Local models are weaker at using tools. Same n8n limits. |
| **INDUSTRIA** | Odoo + Claude via MCP | Clean for companies already on Odoo | Tied to Odoo |
| **ERP.net Operator** | Agents built into its own ERP | Deep integration | Only for its own ERP customers |
| **Claude for Small Business, Copilot, Gemini, Viktor ($50/workspace)** | Off-the-shelf assistant with connectors and approvals | Cheap or free, and good enough for simple cases | No local systems, no process work, nobody runs it for the client |

**What we take from this**
- The no-code builds are fast but fragile and hard to white-label.
- Mission Point's full custom build is expensive.
- The off-the-shelf assistants are commodity.
- **The gap is a thin, solid core with local connectors, run as a managed service.**

---

## 2. Approaches compared

These are estimates.

| | A: no-code (n8n / Make) | B: white-label platform | **C: assemble (recommended)** | D: full custom (Mission Point) |
|---|---|---|---|---|
| Time to first client | 2–4 wks | 1–3 wks | 6–10 wks for the core, then 2–4 wks per client | 4–9 months |
| Monthly cost per client | €30–150 + AI, plus 3–8 h of fixes | $50–500+ platform fee | €40–150, plus 1–3 h | €50–200, plus 2–5 h |
| Margin | Good at first, then shrinks | Capped by vendor fees; vendor can reprice | **Best from about 8 clients** | Only pays off at scale |
| Differentiation | Low | Lowest | Medium–high | High but costly |
| Lock-in | Medium (n8n licence) | High | **Low** (MCP is an open standard) | None |
| Main risk | Every client is a one-off; silent failures | Vendor reprices or gets acquired | Open-source dependencies die | Building commodity modules |

**White-label platforms checked:** Stack AI, Relevance, Botpress, Voiceflow, Lindy, Dust, Langdock, Cassidy, Gumloop, n8n Embed, Make, Activepieces Embed, Stammer and GoHighLevel. None fits.
- **The ones built for internal operations don't allow white-label.**
- **The ones that allow white-label are customer-facing chatbots.**
- Several repriced sharply in 2025–26:
  - Synthflow's agency tier reportedly went from $600 to $1,250 a month.
  - Stack AI dropped its self-serve plans around the time Asana bought it.
  - Relevance and Make moved to credit-based pricing.

---

## 3. Architecture

```
                 Slack / Teams / web dashboard / (Viber add-on)
                        │  questions, alerts, [Confirm] [Edit] [Reject]
                        ▼
┌──────────────────────────── per-client instance (EU VM) ────────────────────────────┐
│                                                                                     │
│  Incoming email ─► 1. Quarantine step (Haiku): raw email → strict JSON              │
│                       (intent, entities, dates, questions). No tools.               │
│                                │                                                    │
│                                ▼                                                    │
│                    2. Planner agent (Sonnet) with READ tools only                   │
│                       + propose_* tools that only write a proposal row              │
│                                │                                                    │
│                                ▼                                                    │
│                    3. proposals table (Postgres): exact payload + hash, status      │
│                                │  human clicks Confirm on the exact payload         │
│                                ▼                                                    │
│                    4. Executor (plain code, no LLM) → connector write → audit log   │
│                                                                                     │
│  Scheduled checks (pg-boss cron) ─► alerts ─► proposals                             │
│  Entity map: client ↔ FreeAgent contact ↔ Redmine project ↔ email domain ↔ …        │
│  Data: Postgres + pgvector (permission-aware), synced records                       │
│                                                                                     │
│  Connectors (TS modules, also exposed as MCP servers):                              │
│   Gmail · M365 · IMAP · Slack · Teams · HubSpot · Redmine · FreeAgent · Xero ·      │
│   WooCommerce · Econt · Speedy · inv.bg · Microinvest (via bridge)                  │
└─────────────────────────────────────────────────────────────────────────────────────┘
        │ AI model calls: Bedrock EU (Frankfurt/Ireland) or Vertex eu
        │ Traces: Langfuse (EU)
```

**Why approvals live in our own database, not in a framework or workflow engine**
- A one-click approval is just a row with a status. The click flips the status and queues the job that runs it.
- It survives restarts, works from Slack, Teams or the web, and gives us the audit trail for free.
- **Several small "agent inbox" and approval projects died in 2026:** LangChain Agent Inbox, HumanLayer's open-source SDK, Airweave. **We own this part.**

**The proposals table (sketch)**
`id, client, agent, action_type, payload (JSON), payload_hash, status (pending/approved/rejected/executed/failed), agent_run_id, approved_by, approved_at, executed_at, result, error`

---

## 4. Recommended stack

### Core and AI

| Layer | Choice | Why | Alternatives |
|---|---|---|---|
| Language | **TypeScript (Node 22)** for the core; **PHP** for WordPress and WooCommerce plugins | Best SDKs for everything we need: MCP (top-tier support), Slack Bolt, Teams SDK, Microsoft Graph, AI SDK. The official PHP MCP SDK is still experimental. | Laravel for the approvals, audit and UI side, with a Node service for agents (only if the team is much stronger in PHP) |
| Agent framework | **Vercel AI SDK 7** (released Jun 2026) | Thin, works with many model providers, built-in MCP client, per-tool approval policies | Mastra (more built in, but 144 of its npm packages were backdoored in Jun 2026; pin versions). Claude Agent SDK (better for coding-style agents; approvals must happen inside the running process). Anthropic's hosted agents product (**no EU option**). |
| Models | **Sonnet 5.5** ($2 / $10 per 1M tokens) for planning and drafts. **Haiku 5.5** ($0.10 / $0.50) for sorting and extraction. | Strong at tool use, and the small model is very cheap | Opus 5.5 ($4 / $20) for hard cases. Mistral or OpenAI EU as a fallback from a different vendor, used only during outages. |
| EU processing | **AWS Bedrock EU inference profile** or **Vertex `eu`** (about +10% on price) | Anthropic's own API only offers US or global processing | Microsoft Foundry EU ("coming soon") |
| Monitoring | **Langfuse** (MIT; EU cloud, or one self-hosted EU instance with a project per client) + OpenTelemetry | Open source, EU region, evals built in | LangSmith (EU, but self-hosting is Enterprise only). Helicone is in maintenance mode; avoid. |

**What we lose on Bedrock or Vertex** (per Anthropic's platform docs):
- batch processing;
- the Files API;
- the server-side MCP connector;
- hosted agents;
- automatic server-side model fallback.

**What still works:** structured outputs, prompt caching and tool use. We run the MCP client and the fallbacks in our own code anyway, so nothing important is lost.

### Integration, workflow and interface

| Layer | Choice | Why / notes |
|---|---|---|
| Connectors | **Our own TypeScript modules** behind one shared interface (sync, read tools, propose tools, execute), **also exposed as MCP servers** | No integration platform covers Econt, Speedy, inv.bg, Viber or Microinvest. In a per-client setup only about 6 sign-in (OAuth) providers matter, and token refresh is simple. Reuse the official HubSpot MCP server (behind our gateway). |
| Integration platforms | **Skip for now** | Nango's licence (Elastic) limits what free self-hosting includes. Composio's prices rose in Aug 2026. Pipedream is being bought by Workday. Unified APIs (Merge, Apideck) are priced for SaaS companies. |
| Jobs and schedules | **pg-boss** (MIT, Postgres only) | Cron, retries and queues with no Redis |
| Multi-step flows | **DBOS Transact** (MIT, a library, Postgres only) or **Hatchet** (MIT) | For things like "follow up in 3 days if no reply". Temporal is MIT but heavy to run. |
| Visual builder (optional, staff only) | **Activepieces CE** (MIT core) | **Not n8n** for anything we white-label. n8n's free licence only allows client automations the client can't edit. Reselling through partners needs a paid Embed deal (reportedly about $50k a year; uncertain). Windmill's free image and Dify's licence also restrict reselling and multi-client use. |
| Chat surface | **Slack (Bolt + Block Kit buttons)**, **Teams (Microsoft 365 Agents SDK)**, Google Chat | Teams: Microsoft's old Bot Framework is archived. Since Jul 2025 you can't create new multi-tenant bots, so we use one single-tenant bot per client. |
| Web dashboard | **Next.js + AI SDK + assistant-ui** (MIT) | For the approvals inbox, entity-map review and web chat. Not Open WebUI (branding must stay above 50 users). LibreChat is fine for plain staff chat. |
| Viber / WhatsApp | **Paid add-ons** | Viber costs about €100/month per bot plus per-message fees. WhatsApp charges per message. |

### Email: the cheapest compliant setup

- **Google Workspace clients:** each client gets a Google Cloud project **inside their own organisation**, with the app marked "Internal". Its sign-in screen is Internal.
  - **No Google verification and no paid security assessment** (CASA costs about $540–720 a year per app, for shared apps that read mail).
  - Prefer per-mailbox sign-in over domain-wide access.
  - Get push notifications through Gmail `watch` → Pub/Sub, renewed daily.
- **Microsoft 365 clients:** a **single-tenant** app in each client's own Microsoft tenant, with admin consent, limited to named mailboxes using Exchange RBAC for Applications.
  - Change notifications expire within 7 days, so renew them.
  - Note: Microsoft turns off basic-auth SMTP by default at the end of 2026.
- **Personal @gmail.com and Bulgarian hosting mailboxes:** IMAP with an app password, using IMAP IDLE re-issued every 29 minutes or less.
- **Drafts only.** The agent creates drafts in the user's mailbox. **It never gets permission to send.**

### Slack terms to respect

- Slack's API terms (2025) **ban bulk export, persistent indexes, and using Slack data in LLMs.** So **Slack is the interface, not a knowledge source.** We don't index Slack.
- Since May 2025, commercially distributed apps that aren't in the Slack Marketplace get very tight read limits on channel history. Treat our per-client apps as covered (uncertain; confirm with Slack).

### Infrastructure

| Item | Choice | Cost (est.) |
|---|---|---|
| Hosting | One small VM per client (app + worker + Postgres with pgvector): Hetzner, OVH or Scaleway | €10–20/mo |
| Deploys | **Kamal** (MIT) or **Dokploy / Coolify** for fleets. One Docker image, automatic database migrations, per-client config only. | — |
| Secrets | **Infisical** (EU cloud, or self-hosted with its MIT core) | share of €5–10/mo |
| Backups | Encrypted, off-site | €3–5/mo |
| **Total infra** | | **≈ €20–35 per client per month** |
| AI usage | 10-person team, ~2,000 emails/month (Haiku sorting, Sonnet drafts, daily digests) | **≈ $60–120/mo**; +$50–80 with heavy chat use |

**Microinvest bridge**
- Microinvest runs on MS SQL Server. **Never expose its database port to the internet.**
- Instead, run a small Windows service on the client's machine. It uses a **read-only** database login and **pushes** changes out over HTTPS to the client's instance, with no inbound ports. Tailscale with strict access rules is the alternative.
- Avoid writing back into Microinvest's database directly.

### Data layer

- **Postgres + pgvector** is enough for SMB volumes. Use filters to make search permission-aware: each chunk stores who is allowed to see it.
- **Entity map:**
  1. Match on fixed keys first: company ID (EIK) or VAT number, normalised phone number, email domain (excluding free-mail domains like gmail.com and abv.bg), and company name with "ЕООД / ООД / АД / Ltd" stripped.
  2. Then find fuzzy candidates with `pg_trgm`.
  3. **A person confirms them** in the dashboard.

---

## 5. Security (the part clients will ask about)

An email agent combines **private data**, **untrusted content** and **a way to send things out.** Simon Willison calls that the "lethal trifecta". Real attacks have already used it:
- **EchoLeak** (Microsoft 365 Copilot, 2025): a single crafted email.
- **ShadowLeak** (ChatGPT with Gmail): hidden text in an email.
- **GeminiJack** (Gemini Enterprise): a poisoned shared doc, invite or email.

**Our specific risk:** in a reply-drafting agent, **the attacker is often the person who gets the reply.**

Rules we build in:
1. **Quarantine step:** a Haiku call turns each email into strict JSON. The planner never sees the raw HTML.
2. **Limits enforced in code:** when drafting a reply to sender X, the read tools only return records linked to X's company.
3. **Least privilege:** permission to create drafts, but **no send permission**. Read-only tools are separate from `propose_*` tools.
4. **The approval screen shows the exact payload** (recipients, body, amounts), not an AI summary. The approval is tied to a hash of that payload.
5. **Plain code executes approved actions.** The LLM never calls a write API directly.
6. **Block ways data can leave:** strip remote images and HTML from drafts, never fetch URLs automatically, and allow-list outbound hosts.
7. **Rate limits and a kill switch** per client and per tool. **No long-term memory written from email content.**
8. **Append-only audit log:** model and version, prompt, tool calls, approver, payload hash.
9. **A prompt-injection test set** (Bulgarian and English) runs on every release: hidden text, "forward all invoices to…", calendar invites, PDF attachments.
10. **MCP hygiene:**
    - Review any community server before use, and pin its version.
    - Pin tool descriptions.
    - Run connectors behind our own gateway that only allows listed tools, each in its own container with outbound access only to its target API.
11. **Compliance:**
    - GDPR data processing agreements with clients and with AWS or Google.
    - A data protection impact assessment for email processing.
    - EU AI Act Art. 50 disclosure (tell people they're dealing with AI).
    - **No per-employee performance scoring** (high-risk under the AI Act).

Guidance: OWASP Top 10 for Agentic Applications 2026 (goal hijack, tool misuse, privilege abuse, memory poisoning, human trust exploitation), and the OWASP LLM Top 10.

---

## 6. "Missing modules": deploy open source, don't build

| Need | Use | Licence | Notes |
|---|---|---|---|
| CRM | **Twenty** (modern; GraphQL/REST, MCP on cloud) or **EspoCRM** (PHP/MySQL, matches our skills) | AGPLv3 | AGPL is fine for an agency: we share changes with the client who uses them. |
| Time tracking | **Kimai** (PHP) | AGPLv3 | Feeds the "unbilled time → draft invoices" agent |
| Shared inbox / helpdesk | **Chatwoot** (MIT core) or **Zammad** | MIT / AGPL | |
| Tasks / projects | **Plane** | AGPLv3 | |
| Stock / accounting for shops | **Odoo Community** | LGPLv3 | Build on the JSON-2 API; the old RPC API is removed by Odoo 22 |
| Document Q&A ("company brain") | **Onyx** (MIT community edition) | MIT | Only if a client needs heavy document search |
| **Avoid** | Flowise (shut down Aug 2026), Huly (frozen), self-hosted Cal.com (went closed source), Invoice Ninja hosted for clients (its Elastic licence bans that) | | |

**Rule:** build our own module only when **3 or more clients** hit the same gap and open source doesn't cover it.

---

## 7. Learn from these (don't fork blindly)

- **OpenExecutive** (Apache-2.0, ~5.6k stars): FastAPI + Next.js, Claude by default. Has an "act as me" mode that drafts email for review, an MCP gateway, and Slack, Google and Microsoft 365 connectors. **The closest reference design.** It's young, and its scheduler can only run as a single instance.
- **Inbox Zero** (~12.6k stars, licence not checked): drafts replies and tracks emails waiting for a response. **A direct reference for the team follow-up agent.**
- **Vercel AI SDK approval examples:** the per-tool `needsApproval` flag, for the web chat.

---

## 8. Repo layout (proposal)

```
core/          approvals, audit log, policy (limits, who approves what), entity map, agent runner
connectors/    one folder each: gmail, m365, imap, slack, teams, hubspot, redmine, freeagent,
               xero, woocommerce, econt, speedy, invbg, microinvest-bridge
               └ each = sync + read tools + propose tools + executor + MCP adapter + tests
agents/        team-followup, unbilled-time, order-exceptions, overdue-chasing
               └ each = prompts + tool list + policy + eval set + injection tests
surfaces/      slack-app, teams-bot, web (Next.js dashboard + chat)
deploy/        per-client config (which connectors and agents, secrets refs), fleet deploy scripts
evals/         Bulgarian/English email sets, injection set, graders
```

Every client runs the same code. **The only thing that differs between clients is config.**

---

## 9. Build plan

Estimates assume 2 developers.

| Phase | What | Rough time |
|---|---|---|
| 0 | Move credentials out of Redmine. Run a read-only trial with Claude + existing MCP servers on our own tools. Set up Bedrock EU and Langfuse. | 1 wk |
| 1 | **Core v0 + team follow-up for Sofyma.** Gmail (internal app), Slack bot, proposals and audit, quarantine step, drafts only. Measure before and after. | 3–4 wks |
| 2 | Redmine + FreeAgent connectors. **Unbilled time → draft invoice.** Entity map v1. Per-client deploy tooling, Infisical, backups. Injection test set. | 3–4 wks |
| 3 | **First 2–3 design-partner clients** (UK agencies). Microsoft 365 + IMAP, Teams bot, web approvals inbox. | 4–6 wks |
| 4 | Bulgarian pack: WooCommerce, Econt, Speedy, inv.bg; **order exceptions**; Viber add-on; Microinvest bridge (read-only). | 4–6 wks |
| 12 mo | With 8+ clients on one core: consider a multi-tenant version (our own code, so no licence problem) and a client-facing approvals app. | — |

---

## 10. What makes our software better, more practical and easier to roll out

| | Typical competitor | Us |
|---|---|---|
| Trust | "The AI does it" | **Approve the exact action; plain code runs it; full audit log** |
| Data location | Often US or global AI endpoints | **EU-only processing** (Bedrock or Vertex EU), one isolated instance per client |
| Lock-in | Closed app or vendor platform | **Connectors are MCP servers.** They also work with the client's own Claude, Copilot or ChatGPT. |
| Local systems | "Confirmed on the first call" | **Econt, Speedy, inv.bg, WooCommerce, Microinvest bridge**, published |
| Rollout | Custom build per client | **Same code, config only.** New client = pick connectors and agents, deploy. |
| Client effort | New tool to learn | **Works inside Slack, Teams or email**, which they already use |
| Security | Rarely discussed | **Written prompt-injection defences + test set**, data processing agreement and impact assessment templates |
| Running cost | Platform fees and credits | **≈ €20–35 hosting + AI usage**, all open licences |

---

## 11. Risks and open items

**Risks**
1. **Commoditization.** Claude for Small Business, Copilot and Gemini keep improving. Stay on local systems, cross-system work and the managed service. Sell outcomes (cash recovered, hours billed).
2. **Maintenance drag.** API changes (e.g. Odoo's RPC removal, MCP's 2026-07-28 revision), per-client hosting and dying dependencies. Keep one codebase, pin versions, monitor every run, and keep the menu to 4 agents.
3. **Security and liability.** Approval for every external write or send, limits on where data can go out, data processing agreements, and the injection test set from day one.
4. **New models reach EU regions with a delay.** Check Sonnet 5.5 and Haiku 5.5 access on Bedrock EU before promising them.
5. **Bulgarian quality is unproven.** There's no public benchmark comparing the models in Bulgarian. **Build our own eval from 50–100 real emails before promising quality.**

**Confirm before relying on these**
- How strong the team is in TypeScript vs PHP. This decides whether Laravel owns part of the core.
- Slack: do per-client apps count as "commercial distribution"?
- Bedrock EU access for Sonnet 5.5 and Haiku 5.5. Vertex `eu` as the backup.
- inv.bg API (docs exist at api.inv.bg/v3; check which plans include it). Is there an official Microinvest integration route?
- n8n Embed price and terms, only if we ever want a visual builder for clients.
- Licence for Inbox Zero, and whether self-hosted Twenty includes MCP.
