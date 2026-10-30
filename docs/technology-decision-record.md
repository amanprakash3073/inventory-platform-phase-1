# Technology Decision Record (TDR)

## 1. Purpose

This document records the technology choices for the Inventory Platform, explains why each technology is suitable, and identifies important trade-offs.

## 2. Python — Event Generation and Processing

**Decision:** Use Python for the event generator and stream processor.

**Reasoning:**
- Supports rapid development and readable data-processing logic.
- Provides Kafka client libraries and PostgreSQL connectivity.
- Makes it practical to implement inventory calculations and event handling.

**Trade-offs and limitations:**
- Runtime performance may be lower than compiled languages for some workloads.
- Processing throughput and memory usage must be measured before production deployment.

## 3. Apache Kafka — Event Streaming

**Decision:** Use Apache Kafka with separate topics for products, sales, stock updates, and replenishments.

**Reasoning:**
- Separates event producers from consumers.
- Supports event retention and replay within configured retention limits.
- Allows additional consumers to be introduced as requirements grow.

**Trade-offs and limitations:**
- Requires monitoring, capacity planning, and careful consumer-offset management.
- The current demo uses a single Kafka broker and one partition per topic, which limits fault tolerance and parallelism.
- Seven-day retention must be configured and validated before it can be treated as a requirement met.

## 4. PostgreSQL — Relational Storage

**Decision:** Use PostgreSQL to store product data, sales, replenishments, stock history, and inventory status.

**Reasoning:**
- Supports relational tables, SQL queries, indexes, and numeric data types.
- Provides a convenient source for dashboard queries.
- Supports persistent storage through a Docker volume.

**Trade-offs and limitations:**
- Write throughput and dashboard-query performance require testing at higher event volumes.
- Database backup, recovery, and data-retention policies must be defined for production use.

## 5. Grafana — Visualization

**Decision:** Use Grafana for inventory analytics and operational dashboards.

**Reasoning:**
- Presents metrics and trends in visual panels.
- Connects to PostgreSQL as a data source.
- Supports dashboard provisioning for repeatable local setup.

**Trade-offs and limitations:**
- Dashboard accuracy depends on the quality and freshness of the stored data.
- Access control and secure credentials must be configured for deployment beyond a local demo.

## 6. Docker Compose — Local Deployment

**Decision:** Use Docker Compose to run the Kafka broker, PostgreSQL, event generator, processor, and Grafana services.

**Reasoning:**
- Provides a repeatable local development environment.
- Defines service dependencies and persistent volumes.
- Simplifies starting and stopping the demo.

**Trade-offs and limitations:**
- A single-machine Compose deployment is not a highly available production architecture.
- Service health checks do not replace application-level monitoring and recovery testing.

## 7. Architecture Direction

The current implementation is primarily a streaming pipeline consistent with a Kappa-style direction. It does not currently provide a complete Lambda architecture with a separate batch layer and reconciliation process.

A future design should evaluate durable raw-event storage, event deduplication, late-event handling, replay procedures, and recovery after processor restarts.

## 8. Summary

The selected stack is suitable for demonstrating a local real-time inventory analytics pipeline. Production readiness would require further work on availability, security, observability, recovery, and data correctness.
