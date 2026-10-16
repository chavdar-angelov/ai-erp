# Market research: AI operations layer

Date: 2026-10-16. Companion to [idea.md](../idea.md).

The scope has three parts. What tools agencies like us and Bulgarian SMBs use. Who competes with Mission Point's One-Click, and how. What we can offer that's better, more practical and easier to roll out.

Vendor numbers are self-reported unless noted. **(uncertain)** marks claims we couldn't verify.

---

## Summary

- **Mission Point only started selling software it builds itself in 2026.** It has a working app built on Claude, but **no named clients, no prices and no named connectors**, and every scenario on its site is marked "illustrative".
- **The Bulgarian market is small and thin.** About 10 AI agencies, mostly selling chatbots or n8n automations. **Only one (Digital Agent) shows named clients with numbers.** Nobody lists local connectors (Microinvest, Econt, Speedy, Plus Minus). Nobody publishes prices for anything bigger than a chatbot.
- **AI that drafts replies in one person's inbox is now a commodity.** Gmail, Outlook, Fyxer and Superhuman all do it, at $0–64 per user per month. **Don't sell that.**
- **The value is in what isn't off the shelf:** linking systems together, local systems with no connector, team-wide follow-up, approval rules, and keeping it all running.
- **Bulgaria is behind on AI.** 8.5% of firms use it, against 20% across the EU. **The #1 blocker is lack of skills (73%)**, not cost. That's the case for a done-for-you service.
- **Our edge:** real developers who have already built custom ERPs and CRMs, a HubSpot and WordPress/WooCommerce base, UK agency partners, and 17 years of delivery.
- **How we win: proof, published prices, and a published connector list.** No local competitor has all three.

---

## 1. Us (Sofyma) at a glance

