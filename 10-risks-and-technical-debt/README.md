# 10. Risks & Technical Debt

This section summarizes the most important known architectural risks and intentionally deferred technical improvements.

Risks are ordered approximately by their potential impact on production operation and telemetry reliability.

## 10.1 Architectural Risks

| Priority | Risk | Impact | Mitigation / Current Protection |
|---|---|---|---|
| High | Single Area Gateway per production area | Failure of an Area Gateway interrupts machine integration and workplace communication within the affected Area. | Failures are isolated to one Area. A future deployment can introduce an active/standby Gateway if higher availability is required. |
| High | Single RabbitMQ instance | Failure of the RabbitMQ server temporarily stops central telemetry ingestion from all Areas. | Each Area Gateway retains unpublished telemetry in its local Outbox until RabbitMQ becomes available again. A RabbitMQ cluster can be introduced if higher availability becomes necessary. |
| High | Central Server is a single failure point for central functionality | Failure of the server hosting the Central Panel and Central Panel Storage makes telemetry querying, configuration management, dashboards, and telemetry persistence temporarily unavailable. | RabbitMQ is deployed on a separate physical server and continues buffering telemetry until the Central Server returns. Local Area communication remains unaffected. |
| High | Machine events can be lost before reaching the Area Gateway | A network interruption between a machine and its Area Gateway can result in missing events if the machine does not provide historical/replay capabilities. | Where supported, protocol adapters can recover data from machine-side history, for example OPC UA Historical Access. This cannot be guaranteed for all machines and protocols. |
| Medium | Local Outbox can grow during long central outages | If RabbitMQ remains unavailable for a long period, the Gateway SQLite database can consume significant disk space and eventually exhaust local storage. | Monitor Outbox size and available disk capacity. Define operational limits and alerting according to expected telemetry volume. |
| Medium | At-least-once telemetry delivery can produce duplicates | Failure between central persistence and message acknowledgement can cause RabbitMQ to redeliver an already processed event. | Telemetry messages require stable identifiers and central processing should be idempotent or otherwise capable of recognizing duplicate events. |
| Medium | Recent Event Journal has limited retention | A Workplace disconnected longer than the retention period may no longer be able to recover every missed SSE event. | Retention is intentionally limited because the journal provides short-term recovery rather than permanent event history. Longer retention can be configured if required. |
| Medium | Telemetry volume may exceed initial assumptions | A large increase in machines, event frequency, or number of Areas can increase RabbitMQ backlog, SQL storage requirements, and Telemetry Worker processing load. | Monitor throughput and backlog. The Telemetry Worker can later be scaled or messaging/storage architecture reevaluated if actual load requires it. |
| Medium | Area Gateway can become a local capacity bottleneck | Large numbers of machines, subscriptions, workplaces, or high-frequency signals may exceed the capacity of one Gateway instance. | Area-based distribution already limits the scope of each Gateway. Large Areas can be partitioned between additional Gateway instances, with every machine assigned to exactly one active Gateway. |
| Medium | Conventional load balancing is unsuitable for Area Gateways | Area Gateways maintain long-lived, stateful connections to industrial machines and own local runtime and persistent state. Running multiple active Gateway instances for the same machines behind a conventional load balancer could cause duplicate subscriptions and telemetry, competing machine commands, additional machine sessions, and inconsistent state. | Do not scale Area Gateways through generic active-active load balancing. For additional capacity, partition machines between Gateway instances. For higher availability, prefer an active/standby deployment with controlled failover and explicit machine ownership. |
| Medium | Industrial protocol and vendor differences | Some machines may expose capabilities, error behavior, connection limits, or data semantics that cannot be represented uniformly. | Protocol-specific behavior is isolated behind adapters. Machine-specific extensions may still be required where a normalized model is insufficient. |
| Low / Scope Boundary | Hard real-time communication is not guaranteed | Very low-latency or deterministic machine-control requirements may not be achievable through the Area Gateway and application-level protocols. | Hard real-time and safety-critical control remains outside the integration system and should execute on PLCs, machines, or dedicated real-time components. |

## 10.2 Technical Debt and Deferred Work

The following items are known gaps or intentionally deferred improvements rather than flaws that must be solved for the initial deployment.

| Technical Debt | Description | Suggested Resolution |
|---|---|---|
| DLQ remediation process is not defined | Messages that exceed the RabbitMQ retry limit are moved to the Dead-Letter Queue, but the operational process for inspection, correction, replay, or deletion is not yet defined. | Define an operational DLQ workflow together with the customer and operations team. |
| No Area Gateway high availability | Every production Area initially relies on one Gateway instance. | Introduce active/standby Gateway deployment if Area availability requirements justify the additional complexity. |
| Area Gateway failover strategy is not defined | Conventional load balancing cannot safely provide Area Gateway redundancy because Gateway instances maintain stateful machine connections and local persistent state. | If higher Area availability becomes necessary, design an active/standby failover mechanism with explicit machine ownership and state recovery instead of generic active-active load balancing. |
| RabbitMQ is not clustered | RabbitMQ is separated from the Central Server but still runs as a single broker instance. | Introduce a multi-node RabbitMQ cluster with replicated/quorum queues if central telemetry availability requirements increase. |
| Central Panel has no redundant deployment | APIs, UI Dashboard, Telemetry Worker, and Central Panel Storage rely on one Central Server in the initial deployment. | Introduce redundant application instances and database HA when required by availability targets. |
| Operational alerting thresholds are not yet defined | The architecture exposes health information, but concrete thresholds for Outbox growth, queue backlog, disconnected machines, database capacity, and failed messages are not yet specified. | Define monitoring and alerting rules together with the customer's operational environment. |
| Telemetry capacity limits have not yet been validated with production load | Actual event frequency, message size, retention period, and number of future Area Gateways may differ from initial assumptions. | Perform load and endurance testing with representative production telemetry and derive concrete capacity limits. |
| Configuration rollback procedure requires further definition | Configuration versions are stored and activation is validated, but the exact operational procedure for automatic or manual rollback after post-activation problems is not yet fully specified. | Define rollback rules and whether previous configuration versions can be reactivated automatically or only by an administrator. |

## 10.3 Risk Summary

The most significant limitation of the initial deployment is that it provides data durability and fault isolation, but not complete high availability.

The architecture already reduces the impact of failures through:

- independent Area Gateways;
- local Telemetry Outboxes;
- short-term Workplace Event Journals;
- persistent RabbitMQ messaging;
- separation of RabbitMQ and the Central Panel across different physical servers;
- locally stored Applied Configuration.

Area Gateway scalability must also respect the stateful nature of machine communication. Additional capacity should therefore be achieved primarily by partitioning machines between Gateway instances, while higher availability should use a controlled active/standby failover model rather than conventional active-active load balancing.

However, individual Area Gateways, RabbitMQ, and the Central Server remain single failure points within their respective scopes.

These limitations are accepted for the initial deployment and can be addressed incrementally if future quality requirements justify the additional infrastructure and operational complexity.
