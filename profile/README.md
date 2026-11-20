<picture>
  <source media="(prefers-color-scheme: dark)"  srcset="https://raw.githubusercontent.com/stacksaga/.github/main/profile/assets/readme-banner-dark.png">
  <source media="(prefers-color-scheme: light)" srcset="https://raw.githubusercontent.com/stacksaga/.github/main/profile/assets/readme-banner-light.png">
  <img alt="StackSaga — The Next Evolution of Saga Orchestration" src="https://raw.githubusercontent.com/stacksaga/.github/main/profile/assets/readme-banner-light.png" width="100%">
</picture>

<br/>

<div align="center">

[![License: SSAL-1.0](https://img.shields.io/badge/License-SSAL--1.0-28B76B.svg)](https://stacksaga.org/legal/license/ssal-1.0)
[![Spring Boot](https://img.shields.io/badge/Spring%20Boot-4.x-6DB33F?logo=springboot&logoColor=white)](https://spring.io/projects/spring-boot)
[![Java](https://img.shields.io/badge/Java-21%2B-ED8B00?logo=openjdk&logoColor=white)](https://openjdk.org/)
[![Project Reactor](https://img.shields.io/badge/Project%20Reactor-Natively%20Reactive-28B76B)](https://projectreactor.io/)
[![Docs](https://img.shields.io/badge/Docs-docs.stacksaga.org-28B76B)](https://docs.stacksaga.org)
[![Discussions](https://img.shields.io/badge/GitHub-Discussions-181717?logo=github)](https://github.com/stacksaga/stacksaga-community/discussions)

</div>

---

## The Next Evolution of Saga Orchestration

**StackSaga** is the **domain-centric, natively reactive** saga orchestration ecosystem for Spring Boot.

Build resilient, high-throughput distributed microservices with **zero dedicated orchestrator servers**, **zero Transactional Outbox**, and automated reverse compensation — in a developer-friendly, compile-time type-safe environment.

> _"One framework. From Modular Monolith to full microservices — with zero saga rewrites."_

---

## Why StackSaga?

| | StackSaga | Temporal / Camunda | Kafka Choreography |
|---|---|---|---|
| **Dedicated Servers** | ✅ **Zero** — embedded in your service | ❌ Heavy external cluster | ⚠️ Broker clusters only |
| **Execution Model** | ✅ Natively Reactive (Project Reactor) | ❌ Thread-pool long-polling | ⚠️ Consumer thread pools |
| **Mixed STP + HITL** | ✅ Native in one pipeline | ❌ Requires external orchestration | ❌ Fragile ad-hoc state machines |
| **Type Safety** | ✅ 100% compile-time verified | ⚠️ Partially typed | ❌ Untyped JSON + string topics |
| **Transactional Outbox** | ✅ **Eliminated** — atomic Event Store | ❌ Required for every trigger | ❌ Required before every publish |
| **Monolith → Microservices** | ✅ Zero-rewrite extraction | ❌ Microservices-only | ❌ Forces broker between modules |
| **Downstream Coupling** | ✅ Zero — pure REST / gRPC APIs | ❌ SDK installation required | ❌ Topic binding required |

---

## Core Capabilities

### ⚡ Natively Reactive & Non-Blocking
Built on **Project Reactor** from the ground up. The entire Saga Execution Coordinator (SEC) is non-blocking — thousands of concurrent sagas run on minimal CPU threads with zero thread exhaustion.

### 🔀 Unified Saga Model (USM) — STP + HITL in One Pipeline
The first framework to execute **mixed-style sagas** in the same pipeline:
- **Continuous STP** — sub-millisecond automated processing with zero pauses
- **Pausable HITL** — days-long human approval wait-states that release 100% of worker threads, with decoupled `ResumeContext` callback ingestion

### 🛡️ Spaghetti-Less, Compile-Time Type-Safe Architecture
- Strongly-typed `CommandExecutor<T>` and `QueryExecutor<T>` generics
- Compiler-verified step transitions via `.toNextExecutor(NextStep.class)` — no broken string references
- IDE autocomplete and refactor-safe pipeline definition

### 🏗️ Domain-Centric — Monolith to Microservices, Zero Rewrites
Run as an in-process Modular Monolith at `<1ms` speeds today. When you extract a module into a standalone microservice, your saga pipelines, compensations, and event log stay **100% intact** — no rewrites, no migration.

### 🔄 1-to-Many Multi-Stage Compensations
Real enterprise rollbacks need more than a single undo. Chain `Pre-RevertExecutor` → `doRevert()` → `Post-RevertExecutor` for fraud holds, audit logs, and customer notifications — without polluting primary business logic.

### 🗄️ Zero-Infrastructure Event Sourcing
Your existing **PostgreSQL, MySQL, Oracle, or Cassandra** database becomes the immutable event store. Event Store writes commit **atomically within the same ACID transaction** as your domain logic — the Transactional Outbox pattern is completely eliminated.

### 🔁 Lock-Free Distributed Retries
**Murmur3 token-ring partitioning** assigns conflict-free retry ownership across cluster nodes — no locks, no single point of failure, automatic self-healing rebalancing on scale-out.

---

## A Taste of StackSaga

```java
// Define the saga pipeline — 100% type-safe, compiler-verified
@SagaExecutor(executeFor = "order-service", value = "OrderFulfillmentSaga")
public class OrderFulfillmentSaga implements UniSagaExecutorTemplate<OrderProcess> {

    @Override
    public ProcessStepManager<OrderProcess> registerSteps(
            ProcessStepManagerPool<OrderProcess> stepManagerPool) {

        return stepManagerPool
            .next(CheckCustomerEligibilityExecutor.class)   // STP: in-process, <1ms
            .next(Charge3DSecureExecutor.class)             // HITL: pauses, releases all threads
            .next(ReserveInventoryExecutor.class)           // STP: resumes after webhook callback
            .next(DispatchWarehouseExecutor.class)          // Two-Stage: atomic commit barrier
            .build();
    }
}
```

```java
// Each executor: exactly one atomic transaction boundary
@SagaExecutor(executeFor = "order-service", value = "ReserveInventoryExecutor")
public class ReserveInventoryExecutor implements CommandExecutor<OrderProcess> {

    @Override
    public ProcessStepManager<OrderProcess> doProcess(
            OrderProcess process,
            ProcessStepManager<OrderProcess> stepManager) {

        inventoryService.reserve(process.getOrderId(), process.getItems());
        return stepManager.next(DispatchWarehouseExecutor.class);
    }

    @Override
    public ProcessStepManager<OrderProcess> doRevert(
            OrderProcess process,
            ProcessStepManager<OrderProcess> stepManager) {

        inventoryService.release(process.getOrderId());   // automatic LIFO rollback
        return stepManager.compensated();
    }
}
```

---

## The Ecosystem

<table>
<tr>
<td width="50%">

### 🔧 Unified Orchestrator Engine (SEC)
`stacksaga-spring-boot-starter`

Domain-embedded execution coordinator. Drives Continuous STP, Pausable HITL, Two-Stage messaging, and deterministic reverse compensation — all in one framework.

</td>
<td width="50%">

### 🗃️ Zero-Infrastructure Event Store
`stacksaga-database-support`

Turns your existing database (MySQL, PostgreSQL, Oracle, Cassandra, ScyllaDB) into an immutable saga event store. No extra clusters. No background pollers.

</td>
</tr>
<tr>
<td width="50%">

### 🔁 Ring Coordinator
`stacksaga-ring-coordinator`

Cluster coordination via Murmur3 token-ring partitioning. Conflict-free, lock-free retry ownership assignment across nodes. Zero SPOF.

</td>
<td width="50%">

### 📊 Trace Window
`stacksaga-trace-window`

Real-time visualization of saga executions, step states, and failure diagnostics. Data stays inside your VPC. Manual replay for incident recovery.

</td>
</tr>
</table>

**Start with SEC + Event Store. Add Ring Coordinator for distributed retries. Add Trace Window for full observability.**

---

## Get Started in Minutes

**Step 1 — Generate your project config with [StackSaga Initializer](https://init.stacksaga.org)**

Select your Spring Boot version, build tool, orchestrator type, and database — get the exact BOM and dependencies to paste into your project.

**Step 2 — Add to your Spring Boot service**

```xml
<!-- Maven — from StackSaga Initializer output -->
<dependencyManagement>
  <dependencies>
    <dependency>
      <groupId>org.stacksaga</groupId>
      <artifactId>stacksaga-bom</artifactId>
      <version>LATEST</version>
      <type>pom</type>
      <scope>import</scope>
    </dependency>
  </dependencies>
</dependencyManagement>

<dependencies>
  <dependency>
    <groupId>org.stacksaga</groupId>
    <artifactId>stacksaga-spring-boot-starter</artifactId>
  </dependency>
  <dependency>
    <groupId>org.stacksaga</groupId>
    <artifactId>stacksaga-mysql-support</artifactId>  <!-- or postgresql / oracle / cassandra -->
  </dependency>
</dependencies>
```

**Step 3 — Define your first saga and start orchestrating**

---

## Resources

<div align="center">

| Resource | Link |
|---|---|
| 📖 **Documentation** | [docs.stacksaga.org](https://docs.stacksaga.org/stacksaga/latest/introduction-to-stacksaga.html) |
| 🚀 **Project Initializer** | [init.stacksaga.org](https://init.stacksaga.org) |
| 📐 **USM Whitepaper & Specification** | [Unified Saga Model Spec](https://docs.stacksaga.org/stacksaga/latest/implementations/stacksaga-sync/orchestrator/unified-saga-patterns.html) |
| 🔭 **Trace Window Cloud** | [trace.stacksaga.org](https://trace.stacksaga.org) |
| 💬 **Discussions & FAQs** | [GitHub Discussions](https://github.com/stacksaga/stacksaga-community/discussions) |
| 🌐 **Website** | [stacksaga.org](https://stacksaga.org) |
| 📜 **License (SSAL 1.0)** | [View License](https://stacksaga.org/legal/license/ssal-1.0) |

</div>

---

<div align="center">

Built with ❤️ by **[Zoftstack](https://zoftstack.com)**

*StackSaga and the StackSaga logo are trademarks of Zoftstack.*

</div>
