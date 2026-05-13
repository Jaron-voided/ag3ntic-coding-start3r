# Product Requirements Document — EdgeDesk

> **Status:** Draft
> **Last updated:** 2026-05-07
> **Author:** Jaron Singley
> **Stakeholder:** Jaron's automation and web services clients

---

## 1. The problem

Owner-operators of trades businesses (HVAC, construction, contracting) are buried in admin work on top of their actual jobs. They miss high-value calls because they're on a job site. They lose leads because there's no follow-up system. They can't afford a full-time assistant, and they don't have time to learn 10 different tools. Jaron already sells an automation stack that solves this — Twilio for missed-call lead capture, Airtable for CRM, n8n for orchestration, and Facebook auto-posting with AI-generated captions. The problem is delivery: clients don't have a clean, single place to interact with these services. EdgeDesk fixes that.

---

## 2. The user

- **Primary user:** Owner-operator of a trades business (HVAC, contractor, roofer, etc.). Runs the business solo or with a small crew. High-ticket jobs, tight margins on admin time. Can't afford a receptionist. Not tech-savvy — if it takes more than a few taps, they won't use it.
- **Their current workflow:** Miss a call → maybe call back later → lead goes cold. Posting to Facebook means logging in, writing something, hoping it looks professional. Customer list lives in their head or a notes app.
- **Their technical comfort:** Low. Comfortable with smartphones. Will use a web app for demo purposes; eventually mobile-first.
- **What device will they use it on?** Phone in the real world. Desktop browser for v1 demo.
- **The moment they open EdgeDesk:** They've got 10 minutes between jobs. They want to know who called, who's waiting on a quote, and whether their Facebook post went out.

---

## 3. What success looks like

- **Must-have outcome:** A client can log in, see their captured leads, upload a photo that automatically posts to Facebook, and access their deliverables — without touching Airtable, n8n, Gmail, or any other backend tool.
- **Nice-to-have outcome:** Client manually adds a customer and it shows up in their lead list instantly.
- **Not a goal for v1:** Analytics dashboards, revenue graphs, customer trend charts, multi-tenant client isolation, status monitoring for automations, or replacing any of the existing automation logic in n8n.

---

## 4. Core user stories

1. **[Must]** As a client, I want to upload a photo to EdgeDrive so that it automatically posts to my Facebook page with an AI-generated caption — without me writing copy or logging into Facebook.

2. **[Must]** As a client, I want to see the leads my automations captured (via Twilio SMS, web form, or website) as a simple Todo list so that I know who to follow up with without digging through texts, voicemails, or Airtable.

3. **[Must]** As a client, I want to access my deliverables and important links (my website, intake forms, assets Jaron has built) from one place so that I don't have to dig through emails to find them.

4. **[Should]** As a client, I want to manually add a customer to my lead list so that people I meet in person are tracked alongside automated leads.

5. **[Won't — v1]** As a client, I want to see an analytics dashboard showing customer upticks and revenue trends over time.

6. **[Won't — v1]** As a client, I want to see real-time status indicators for each active automation (e.g., "missed call texting: active").

---

## 5. Out of scope

- **Analytics and reporting** — graphs, revenue trends, customer volume charts. Requires financial data connections that don't exist yet.
- **Automation status monitoring** — showing whether n8n workflows are healthy. Important, but not needed to sign client #1.
- **Multi-tenancy** — v1 assumes one client. Client isolation (Client A can't see Client B's data) is a v2 concern.
- **Admin panel UI** — if something breaks, Jaron accesses the database directly. No built admin interface for v1.
- **Full business OS** — leads, tasks, calendar, documents as a general small business management suite. That is a future phase, not v1.
- **Mobile app** — v1 is a desktop web app. Mobile-responsiveness is a stretch goal, not a requirement.

---

## 6. Technical shape

- **Type of app:** Full-stack web app. React frontend, Workers API backend.
- **Does it need to store data?** Yes — file uploads (photos), deliverables/links, manually added customers. Lead data pulled from Airtable in v1; migrating to native D1 in v2.
- **Does it need authentication?** Yes — single client login for v1 (username + password, JWT-based). No multi-tenancy or role management needed yet.
- **Does it need to call external services?**
  - **Airtable API** — read leads/customers for the Todo list
  - **n8n webhook (live)** — Workers fires a POST after a photo is uploaded to R2; n8n handles AI caption + Facebook post
- **Who pays for hosting?** Jaron. Cloudflare's free tier covers v1 comfortably.

### Proposed Cloudflare stack

| Need | CF Product | Why |
|---|---|---|
| Frontend hosting | **Pages** | Deploy React + Tailwind globally, zero config, free tier |
| API / backend logic | **Workers + Hono** | Handle file uploads, Airtable fetches, auth, n8n webhook triggers |
| File storage (EdgeDrive) | **R2** | Store uploaded photos; no egress fees; fires event to Worker on upload |
| App data (deliverables, manual leads, sessions) | **D1** | Serverless SQL — stores links, manually added customers, JWT sessions; future home of all lead data |
| Session / auth tokens | **KV** | Fast global key-value store for lightweight session lookups |
| Lead data (v1 only) | **Airtable API** | Pull existing leads — migrate to D1 in v2 |
| AI captions + Facebook posting | **n8n (existing, live)** | Already built and working — Workers fires webhook, n8n does the rest |

---

## 7. Risks and unknowns

- **Biggest risk — Desktop UI metaphor:** Building a convincing "app launcher" experience (clickable icons that open panels or windows) in a web app is a UX challenge without a standard library solution. Needs a design decision before coding starts. → Tackle in `/design-brief` before writing any UI.
- **n8n reliability:** If an automation fails silently after a webhook fires, the client sees no feedback. This is an n8n-side gap to close (error handling, retries) before onboarding real clients — not an EdgeDesk v1 feature.
- **Airtable API rate limits:** Airtable's free tier has rate limits (5 req/sec). Fine for one client demo; becomes a problem at scale. Migrating to D1 in v2 removes this dependency entirely.
- **Assumption to verify:** That the live n8n instance's webhook URL is stable and doesn't require auth headers that would complicate the Worker-to-n8n call.

---

## 8. Milestones

- **Week 1 end:** Project scaffolded on Cloudflare Pages + Workers. Single-client auth working (login → JWT → protected routes). Airtable API connected and returning real leads. Deployed to a live URL — ugly but functional. You can log in and see a real lead list in a browser.

- **Week 2 end:** EdgeDrive upload working (R2 storage → Worker → n8n webhook fires → Facebook post triggered). All three app panels functional: Lead/Todo list, EdgeDrive upload, Deliverables/Links page. Everything wired to real data. Showable to a real person as a working prototype.

- **Week 3 demo:** Desktop homepage with app-launcher UI (clickable icons opening panels). Polish pass on layout and typography. Full demo flow works end-to-end: log in → see desktop → click CRM icon → see lead list → click EdgeDrive icon → upload photo → automation fires → Facebook post goes out.
