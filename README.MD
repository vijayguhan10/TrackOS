---
# Product Requirements Document (PRD)

## Real-Time Rail Slot Allocation & Runtime Observability Engine
---

## 1. Product Overview

### Product Name

TrackOS

### One-line Description

TrackOS is a real-time rail slot allocation and conflict resolution engine that dynamically schedules train movements under strict latency and safety constraints, while deeply analyzing JVM runtime behavior, OS scheduling, and memory pressure to explain and predict SLA violations.

---

## 2. Problem Statement

Modern rail and metro systems operate under tight time windows and shared infrastructure (tracks, platforms, signaling zones). When delays or disruptions occur, existing systems struggle to:

- Reallocate track slots in real time
- Resolve conflicts efficiently under load
- Explain why scheduling decisions slow down
- Predict cascading failures before SLAs are breached

Traditional systems rely heavily on:

- Database-centric logic
- Static schedules
- Surface-level monitoring (CPU %, memory %)

These approaches fail to capture runtime execution realities such as:

- JVM thread contention
- Garbage collection pauses
- OS-level scheduling delays
- Context switching overhead

---

## 3. Goals & Non-Goals

### Goals

- Perform real-time slot allocation with deterministic latency
- Detect and resolve track occupancy conflicts efficiently
- Guarantee strict SLA compliance (e.g., scheduling decision < 100ms)
- Provide deep runtime explainability:
  - JVM thread behavior
  - Memory allocation and GC pauses
  - OS scheduling delays
- Predict SLA violations before they occur

### Non-Goals

- Ticket booking or passenger-facing systems
- UI-heavy dashboards (visualization is secondary)
- Hardware-level signaling control
- Distributed multi-cluster orchestration (initial scope is single-region)

---

## 4. Target Users

- Rail Operations Control Teams
- Platform and Infrastructure Engineers
- SRE and Performance Engineers
- Transportation System Architects

---

## 5. Core Functional Requirements

### 5.1 Real-Time Slot Allocation Engine

- Allocate track slots based on time windows, track availability, and safety constraints
- Support live rescheduling during delays or disruptions
- Enforce deterministic execution latency

### 5.2 Conflict Detection (Data-Structure Driven)

- Use Interval Trees to represent track occupancy over time
- Detect overlapping time windows in O(log n) time
- Support concurrent reads and controlled writes

### 5.3 Priority-Based Conflict Resolution

- Use Priority Queues (Heaps) to decide conflict resolution order
- Priority factors may include passenger volume, delay severity, and train category (express, cargo, local)
- Ensure fairness while respecting business rules

### 5.4 Network Path Validation

- Model the rail network as a graph (nodes = stations, edges = tracks)
- Validate rerouting decisions dynamically
- Avoid blocked or congested segments

### 5.5 Capacity Tracking and Saturation Detection

- Use Segment Trees or time buckets to track load per time slice
- Detect near-saturation conditions
- Prevent unsafe over-allocation proactively

---

## 6. JVM Runtime Observability (Key Differentiator)

### 6.1 Thread-Level Analysis

- Track total JVM threads, runnable vs blocked vs waiting threads, thread pool saturation
- Correlate scheduling delays with business operations
- Monitor allocation rate, young/old generation pressure, GC pause durations

### 6.2 Garbage Collection and Memory Pressure

- Correlate GC safepoints with SLA breaches

### 6.3 Latency Decomposition

For each scheduling decision, break latency into:

- CPU execution time
- JVM thread wait time
- OS run-queue delay
- GC pause impact
- I/O wait (if applicable)

---

## 7. OS-Level Observability (eBPF-powered)

- Use eBPF to collect context switch frequency, CPU run-queue latency, and thread scheduling delays
- Attach dynamically to JVM process via PID
- Zero application code instrumentation

---

## 8. SLA Monitoring and Prediction

### 8.1 SLA Tracking

- Define SLAs such as slot allocation < 100ms and conflict resolution < 50ms
- Track violations in real time

### 8.2 SLA Violation Prediction

- Use runtime signals: thread queue length, context switching spikes, allocation rate surges
- Predict SLA risk windows (for example, next 2–5 minutes)

---

## 9. System Architecture

```
Train Event
	↓
Conflict Detector (Interval Tree)
	↓
Priority Resolver (Heap)
	↓
Path Validator (Graph)
	↓
Slot Allocation Commit
	↓
Latency and Runtime Analysis

```

```
Linux Kernel
	↓
eBPF Programs
	↓
Collector Agent (Go or Rust)
	↓
Analysis Engine (Java)
	↓
Metrics / Visualization

```

---

## 10. Technology Stack

- Java: Core runtime and scheduling engine
- Custom in-memory data structures: Interval Tree, Heap, Graph
- Linux eBPF (BCC or libbpf): OS-level telemetry
- JVM JFR or JVM TI: GC and safepoint insights
- Prometheus: Metrics collection
- Grafana: Visualization and SLA dashboards

---

## 11. Performance and Scalability Requirements

- Handle burst rescheduling events without SLA violation
- Maintain predictable latency under thread contention
- Graceful degradation under extreme load
- No unbounded memory growth

---

## 12. Failure Scenarios to Handle

- Sudden multi-train delays
- GC pause spikes during peak traffic
- Thread pool exhaustion
- CPU starvation
- OS-level scheduling contention

Each failure must be detected, explained, and correlated to business impact.

---

## 13. Success Metrics

- SLA compliance rate (>99.9%)
- Mean scheduling latency
- Predictive accuracy of SLA violation alerts
- Reduction in unexplained delays

---

## 14. Why TrackOS is Different

- Data-structure-driven core, not CRUD
- JVM and OS observability baked into product logic
- Business decisions directly linked to runtime behavior
- Designed like a real execution runtime, not an API service

---

## 15. Future Enhancements

- Multi-region scheduling
- Adaptive thread pool tuning
- ML-assisted priority weighting
- Kubernetes-aware scheduling insights

---

### Final Positioning Statement

TrackOS is not a rail application — it is a real-time execution runtime that proves how JVM internals and OS scheduling directly impact mission-critical business SLAs.

---

If you want next, I can:

- Convert this into a Notion-ready template
- Create architecture diagrams
- Write resume bullets
- Prepare a 10-minute interview walkthrough based on this PRD
