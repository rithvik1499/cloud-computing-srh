# Architecture: Greenleaf Foods (SHP04)

Targets: freshness 60 s, reliability 65% on time on every lane, budget $30/month. Ocean only.

```mermaid
flowchart LR
  subgraph BATCH["BATCH: daily files, scheduled"]
    G1["Event generator<br/>rates, schedules, bookings files"] --> RAW[("Cloud Storage<br/>raw bucket")]
    RAW --> LOAD["bq load<br/>+ scheduled queries"]
    LOAD --> CUR[("BigQuery<br/>curated tables")]
  end

  subgraph LIVE["LIVE: within 60 s"]
    G2["Event generator<br/>ocean shipment events"] --> PS["Pub/Sub topic"]
    PS --> DF["Dataflow<br/>streaming job"]
    DF --> EV[("BigQuery<br/>events table")]
    DF --> ARC[("Cloud Storage<br/>raw events archive<br/>for replay")]
  end

  EV --> DASH["Ops dashboard<br/>vessel slips and delays"]
  CUR --> REL["Reliability report<br/>on-time % per lane vs 65%"]
  CUR --> DASH

  subgraph REMOVED["Removed from reference platform"]
    K["Kafka on Kubernetes<br/>too expensive for $30/month"]
    AIR["Air freight path<br/>Greenleaf is ocean only"]
    SPK["Dataproc / Spark<br/>data is small, BigQuery is enough"]
  end

  style LIVE fill:#fde2e2,stroke:#c0392b
  style BATCH fill:#e2eefd,stroke:#2e6fbf
  style REMOVED fill:#eeeeee,stroke:#999999,stroke-dasharray: 5 5
  style K stroke-dasharray: 5 5
  style AIR stroke-dasharray: 5 5
  style SPK stroke-dasharray: 5 5
```

| Box | Live or batch | Why |
|---|---|---|
| Shipment events, Pub/Sub, Dataflow, events table | Live | Warehouse teams re-plan when a vessel slips; freshness target is 60 s |
| Ops dashboard | Live | Reads the events table |
| Rates, schedules, bookings files, curated tables | Batch | Arrive as daily files; no sub-day need in the profile |
| Reliability report | Batch | On-time % per lane vs the 65% target is a historical measure |
