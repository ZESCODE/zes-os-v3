# zes-os-v3

[![Next.js](https://img.shields.io/badge/Next.js-15%2B-black?logo=next.js)](https://nextjs.org/)
[![TypeScript](https://img.shields.io/badge/TypeScript-5.x-blue?logo=typescript)](https://www.typescriptlang.org/)
[![SSE](https://img.shields.io/badge/Realtime-SSE-green)](https://developer.mozilla.org/en-US/docs/Web/API/Server-sent_events)
[![License](https://img.shields.io/badge/License-MIT-orange)](LICENSE)

A real-time workflow dashboard for visualizing multi-stage work, live agent status, flow metrics, and bottlenecks. It uses the Next.js App Router, Server-Sent Events (SSE), and an event-driven data model to keep the UI synchronized with backend state. [web:30][web:22][web:25]

## Overview

Workflow Mesh Dashboard is built for teams that need a live operational view of work moving through stages such as Planning, Research, Coding, Review, and Delivery. It combines a workflow board, KPI strip, event feed, and analytics charts so operators can see both the current state and the historical flow of work. [web:22][web:25][web:81]

The dashboard focuses on core Kanban metrics: lead time, cycle time, throughput, and WIP. These metrics help reveal bottlenecks, describe system capacity, and improve predictability. [web:22][web:25][web:64]

## Features

- Live workflow board with stages, agents, and work items.
- KPI strip for arrival rate, throughput, lead time, and WIP.
- Real-time updates via SSE.
- Item detail drawer and operator actions.
- Analytics helpers for CFD, throughput, and cycle time.
- Event-first storage model for replay and auditability. [web:22][web:25][web:55][web:73]

## Why this exists

Kanban boards alone are useful, but they do not always explain why work slows down. By adding flow metrics and live telemetry, this dashboard makes queues, blocked work, and capacity issues visible in real time. [web:22][web:25][web:71]

## Architecture

The system follows a snapshot-plus-stream model:

1. The server returns a snapshot for initial page load.
2. The client renders the snapshot in a Server Component.
3. The client subscribes to an SSE stream for live updates.
4. The UI applies deltas to local state instead of reloading the whole page.
5. The backend stores canonical state in Postgres and emits events from the workflow engine. [web:55][web:73][web:33]

## Folder structure

```txt
zes-os-v2:workflow-dashboard/
├─ app/
│  ├─ layout.tsx
│  ├─ page.tsx
│  ├─ globals.css
│  ├─ (dashboard)/
│  │  ├─ dashboard/page.tsx
│  │  ├─ workflows/[workflowId]/page.tsx
│  │  ├─ agents/[agentId]/page.tsx
│  │  └─ alerts/page.tsx
│  └─ api/
│     ├─ dashboard/
│     │  ├─ snapshot/route.ts
│     │  └─ stream/route.ts
│     ├─ workflows/[workflowId]/
│     │  ├─ route.ts
│     │  ├─ metrics/route.ts
│     │  ├─ stages/route.ts
│     │  └─ events/route.ts
│     ├─ items/[itemId]/
│     │  ├─ route.ts
│     │  ├─ events/route.ts
│     │  └─ actions/route.ts
│     ├─ agents/[agentId]/route.ts
│     └─ alerts/route.ts
├─ components/
│  ├─ dashboard/
│  │  ├─ DashboardShell.tsx
│  │  ├─ MetricStrip.tsx
│  │  ├─ WorkflowBoard.tsx
│  │  ├─ StageCard.tsx
│  │  ├─ AgentCard.tsx
│  │  ├─ WorkItemCard.tsx
│  │  ├─ EventFeed.tsx
│  │  ├─ TaskDrawer.tsx
│  │  ├─ AlertPanel.tsx
│  │  └─ ConnectionBadge.tsx
│  ├─ charts/
│  │  ├─ CfdChart.tsx
│  │  ├─ ThroughputChart.tsx
│  │  ├─ CycleTimeChart.tsx
│  │  └─ BlockedAgeChart.tsx
│  └─ ui/
│     ├─ Card.tsx
│     ├─ Badge.tsx
│     ├─ Button.tsx
│     └─ Drawer.tsx
├─ lib/
│  ├─ api/
│  ├─ realtime/
│  ├─ metrics/
│  ├─ state/
│  └─ utils/
├─ types/
│  ├─ workflow.ts
│  ├─ agent.ts
│  ├─ item.ts
│  ├─ event.ts
│  ├─ alert.ts
│  ├─ metrics.ts
│  └─ dashboard.ts
├─ db/
│  └─ migrations/
├─ public/
├─ package.json
└─ README.md
````

This structure matches the recommended Next.js App Router convention of keeping routes in `app/`, reusable logic in `lib/`, shared contracts in `types/`, and UI components in `components/`. [web:30][web:34][web:39][web:75]

## Quick start

```bash
pnpm install
pnpm dev
```

Open `http://localhost:3000` in your browser after the app starts.

If you are starting from scratch, you can create a Next.js App Router project with the official starter flow. [web:30][web:34]

## Environment variables

Create a `.env.local` file:

```bash
DATABASE_URL=postgresql://user:password@localhost:5432/workflow_mesh
REDIS_URL=redis://localhost:6379
NEXT_PUBLIC_APP_NAME=Workflow Mesh Dashboard
```


## Data model

The app is event-first. Workflows define the pipeline, work items move through stages, agents perform work, and events record each state transition. That design supports replay, audit logs, live updates, and reliable metric rollups. [web:55][web:73][web:22]

### Main tables

- `workflows`
- `stages`
- `agents`
- `work_items`
- `events`
- `alerts`
- `action_logs`
- `metric_rollups_minute`
- `metric_rollups_hour`


## Metrics

These are the core flow metrics used throughout the UI:

- Lead time: request to delivery.
- Cycle time: start of work to completion.
- Throughput: completed items per time period.
- WIP: items currently in progress.
- CFD: stacked stage occupancy over time. [web:22][web:25][web:64]

Kanban metrics are most useful when they are viewed together, because WIP, cycle time, and throughput influence one another. [web:22][web:81][web:64]

## API routes

### Snapshot

`GET /api/dashboard/snapshot`

Returns the full dashboard state for initial render.

### Stream

`GET /api/dashboard/stream`

Returns an SSE stream of live events.

### Workflows

- `GET /api/workflows/[workflowId]`
- `GET /api/workflows/[workflowId]/stages`
- `GET /api/workflows/[workflowId]/metrics`
- `GET /api/workflows/[workflowId]/events`


### Items

- `GET /api/items/[itemId]`
- `GET /api/items/[itemId]/events`
- `POST /api/items/[itemId]/actions`


### Agents

- `GET /api/agents/[agentId]`


### Alerts

- `GET /api/alerts`
- `POST /api/alerts`

The streaming endpoint should be implemented as a Next.js Route Handler and should use SSE headers such as `text/event-stream`, `no-cache`, and `keep-alive`. [web:55][web:73][web:33]

## TypeScript interfaces

Shared interfaces live in `types/` so the frontend and backend stay in sync.

### Snapshot types

```ts
export interface DashboardSnapshot {
  workflow: Workflow;
  stages: Stage[];
  agents: Agent[];
  items: WorkItem[];
  metrics: DashboardMetrics;
  alerts: Alert[];
  latestEvents: WorkflowEvent[];
  lastEventId: string;
  serverTime: string;
}
```


### Event types

```ts
export interface StreamEnvelope<T = unknown> {
  event: string;
  id: string;
  data: T;
}
```


### Core entities

```ts
export interface Workflow { /* ... */ }
export interface Stage { /* ... */ }
export interface Agent { /* ... */ }
export interface WorkItem { /* ... */ }
export interface WorkflowEvent<TPayload = unknown> { /* ... */ }
export interface Alert { /* ... */ }
export interface DashboardMetrics { /* ... */ }
```


## Real-time behavior

The dashboard uses SSE because it mainly receives server-to-client updates, which makes it simpler than WebSockets for this use case. SSE is a strong fit for live dashboards, progress tracking, and event streams, while WebSockets are usually better when the browser must send frequent real-time messages back. [web:74][web:55][web:33]

## Local development

### Install dependencies

```bash
pnpm install
```


### Run migrations

```bash
pnpm db:migrate
```


### Start the app

```bash
pnpm dev
```


## Production notes

- Keep the SSE route long-lived and lightweight.
- Use heartbeat comments to keep proxies from closing idle connections.
- Preserve event IDs so reconnects can resume using a cursor or `Last-Event-ID`.
- Compute rollups in background jobs rather than inside the request path.
- Use a replay buffer or event store for missed messages. [web:55][web:73][web:58][web:59]


## Recommended workflow

1. Create the database schema.
2. Implement the snapshot endpoint.
3. Implement the SSE stream.
4. Add the dashboard shell and workflow board.
5. Add charts and alerting.
6. Add action logging and auth.
7. Add replay and reconnect support. [web:55][web:33][web:73]

## Contributing

1. Fork the repository.
2. Create a feature branch.
3. Add or update tests.
4. Open a pull request with a clear description of the change.

## License

MIT.

## Acknowledgements

This project design is based on common Kanban flow metrics and modern Next.js App Router streaming patterns. [web:22][web:25][web:30][web:55]