From [sofyma.com](https://www.sofyma.com/):

- Web development agency in Sofia, 17+ years, hundreds of projects. Values: family, trust, transparency.
- **Services:** web design, WordPress, custom software ("customer-specific ERPs, CRMs, marketplaces"), HubSpot development, hosting and support, HTML email.
- **A lot of white-label work** for UK agencies (Founded, Imagination, Agile Digital, Redbox Digital, Onstate), delivering for brands like Ford, SAP, Financial Times, ING and Canon. Other clients include Benefit Systems Poland and two Spanish projects.
- **Internal stack:** FreeAgent, Redmine, Google Workspace, Slack.

**What this means**
- We already have the building skills: integrations, custom modules, HubSpot.
- We're missing two things: **process discovery**, and **selling to owners and operations managers** rather than to marketing teams or agencies.

---

## 2. Tools: small agencies (5–25 people, EU/UK)

| Category | Common tools | Integration (Oct 2026) |
|---|---|---|
| Projects / issues | Jira, Asana, ClickUp, monday, Teamwork, Productive, Redmine | Most have **official MCP servers** (Atlassian, Asana, ClickUp, monday, Teamwork, Linear). Redmine has a REST API; native webhooks only from v7.0 (uncertain), so poll on older versions. |
| Time tracking | Harvest, Toggl, Clockify, or built into the PM tool | Good REST APIs. No official MCP. |
| Accounting | Xero, QuickBooks, FreeAgent, Sage | Xero and QuickBooks have official MCPs. FreeAgent has REST but **no webhooks**, and only community MCPs. |
| CRM | HubSpot, Pipedrive, **often none** | HubSpot's MCP has been generally available since Apr 2026, with write access. Pipedrive's since Jun 2026. |
| Chat | Slack, Google Chat, Teams | Slack and Google have official MCPs. Microsoft's are in preview and need a Copilot licence. |
| Docs | Google Docs, Notion, Confluence, wikis | Official MCPs. |
| Helpdesk | Often a shared Gmail inbox; Freshdesk, Zendesk, Help Scout | Mixed. |
| Passwords | 1Password, Bitwarden, **or wiki pages and spreadsheets** | Bitwarden has an official MCP, but **an agent should never see the secrets themselves.** |

**Where agencies lose time and money**
1. **Logged time never gets invoiced.** In Ignition's 2025 survey (quoted second-hand via [breeze.pm](https://www.breeze.pm/articles/agency-project-management-statistics)), **57% of agencies lose $1–5k a month to unbilled work, and 30% lose more than $5k.**
2. **Client emails don't become tickets.** Requests land in Gmail and someone copies them into the PM tool by hand.
3. **Passwords are scattered** across wiki pages, Slack and spreadsheets.
4. **No CRM.** The sales pipeline lives in the inbox and a spreadsheet.
5. **Budget overruns are spotted late.** Estimates sit in spreadsheets, actual hours in the PM tool.
6. **Renewals.** Domains, SSL and hosting are tracked and re-billed by hand.

**Takeaway:** an agency stack is the **easiest place to start.** Every tool has a good API, and most have official MCPs. We'd be building on ready parts.

---

## 3. Tools: Bulgarian SMBs (our likely clients)

Context:
- **Euro:** Bulgaria switched to the euro on 1 Jan 2026. Connectors must handle BGN history alongside EUR.
- **E-invoicing (draft law):** on 23 Sep 2026 the government proposed **mandatory B2B e-invoicing through the NRA from 1 Jan 2028.** Third-party software would send invoices in real time, and an invoice would only count once the NRA confirms it. If it passes, every invoicing tool will need an NRA connector. ([Fiscal Requirements](https://www.fiscal-requirements.com/news/6071-bulgaria-proposes-vat-and-e-invoicing-changes-from-2027-and-2028), [Sovos](https://sovos.com/regulatory-updates/global-vat/bulgaria-proposes-mandatory-e-invoicing-for-domestic-supplies/))
- **SAF-T reporting:** largest companies only for now. Most SMBs aren't in scope until about 2029–30.

| System | What it is | How hard to connect |
|---|---|---|
| **WooCommerce** | ~46% of Bulgarian online stores | **Easy.** REST API + webhooks. We already know WordPress. |
| Shopify | ~15% of stores | Easy. GraphQL API + webhooks. |
| CloudCart / OpenCart | Local SaaS / old open-source | Medium. CloudCart has a REST API; OpenCart usually means DB access. |
| **Econt, Speedy, BOX NOW** | Couriers | **Easy–medium.** All have APIs. No webhooks found, so poll. |
| **inv.bg** | Invoicing | **Easy.** REST API v3. Exports to Microinvest and Ajur. |
| **Microinvest** (Warehouse Pro, Delta) | Very common for warehouse and POS | **Hard.** On-premises Windows software. A local REST API or direct SQL access means installing a **small program on the client's machine.** |
| Ajur L, Plus Minus | Common accounting | **Hard.** No public API found (uncertain). Realistically file exports. |
| ERP.net, Odoo, SAP B1, Business Central | Modern ERPs | **Easy–medium.** Good APIs. Odoo's old API is being removed, so build on JSON-2. |
| **Viber** | ~90%+ of Bulgarian smartphone users (vendor claim) | **Medium, and it costs money.** A bot is about €100/month plus fees for messages the bot starts. You go through a partner, not self-serve sign-up. |
| Email | Gmail, Microsoft 365, **lots of hosting-provider IMAP mailboxes** (uncertain) | Need Gmail, Microsoft Graph **and** IMAP connectors. |
| Clinics (НЗИС) | National health system | Hard. No public API. **A poor first target.** |

**Takeaway:** online stores (WooCommerce plus couriers plus inv.bg) are the easiest Bulgarian vertical, and it's one we already know. **A Microinvest connector would set us apart:** it's hard, so nobody advertises one.

---

## 4. Mission Point: a close look

**What they are:** an organizational consultancy, founded and run by Verginia Nakova. It says it has 12 years in consulting and good media coverage ([Forbes BG](https://forbesbulgaria.com/2026/04/24/verginiya-nakova-ai-nyama-da-reshi-haosa-v-kompaniite-sthe-go-uskori/), [Bloomberg TV](https://www.bloombergtv.bg/a/16-biznes-start/155547-ai-v-biznesa-podredete-haosa-v-organizatsiyata-i-posle-vnedryavayte)).

**The AI offer is new.** Page creation dates from their site's public data:

| Page | Created |
|---|---|
| AI assistants | Jun 2026 |
| One-Click | **9 Oct 2026** |
| Technology Transformation Lead | 15 Oct 2026 |

**What's under the hood** (from public pages and login screens):
- Their own app (Next.js), titled "1-Click Business System".
- **Built on Claude (Anthropic)**, self-hosted.
- Modules: CRM, opportunities, projects, quotes, tickets, documents.
- A Google Meet bot that transcribes Bulgarian and writes meeting summaries.
- **No sign of n8n, Make or Odoo.**

**What they sell (no prices anywhere)**
- Free lead magnets: an "efficiency index" and an AI readiness quiz.
- AI assistants (5 templates), One-Click, CAiO as a service, and "Ongoing AI Ops".
- A new subscription with development capacity included. That suggests they're moving into delivery work.

**Good**
- A clear story ("one-click organization") and strong founder PR.
- They sell process work first, and the free index brings in leads.
- The software is real.
- A clear control story: permissions, approvals, action logs.
- Bulgarian-specific demos (Econt waybills, prices in leva).

**Bad**
- **Zero proof.** No client names, logos or case studies, and the stats contradict each other ("30+ clients" vs "300+ clients" on the same home page).
- **No prices. No named connectors** ("we confirm systems on the first call").
- **"1–3 weeks" is aggressive.** Their closest competitor's own guide says 8–14 weeks for a first agent.
- A custom deployment per client is hard to scale and maintain.
- **Mixed message:** their AI Ops page says "we don't sell technology, IT firms are just programmers", while One-Click sells technology they build.
- A sloppy site: many nav links return 404, and the default "Hello world" post is still live.

---

## 5. Bulgarian competitors

| Company | What they sell | Proof | Prices | Claimed time |
|---|---|---|---|---|
| [Mission Point](https://missionpoint.bg/one-click/) | Consulting + own Claude-based layer + CAiO | None | None | 1–3 wks |
| [Digital Agent](https://digitalagent.bg/en/) (Plovdiv) | Chat and voice agents, n8n, CRM/PMS integrations, Viber/WhatsApp | **Named clients + numbers** (mostly hotels) | Benchmark: setup €9–40k (typical **€19k**), monthly €420–5.1k (typical **€1,430**) | 1–3 wks on the site, **8–14 wks** in their own guide |
| [Encorp.ai](https://encorp.ai/bg) (Sofia) | Agents + fractional AI director + training + AI Act governance | Logos, but not tied to AI work | **Public:** €3,500/mo Growth, €50k/yr Enterprise, training €1k per group | 2–4 wks |
| [AI-Masters](https://ai-masters.bg/ceni/) | Chatbots, back-office assistant (Odoo/HubSpot), voice agents | None | **Full list:** bots €29–299/mo, diagnostics ~€2.6k, implementation ~€9k + ~€255/mo | 4–6 wks for an MVP |
| [FlowexAI](https://flowexai.bg/ai-chatbot-obsluzhvane-tsena/) | Chatbots + n8n, open-source LLMs | Anonymous | Setup €2.5k / €6k / €12k + €400–1,500/mo | n/a |
| [Picase](https://picase.bg/) | Agents, RAG, on-premises / EU hosting | Anonymized cases with numbers | Budget brackets only | 5–10 wks |
| [TalkX.ai](https://talkx.ai/) | Custom chat, voice and email agents | First-name testimonials | None | ~4 wks |
| [INDUSTRIA](https://www.industria.bg/izkustven-intelekt) | Odoo partner, Claude via MCP to Odoo; edge AI for factories | None | None | Proof of concept 1–2 wks, ERP 2–3 months |
| [ERP.net Operator](https://operator.net/) | AI agents built into its own ERP | Demos only | None | n/a |
| [AI4BG](https://ai4bg.com/) | Cheap "AI employees" + own CRM, for micro-businesses | Small named projects | From €49–99/mo | n/a |
| [MA3X](https://ma3x.bg/) | **Web agency like us.** Shops with Econt, Speedy, Microinvest; AI add-ons | Client sites | Fixed "from" prices (chatbot €1,380) | n/a |

Nothing found from the telcos (A1, Vivacom, Yettel), from Microinvest itself, or from most Odoo partners.

---

## 6. What clients could buy instead (international)

### Platforms

| Platform | Price (late 2026) | Good enough for | Weak at |
|---|---|---|---|
| Microsoft 365 Copilot + Copilot Studio | $21/user/mo + agent credits | Companies fully on Microsoft 365 | Non-Microsoft systems; confusing licensing |
| Gemini Enterprise / Workspace | Included in Workspace; Enterprise $21/seat | Google shops: docs, email, simple agents | Real actions in other systems |
| ChatGPT Business / Claude Team | ~$20–25/user/mo | Q&A over connected apps; human approval built in | Systems with no connector; nobody sets up approval rules for you |
| Zapier / Make | From ~$10–70/mo; usage-priced | 2–3 step automations between popular SaaS tools | Gets fragile and expensive at volume |
| n8n | Free self-hosted; cloud from €20/mo | **The best fit as our own toolkit** (self-hosted, EU, approval via Slack or email) | Too hard for an SMB to run alone |
| Lindy | $30/user/mo (credits) | Closest off-the-shelf email and agent assistant | Credit usage is unpredictable |
| HubSpot Breeze | Assistant free; agents on credits | Companies where HubSpot is the core system | Blind to accounting and ERP |
| Glean | Large contracts only | Not for SMBs | — |
| Odoo AI, Notion AI | Bundled or credits | Only for companies already on them | — |

What changed in 2025–26:
- **Almost everything moved to credit or usage pricing.**
- **"Ask before acting" is now a standard switch.**
- Big suites cut prices (ChatGPT Business to $20, Copilot Business at $21).

### Email tools

| Tool | Price per user/mo | Does | Doesn't do |
|---|---|---|---|
| Gmail + Gemini, Outlook + Copilot | Included / $21 | Drafts, summaries, priorities | Team-wide follow-up; business data |
| Fyxer | $37–99 | Labels, drafts in your tone, follow-ups; Team plan shares follow-ups | Live data from CRM or ERP |
| Superhuman | $33–40 | Auto drafts, reminders; CRM view is read-only | Acting in other systems |
| Shortwave | $30–120 | AI assistant, to-dos (Gmail only) | Outlook; other systems |
| Front, Hiver, Missive | $14–105 | Shared inbox, assignment, SLAs | Everyone has to move to a new inbox |

**Gap:** no tool tracks "who owes a reply" across the team **without changing inboxes**, and none uses live business data (unpaid invoice, order status, open ticket).

### Service providers abroad (price anchors)

| Offer | Typical price |
|---|---|
| Audit | $250–1,000 (small business), $1.5–5k (mid-market) |
| Simple workflow | $500–2k |
| Multi-tool, 3–5 integrations | $2–5k |
| Custom agent with human approval | $5–15k, 4–8 weeks |
| Monitoring retainer | $300–1,500/mo |
| Improvement retainer | $1.5–5k/mo |
| UK first automation | £1.5–5k |
| UK CRM-connected agent | £8–18k |
| Fractional CAIO | $3–10k/mo |

Sources: [Vantaige](https://vantaige.io/blog/2026-ai-automation-rate-card-operators-charge), [Softomate](https://www.softomatesolutions.com/blog/ai-automation-pricing-uk-businesses-2026/), [Groovy Web](https://groovyweb.co/blog/ai-consulting-rates-2026).

### Why AI projects fail

- **Gartner (2025):** more than 40% of agentic AI projects will be cancelled by 2027, over cost, unclear value and weak controls.
- **S&P Global:** 42% of companies abandoned most of their AI initiatives, and 46% of proofs of concept never reached production.
- **MIT (2025, method disputed):** buying from a partner succeeded about 2× as often as building in-house.
- **The usual causes:** silent breakage (expired logins, renamed fields), maintenance nobody pays for, and platform price changes.

### Market signals ([Eurostat 2025](https://ec.europa.eu/eurostat/statistics-explained/index.php?title=Use_of_artificial_intelligence_in_enterprises))

| Enterprises using AI | EU | Bulgaria |
|---|---|---|
| All (10+ staff) | 20% | **8.5%** (3rd lowest in the EU) |
| Small (10–49) | 17% | 7.2% |
| Medium (50–249) | 30% | 13.3% |

- Of Bulgarian firms that considered AI but didn't adopt it: **lack of skills 73%**, unclear legal consequences 50%, privacy 48%.
- **EU AI Act for companies using AI:** staff AI literacy (Art. 4), and telling people when they're dealing with AI (Art. 50).
- **Watch out:** per-employee "reply speed" scoring could count as **high-risk worker monitoring** (rules apply from Dec 2027). Frame the email agent as workload routing, never as scoring people.

---

## 7. Where we can win

**Don't compete here** (already good enough off the shelf):
- AI drafts and summaries inside one inbox.
- Q&A over docs and email inside one ecosystem (Copilot, Gemini, ChatGPT, Claude).
- Agents inside HubSpot alone (Breeze).
- Simple 2–3 step automations (Zapier, Make).

**Compete here** (where a technical agency clearly adds value):
1. **Linking systems together:** email thread + CRM deal + unpaid invoice + order + ticket, in one answer or one alert.
2. **Local systems with no connector:** Microinvest, Econt, Speedy, inv.bg, Viber, NRA e-invoicing from 2028.
3. **Team-wide follow-up without changing inboxes.**
4. **Approval rules:** what needs a Confirm, who approves, limits, audit log. The platforms give a switch, not the rules.
5. **Running it:** monitoring, fixing breakage, model updates, cost caps. This is the missing piece behind most failures.
6. **Compliance:** GDPR data processing agreement, data protection impact assessment, EU hosting, AI literacy session.

### Us vs Mission Point

| | Mission Point | Sofyma (proposed) |
|---|---|---|
| Proof | Illustrative scenarios only | Us first, then 2 named pilot clients **with numbers** |
| Prices | None published | **Published packages** |
| Connectors | "Confirmed on the first call" | **Public "works with" list** |
| Who builds it | Consultants; developer capacity unclear | In-house developers who have already built ERPs and CRMs |
| Timeline | "1–3 weeks" | **1-week read-only pilot**, then 2–4 weeks per packaged agent. Honest. |
| Lock-in | Their own closed app | **Connectors built as MCP servers.** They also work with the client's Copilot, ChatGPT or Claude. |
| Process skills | Strong | Weak. **Build a discovery template.** |
| Sales channels | Media and PR | Existing clients + **white-label through UK agency partners** |

---

## 8. What to offer (practical and easy to roll out)

### Offer ladder

Prices are starting guesses based on the market anchors above. Test them with pilot clients.

| # | Offer | What the client gets | Time | Price idea |
|---|---|---|---|---|
| 1 | **Read-only pilot** | Connect 2–3 of their systems read-only. The team asks questions in Slack, Teams or Viber for a week. We finish with a short report: top 3 processes to fix, with estimated hours saved. | 1–2 wks | €1–2k, **credited toward the build** |
| 2 | **Packaged agents** (fixed scope) | One job done end to end, with approvals and a log. See the list below. | 2–4 wks | Setup €2.5–6k + €250–600/mo |
| 3 | **Connected layer** (One-Click equivalent) | Several agents + chat assistant + alerts + approvals + monthly report | 6–10 wks | By scope (anchor: Digital Agent's typical €19k + €1.4k/mo) |
| + | **Run it** retainer | Monitoring, fixes, model updates, cost cap, monthly "what the AI did" report | ongoing | Included in the monthly fee |
| + | **Compliance pack** | GDPR data processing agreement, impact assessment template, EU hosting, 1 AI literacy session | — | In setup, or ~€1k |

### Packaged agents (first candidates)

| Agent | For | Why it sells | Systems |
|---|---|---|---|
| **Team follow-up** | Any team on Gmail or Outlook | "Nothing falls through the cracks." Tracks threads waiting on us across the team, with a daily digest and drafts that use live data. **Not** "AI writes replies". | Gmail / M365 / IMAP + Slack, Teams or Viber + CRM or tickets |
| **Unbilled time → draft invoices** | Agencies, consultancies | Hard money: 57% of agencies lose $1–5k a month | Redmine, Jira, Harvest → FreeAgent, Xero, QuickBooks |
| **Order and delivery exceptions** | Online stores | Stuck parcels, cash-on-delivery not collected, "where's my order?" answered automatically | WooCommerce + Econt, Speedy, BOX NOW + inv.bg |
| **Overdue invoice chasing** | Everyone | Fast, measurable cash result | FreeAgent, Xero, inv.bg, Microinvest |
| **Client email → ticket** | Agencies, service companies | Ends the copy-paste | Gmail → Redmine, Jira, HubSpot tickets |

### "Works with" list (publish it)

| Tier | Systems |
|---|---|
| **v1** | Gmail, Google Workspace, Slack, HubSpot, WooCommerce, Redmine, FreeAgent, Xero, Econt, Speedy, inv.bg |
| **v2** | Microsoft 365 / Teams, Viber, Jira, Asana, ClickUp, Odoo, QuickBooks, BOX NOW |
| **Advanced** (on-premises bridge) | Microinvest; file-based Ajur and Plus Minus |
| **Watch** | NRA e-invoicing (if the 2028 mandate passes) |

### How we build it (keeps delivery cheap and repeatable)

- **One core**, deployed per client in the EU: Postgres, the approval and audit log, the chat bot, scheduled checks.
- **Each connector built once, as an MCP server,** and reused. Official MCPs where they exist (HubSpot, Slack, Google, Xero).
- **n8n for simple flows**, code for anything critical.
- **Claude for reasoning.** Numbers always come from data, never from the model.

### Sales channels

1. **Existing direct clients:** WordPress, WooCommerce and HubSpot sites. Pitch: "we built your site, now we connect it to the rest of your business."
2. **UK agency partners, white-label.** They resell to their clients and we deliver. **No Bulgarian competitor has this channel.**
3. **Agencies as clients.** They have the same problems we do, and we speak their language.

---

## 9. Recommended first niche

**Start with agencies and professional services. Make Bulgarian WooCommerce stores the second niche.**

**Why agencies first**
- We *are* one, so we're client #0 and the first case study.
- Every tool has a good API or an official MCP, so it's the **easiest to build.**
- There's a clear money problem (unbilled time).
- **UK partners already trust us and pay UK prices.**

**Why WooCommerce stores second**
- WooCommerce is 46% of Bulgarian stores, and we know WordPress.
- The couriers have APIs.
- A Microinvest bridge would set us apart.
- **But** Bulgarian clients pay Bulgarian prices and need more convincing (8.5% use AI).

This is a judgment call. If the partners show no interest in reselling, flip the order.

---

## 10. Risks

- **We're a small team.** A custom deployment per client doesn't scale. Package early and say no to one-offs.
- **Process discovery isn't our skill yet.** Practise it on our own team first.
- **Big platforms keep improving.** Copilot and Gemini will absorb the simple cases, so stay on cross-system work and local systems.
- **The Microinvest bridge** means installing software on client machines: support and security work.
- **Viber costs money** (bot fee plus messages). Price it into the package.
- **E-invoicing is still a draft** (consultation until 23 Oct 2026). Watch it before investing.
- **Don't build employee-scoring features** (EU AI Act high-risk rules).

---

## 11. Next steps

1. **Internal:** move credentials out of Redmine, run discovery with the team, build team follow-up for ourselves, and **measure before and after.** That's case #1.
2. **Ask 2–3 UK partners** whether they'd resell an "AI ops" add-on. That decides which niche comes first.
3. **Draft a public "works with" page and a price list.**
4. **Line up 2 pilot clients** at a discount, in exchange for a named case study with numbers.
5. **Build the shared core** while doing the pilots: connector library, approval log, chat bot.
