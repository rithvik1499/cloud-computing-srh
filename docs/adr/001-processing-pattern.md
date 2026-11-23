# ADR 001: Lambda, with the speed layer only for shipment events

## Status
Accepted

Date: 2026-11-23

## Context
Shipper: Greenleaf Foods (SHP04), ocean only. Targets: freshness 60 seconds,
reliability 65% on time on every lane, budget $30 per month.

Evidence (docs/metrics.md, October 2026, 114 shipments): three of four lanes miss the
65% target: Transpacific 45.8%, Asia-North Europe 54.8%, Transatlantic 56.7%. Average
delay on those lanes is 33.5 to 48.4 hours. The profile says warehouse teams re-plan
the moment a vessel slips, so late sailings are frequent and the 60-second target applies
to shipment events. Rates, schedules and bookings arrive as one file per day.

## Options considered
1. **Lambda:** a batch layer for the daily files and a speed layer for shipment events.
2. **Kappa:** one streaming pipeline for everything, with a replayable log.
3. **Batch only:** load everything from the daily files on a schedule.

## Decision
We chose **Lambda**, with the speed layer limited to shipment events, because
only events have a freshness target (60 s). Three of the four datasets are
daily files, and streaming them would add no freshness. Batch only cannot meet 60 s.
Kappa would put every dataset through the always-on pipeline and the replay log
without a freshness gain.

## Consequences
- **Cost:** not costed yet. The batch layer runs inside BigQuery (load and scheduled
  queries, no cluster). The always-on streaming job (Pub/Sub and Dataflow) is the main
  risk against the $30 budget and must be checked with the pricing calculator.
- **Freshness:** shipment events within 60 s through the speed layer; rates, schedules
  and bookings as of the last daily load.
- **Complexity:** two paths to build and monitor instead of one.
- **What gets harder:** the live delay events and the final actual arrival in schedules
  can disagree. On-time % is therefore reported from the batch tables only. If rates ever
  need to be fresher than daily, they would have to move to the speed layer, and replay
  depends on the raw events archive in Cloud Storage.
