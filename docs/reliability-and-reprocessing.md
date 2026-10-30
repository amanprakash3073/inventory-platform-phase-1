# Reliability and Reprocessing Design

## 1. Purpose

This document identifies reliability risks in the current Inventory Platform demo and proposes improvements for a more dependable data pipeline.

## 2. Current Data Flow

Python Event Generator → Apache Kafka → Python Processor → PostgreSQL → Grafana

Kafka provides temporary event retention according to its topic configuration. PostgreSQL stores processed business data. Grafana reads stored data for visualization.

## 3. Failure Handling

### Current limitations

- The generator keeps some simulation state in memory.
- The processor keeps important working state, including recent sales and stock values, in memory.
- Restarting the processor can therefore lose working state that has not been reconstructed from persisted data.
- The demo does not implement a complete end-to-end recovery and reconciliation workflow.

### Proposed improvements

- Monitor Kafka consumer lag and service health.
- Add structured logs and alerts for processing failures.
- Define retry policies and a dead-letter strategy for malformed or repeatedly failing events.
- Rebuild processor state from durable event history or a validated database snapshot after restart.
- Test recovery by restarting services and verifying that inventory totals remain correct.

## 4. Duplicate Event Handling

### Risk

Replaying events or retrying writes can create duplicate sales or replenishment records if the same business event is processed more than once.

### Proposed improvements

- Assign a stable unique event ID when each business event is generated.
- Store processed event IDs or enforce suitable database uniqueness constraints.
- Make database updates idempotent so processing the same event again does not change totals incorrectly.
- Test duplicate delivery and replay scenarios.

## 5. Late Events and Event Time

### Proposed policy

- Use the event timestamp to determine when a business event occurred.
- Define an allowed lateness period based on business requirements.
- Use event-time watermarks in a future event-time-aware processing design.
- Specify how events arriving after the allowed lateness period affect historical results.
- Track late events and make any corrections visible to downstream analytics.

**Current status:** A complete watermark and late-event correction policy is not implemented in the demo.

## 6. Seven-Day Kafka Retention Sizing

Estimate storage using:

**Required raw storage ≈ average event rate × average event size × 604,800 seconds**

Then add overhead for replication, indexes or metadata, operational headroom, and any compression assumptions.

For example, at 100 events/second and 1 KB/event:

100 × 1 KB × 604,800 = approximately 604.8 GB of uncompressed event payload over seven days.

This is an illustrative estimate, not a measurement of the current project. Actual sizing must use measured event sizes, event rates, and the configured replication factor.

The current demo uses one broker and one partition per topic. Its seven-day retention requirement has not yet been verified.

## 7. Reprocessing Runbook — Proposed

1. Identify the affected topic, time range, and business metrics.
2. Record current consumer offsets and preserve relevant logs.
3. Confirm that required events remain within Kafka retention or an independent archive.
4. Stop or isolate the affected processing workflow if necessary.
5. Select a safe recovery method, such as restoring a known-good snapshot or replaying events into a separate recovery target.
6. Reset offsets only after confirming the intended consumer group and replay range.
7. Replay events with deduplication and idempotent database writes enabled.
8. Compare sales, stock, replenishment, and inventory-status totals against expected results.
9. Resume normal processing and verify consumer lag and dashboard freshness.
10. Document the cause, recovery actions, validation results, and follow-up work.

**Important:** This is a proposed runbook. It must be tested and adapted before operational use.

## 8. Lambda Architecture Checklist

- [ ] Define durable raw-event storage, such as Amazon S3.
- [ ] Design reconciliation between batch results and streaming results.
- [ ] Reuse consistent business logic across batch and streaming calculations.
- [ ] Define a merge strategy that prevents double counting.
- [ ] Test recovery after streaming failures.

## 9. Kappa Architecture Checklist

- [ ] Configure and verify seven-day Kafka retention.
- [ ] Estimate storage using measured event volume and size.
- [ ] Define event-time, late-event, and watermark policies.
- [ ] Create a tested replay and reprocessing runbook.
- [ ] Add event IDs and idempotent processing.
- [ ] Test processor restart and recovery.

## 10. Conclusion

The current project demonstrates the basic streaming path for inventory analytics. The improvements above are proposed reliability requirements, not claims that the current demo already implements them.
