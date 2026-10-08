# ServiceGrid

**Industrial Servicing Command Center**

Team Thunderbolts, 2026

ServiceGrid is a centralized platform that streamlines industrial equipment maintenance: from service requests and technician allocation to resource management, field execution, and verified closure. It replaces disconnected spreadsheets, emails, and chat groups with one connected application that manages machines, sites, requests, technicians, spare parts, documents, approvals, and service history.

Live demo: https://servicegrid-vercel.vercel.app/

---

## Table of Contents

1. [Problem Statement](#problem-statement)
2. [Existing Challenges](#existing-challenges)
3. [Proposed Solution](#proposed-solution)
4. [Objectives](#objectives)
5. [Users and Role-Based Access](#users-and-role-based-access)
6. [Core Entities](#core-entities)
7. [End-to-End Workflow](#end-to-end-workflow)
8. [Beyond the Core Brief](#beyond-the-core-brief)
9. [Demo Walkthrough](#demo-walkthrough)
10. [System Architecture and Extensibility](#system-architecture-and-extensibility)
11. [Tech Stack](#tech-stack)
12. [Installation and Setup](#installation-and-setup)
13. [Configuration](#configuration)
14. [Project Structure](#project-structure)
15. [Testing and Quality](#testing-and-quality)
16. [Deployment](#deployment)
17. [Scope and Limitations](#scope-and-limitations)
18. [Future Scope](#future-scope)
19. [Team](#team)

---

## Problem Statement

Industrial equipment service operations rely on disconnected systems to record and communicate service requests. This makes it hard to match the right technician, tools, and spare parts to each job, and offers little real-time visibility into technician assignments, machine status, and pending work. Delays and resource conflicts surface only after they disrupt the schedule, while service reports, photos, and completion records are collected and verified manually.

The result is delayed maintenance, inefficient resource use, duplicated work, and no reliable machine service history.

## Existing Challenges

- Limited real-time visibility into technician assignments, machine conditions, service progress, and pending requests
- Difficulty identifying suitable technicians and spare parts quickly
- Fragmented information that makes machine service history hard to reconstruct
- Delays, resource conflicts, and SLA violations that are discovered too late

## Proposed Solution

One connected platform that carries a request through its full lifecycle: created, validated, assigned to the right technician, monitored during execution, then verified and closed, all in the same system. Managers and field staff get real-time visibility into machine status, ongoing activities, technician assignments, and pending requests.

## Objectives

- Make maintenance more efficient, organized, transparent, and easy to monitor
- Support the complete request lifecycle: creation, validation, approval, resource allocation, technician assignment, field execution, completion verification, and closure
- Give managers real-time visibility into which machines need attention, which technicians are available, which requests are active, and which are approaching their SLA limits
- Reduce manual coordination by automating notifications, assignment workflows, and exception alerts
- Keep a centralized, complete service history for every machine

## Users and Role-Based Access

Each user receives access according to their responsibilities.

| Role | Responsibilities |
|------|------------------|
| Customer / Operations user | Create service requests, provide machine information, monitor request progress |
| Operations Manager | Validate and approve requests, assign technicians, monitor the overall service process, verify completion |
| Technician | View assigned tasks, accept jobs, update status, record work, report problems, upload photos and completion evidence |
| Administrator | Manage users, machines, sites, technicians, inventory, SLA targets, and system configuration |

Authorization is enforced through a permission matrix, so a technician cannot change administrative settings while an administrator has broader management privileges. The matrix is visible in Admin Settings.

## Core Entities

Sites, machines, service requests, technicians, spare parts, tasks, documents, approvals, and service history are linked to one another.

- Each machine belongs to a site and keeps a record of its previous maintenance activity
- Each service request carries the affected machine, location, problem description, priority, required skills, required parts, assigned technician, and current status
- Technician records hold skills, certifications, availability, workload, and assignments
- Inventory records track spare-part stock and reservations per site
- Photos and attachments are attached to individual requests

## End-to-End Workflow

```
Create -> Validate -> Approve -> Assign -> Execute -> Verify -> Close
                         |          |         |
                         +----------+---------+--> Exception (dropout, part shortage, SLA breach)
                                                   -> corrective action -> resume
```

### 1. Service request creation
Customers or operations users raise a request against a machine with a description, priority, and required parts. Free-text descriptions can be pre-filled using AI parse.

### 2. Validation and routing
The system checks that the machine exists, is eligible (warranty or contract), and that the required parts are in stock, then produces a validation report. Technicians are ranked on skills, availability, location, and workload, with urgency reflected in the priority and SLA target.

```
score = 0.40 * skill match
      + 0.25 * proximity        (haversine distance, normalized over 120 km)
      + 0.20 * availability     (offline technicians score zero)
      + 0.15 * (1 - workload)
```

The full score breakdown is shown, so every assignment is explainable.

### 3. Resource and conflict management
Required spare parts are reserved atomically before assignment, so stock cannot be oversold. Technician and resource conflicts are detected and raised as exceptions instead of letting both assignments proceed silently.

### 4. Approval and assignment
A manager reviews and approves, rejects, or returns a request, and rejected requests can be resubmitted rather than lost. On approval, the best-ranked technician and reserved resources are assigned and the technician is notified. A timeline records when the request was created, validated, approved, and assigned.

### 5. Technician task execution
The technician accepts the task, moves it through In Progress and Pending Verification, and records work notes, parts consumed, and photo evidence. A geofence indicator (3.2 km site radius) shows whether the technician is on site or en route. Notes written offline are queued locally and sent when the connection returns.

### 6. Exception handling and SLA monitoring
- **Technician dropout:** the exception is flagged and a one-click reassign picks the next-best qualified technician
- **Part unavailable:** an alert is raised with a nearest-site transfer suggestion
- **SLA monitoring:** default targets of P1 4h, P2 24h, P3 72h, P4 168h (configurable by admins), a warning at 80% elapsed, and a breach alert at 100%

### 7. Completion verification and service history
The technician submits completion details and evidence. An authorized manager reviews it and verifies before the request is closed. Everything stays in the machine's service history (requests, completed work, technicians involved, parts used, and attachments) and in the audit trail.

### 8. Real-time dashboard and notifications
A single command center shows pending, active, and completed requests, machines needing attention, available technicians, SLA risks, and resource shortages on a live map with an activity feed. Notifications fire when requests are created, approved, assigned, reassigned, delayed, or completed, and can be forwarded to a webhook.

## Beyond the Core Brief

- **Dispatch Optimizer:** picks the best valid assignment, not just the nearest technician, by combining ranking, road ETA (OSRM), part availability, conflicts, and SLA fit (fit, tight, miss)
- **AI Insights:** fault diagnosis (likely issue, confidence, checks, parts, repair window), technician recommendation, predictive maintenance (health score, risk, likely component), and service summaries
- **Ask ServiceGrid assistant:** answers questions such as "Which requests are at risk?" using only live workspace data, and says so when it does not know
- **IoT Monitoring:** simulated temperature, vibration, pressure, and motor-current telemetry with warning and critical thresholds; a breach raises an emergency P1 request through the normal workflow
- **Simulation Center:** triggers dropout, part depletion, SLA fast-forward, and IoT alerts, all through the real workflow engines
- **Hash-chained audit log:** every action is chained, and one click recomputes the chain to verify integrity
- **Optimistic locking:** stale updates are rejected instead of silently overwriting newer changes

## Demo Walkthrough

Sign in under any of the four domains. In demo mode, any valid email with a password of four or more characters works. Suggested accounts:

| Domain | Email |
|--------|-------|
| Operations Control | operator@servicegrid.demo |
| Field Technician | tech@servicegrid.demo |
| Customer / Site | customer@servicegrid.demo |
| Governance and Admin | admin@servicegrid.demo |

**Two-minute path (Operations):**

1. **Overview:** map, KPIs, and live feed across five sites. The hero asset is M-104, Conveyor Line C-4, which is down.
2. **Service Requests:** open a request to show the validation report, technician score breakdown, part reservation, task log, and audit trail.
3. **Simulation Center:** run *Simulate technician dropout*, then use one-click reassign.
4. **Simulation Center:** run *Deplete reserved part* to see the nearest-site transfer suggestion, then *Fast-forward SLA* to trigger a breach.
5. **Dispatch Optimizer:** compare candidates by ETA, parts, conflicts, and SLA fit.
6. **IoT Monitoring and AI Insights:** trigger an alert, then show diagnosis and predictive maintenance.
7. **Audit History:** click **Verify integrity**.
8. Switch to the **Customer** and **Technician** domains to show the same request from each side.

## System Architecture and Extensibility

The solution is modular. The frontend serves each user role, the workflow engine enforces business rules and the state machine, and external concerns sit behind typed adapter interfaces so each can be swapped without touching the workflow.

```
   Customer / Operations / Technician / Admin (role-based UI)
                            |
                            v
                   services/store.ts
     (state machine, transactions, RBAC call sites)
                            |
        +-------------------+-------------------+
        |                   |                   |
  lib/stateMachine     lib/allocation      lib/validation
  lib/geo (SLA,        (skill, distance,   lib/schemas (Zod)
  geofence, hash)      load scoring)
        |
        v
    Event bus -> live feed, notifications, exceptions
        |
        v
  adapters: notification | AI | IoT | map | ledger | (mobile, queue stubs)
```

**Request state machine**

```
CREATED -> VALIDATED -> PENDING_APPROVAL -> APPROVED -> ASSIGNED
        -> IN_PROGRESS -> PENDING_VERIFICATION -> CLOSED

Any active state -> EXCEPTION -> back to APPROVED / ASSIGNED / IN_PROGRESS
Also: REJECTED and CANCELLED, with resubmit and reopen paths
```

| Adapter | Current implementation | Extension path |
|---------|------------------------|----------------|
| Notification | In-app plus optional webhook | Per-user preferences |
| AI | Deterministic mock provider, optional Claude parsing | Hosted model provider |
| IoT | Simulated telemetry with threshold-to-P1 routing | MQTT and real sensors |
| Map | Leaflet, OpenStreetMap, OSRM road routing | Geofenced dispatch |
| Ledger | SHA hash chain with verify | External anchoring |
| Mobile, queue | Interfaces and stubs | Native app, Kafka or SQS |

## Tech Stack

| Layer | Technology |
|-------|------------|
| Frontend | React 19, TypeScript, Vite |
| Styling and UI | Tailwind CSS, Radix UI primitives, lucide-react |
| Maps and routing | Leaflet, OpenStreetMap, OSRM |
| Validation | Zod |
| State and persistence | Custom store with browser localStorage |
| AI | Typed provider layer (mock by default, optional Claude API) |
| Testing and quality | Vitest, oxlint, GitHub Actions CI |
| Deployment | Vercel (static), Docker with nginx |
| Production data model | Prisma schema for PostgreSQL (included) |

## Installation and Setup

### Prerequisites

- Node.js 22 or later
- npm

### Run locally

```bash
git clone https://github.com/<your-username>/<your-repo>.git
cd <your-repo>
npm install
npm run dev
```

The app runs at `http://localhost:5173`. Use **Reset demo** inside the app to restore the seed data.

### Scripts

| Command | Purpose |
|---------|---------|
| `npm run dev` | Start the development server |
| `npm run build` | Type-check and create a production build |
| `npm run preview` | Preview the production build |
| `npm test` | Run the unit tests |
| `npm run lint` | Lint with oxlint |

### Seed data

Five sites (Guindy, Ambattur, Sriperumbudur, Oragadam, Chennai Central Warehouse), 12 machines, 10 technicians with skills and certifications, and 10 spare-part SKUs stocked across the sites.

### Simulate an IoT breach

```bash
python scripts/simulate_iot.py --machine M-104 --metric vibration --value 128
```

This prints the telemetry payload and reports whether the value breaches the threshold. Inside the app, the same flow runs from the Simulation Center or IoT Monitoring page.

## Configuration

Every variable is optional, and the app works fully offline with none set. Copy `.env.example` to `.env` to override.

| Variable | Purpose |
|----------|---------|
| `VITE_AI_ENABLED` | Turn the AI layer on or off (default on) |
| `VITE_AI_PROVIDER` | AI provider id (default `mock`) |
| `VITE_AI_API_URL` | Reserved endpoint for a future provider |
| `VITE_CLAUDE_API_KEY` | Enables Claude-based complaint parsing in the customer portal |
| `VITE_NOTIFY_WEBHOOK` | Forwards notifications to a webhook |

Variables prefixed with `VITE_` are exposed to the browser. Do not use a production secret as a frontend key; route such calls through a server in production.

## Project Structure

```
.
|-- prisma/schema.prisma        PostgreSQL data model (production path)
|-- scripts/simulate_iot.py     IoT telemetry simulator
|-- public/                     Icons and web manifest
|-- src/
|   |-- App.tsx                 Shell, navigation, role-based pages
|   |-- adapters/               Typed adapter interfaces and implementations
|   |-- components/
|   |   |-- ai/                 Diagnosis, recommendation, predictive, assistant
|   |   |-- auth/               Domain sign-in
|   |   |-- shell/              Pages, map, drawer, optimizer, simulation, IoT
|   |   `-- ui/                 Reusable UI components
|   |-- data/seed.ts            Sites, machines, technicians, parts
|   |-- lib/                    State machine, allocation, geo, validation, schemas
|   |-- services/
|   |   |-- store.ts            Workflow engine and persistence
|   |   |-- ai/                 AI service, mock provider, assistant
|   |   |-- iot/                Simulated telemetry
|   |   `-- offlineOutbox.ts    Offline note queue
|   `-- types/                  Domain types
|-- Dockerfile                  Production image (nginx)
|-- vercel.json                 Vercel build and SPA fallback
`-- .github/workflows/ci.yml    Test and build on every push
```

## Testing and Quality

- Unit tests (Vitest) cover the state machine and optimistic locking, SLA and geofence math, the allocation engine, request validation, the AI layer (including the assistant never inventing records), and telemetry determinism.
- GitHub Actions runs `npm test` and `npm run build` on every push to `main` and on pull requests.
- The codebase is fully typed, and inputs are validated with Zod.

## Deployment

### Vercel

1. Push the repository to GitHub.
2. Import it in Vercel. The framework is detected as Vite, and `vercel.json` sets the build command, output directory, and SPA rewrite.
3. Deploy. No server, database, or API key is required.

### Docker

```bash
docker build -t servicegrid .
docker run -p 8080:80 servicegrid
```

## Scope and Limitations

This is a working demo with a defined production path, and the boundaries are stated plainly:

- **Data** is stored in the browser (localStorage), not a shared server. The Prisma schema mirrors the domain model and is the planned database layer; it is not yet connected to the running app.
- **Authentication** is demo-mode role selection. Production would use JWT-based credentials with the same permission matrix enforced on server routes.
- **AI** runs on a deterministic mock provider by default so the demo is reliable offline. Claude-based parsing activates only when a key is configured.
- **IoT telemetry** is simulated and labeled as such. The script prints the ingest payload; a backend ingest route is part of the production plan.
- **Technician status** is tracked as Assigned, In Progress, and Pending Verification, with on-site presence derived from the geofence. Finer-grained steps such as Travelling are planned.
- **Resources** currently cover spare parts. Tool tracking is planned.

## Future Scope

- Server-side API with PostgreSQL (Prisma) and credential-based authentication
- Real-time transport (SSE or WebSocket, later a queue such as Kafka or SQS)
- Live IoT ingestion over MQTT with per-asset thresholds
- Hosted AI provider behind a server proxy for predictive maintenance and recommendations
- Tool and equipment tracking alongside spare parts
- Finer technician status stages (Accepted, Travelling, On Site, Work in Progress, Completed)
- Native mobile app for technicians
- Anchoring the audit chain root hash to an external ledger

## Team

**Team Thunderbolts**

| Name |
|------|
| A. Guru Charan |- Team Lead
| Harshith Venkataraman |
| Pranav Charan B. |
| Sahana Balaji |
| Sivanee R. |

Built for ThunderBolts, 2026.
