# Xandeum pNode Analytics

![Xandeum pNode Analytics dashboard](docs/dashboard-screenshot.jpg)

Xandeum pNode Analytics is a network monitoring dashboard for exploring pNode gossip health, response performance, and storage capacity. It is designed as a portfolio project that demonstrates data modeling, analytical metrics, API design, and interactive dashboard UX.

## Why this project

Network operators need to quickly answer:

- How many nodes are available right now?
- Is latency getting worse over time?
- Which nodes need attention?
- How much storage is being used?
- How does the current snapshot compare with recent history?

The dashboard answers those questions with a reproducible seeded dataset. The data source is intentionally labeled in the UI so the project remains honest and easy to demo without requiring a live RPC endpoint.

## Features

- Deterministic 32-node demo dataset with stable IDs, versions, statuses, latency, uptime, and storage values
- Historical snapshot endpoint with 24-hour, 7-day, and 30-day ranges
- Interactive time-series chart for availability, p95 latency, and network load
- Derived KPI cards for:
  - Active nodes and availability
  - p95 latency and average latency
  - Storage utilization
  - Network load
- Node health donut chart
- Latency distribution chart
- Storage footprint ranking
- Shared health filters that update both charts and the node table
- Node search by ID, IP address, version, or status
- CSV export of the currently filtered node list
- Responsive dark dashboard UI with loading and empty states

## Architecture

```text
┌────────────────────┐
│ Seeded demo source │
│ 32 reproducible    │
│ pNode records      │
└─────────┬──────────┘
          │
          ▼
┌────────────────────┐
│ Express API        │
│ snapshot + history │
│ metric summaries   │
└─────────┬──────────┘
          │ JSON
          ▼
┌────────────────────┐
│ XandeumClient      │
│ React Query cache  │
└─────────┬──────────┘
          │
          ▼
┌────────────────────┐
│ React dashboard    │
│ Recharts + table   │
└────────────────────┘
```

The data layer is deliberately isolated in `server/routes.ts`. To connect a live pNode source later, replace the seeded adapter while keeping the API contract consumed by the frontend.

## API endpoints

### `POST /api/pnode/get-pods`

Returns the current discovered node snapshot.

```json
{
  "pods": [],
  "total_count": 32,
  "source": "seeded-demo"
}
```

### `POST /api/pnode/get-stats`

Returns the current network summary and derived metrics.

```json
{
  "totalNodes": 32,
  "activeNodes": 28,
  "averageLatency": 94,
  "p95Latency": 165,
  "availability": 91.3,
  "storageUtilization": 43.3,
  "networkLoad": 50,
  "usedStorage": 13846,
  "totalStorage": 32000,
  "source": "seeded-demo"
}
```

### `POST /api/pnode/get-history`

Returns historical snapshots for the selected range.

```json
{
  "range": "24h",
  "snapshots": [],
  "source": "seeded-demo"
}
```

Supported ranges are `24h`, `7d`, and `30d`.

## Metric definitions

- **Availability**: active nodes count as 100%, syncing nodes as 60%, and offline nodes as 0%, averaged across the current snapshot.
- **p95 latency**: the latency value at the 95th percentile of the sorted node latency distribution.
- **Storage utilization**: total used storage divided by total capacity.
- **Network load**: a derived demo signal based on average latency and availability. It is labeled as derived rather than presented as a direct infrastructure measurement.

## Local development

### Requirements

- Node.js 20+
- npm

### Run the app

```bash
npm install
npm run dev
```

The development server runs on port `5000`.

### Validate the project

```bash
npm run check
npm run build
```

## Project structure

```text
client/src/
├── components/
│   ├── network-analytics.tsx   # Charts, filters, and derived visual analytics
│   ├── network-stats.tsx       # KPI cards
│   └── node-list.tsx           # Filtered node table
├── lib/
│   └── xandeum.ts              # Typed client and API contracts
└── pages/
    └── dashboard.tsx           # Dashboard composition and CSV export

server/
└── routes.ts                   # Seeded nodes, summaries, history, and API routes
```

## Resume-ready project summary

> Built a React and Express network analytics dashboard that models pNode gossip telemetry, calculates p95 latency, availability, and storage utilization, and visualizes historical snapshots with interactive Recharts charts. Added shared chart-to-table filtering, CSV export, typed API contracts, and responsive loading/empty states.

## Future improvements

- Replace the seeded source with a live pNode RPC adapter
- Persist snapshots in PostgreSQL for production history
- Add alert thresholds for high p95 latency and low availability
- Add node detail pages with historical per-node performance
- Add automated tests for metric calculations and CSV output