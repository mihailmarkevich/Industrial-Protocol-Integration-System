# 8. Architectural Decisions

This section records the architectural decisions that have a significant impact on the structure, reliability, scalability, or evolution of the Industrial Protocol Integration Solution.

Detailed implementation choices that do not significantly affect the architecture are intentionally omitted.

## ADR-001 - Distribute Machine Integration by Production Area

**Status:** Accepted

### Context

A single centralized gateway for all machines would create a common failure point and would make local production communication dependent on central infrastructure.

### Decision

Each production area receives its own Area Gateway responsible for the machines and workplaces assigned to that area.

### Consequences

- Failure of one Area Gateway affects only its production area.
- Local machine communication remains close to the production environment.
- Additional areas can be introduced by deploying additional Area Gateways.
- Every Area Gateway requires its own deployment, configuration, and local storage.

## ADR-002 - Keep Workplace Communication Local

**Status:** Accepted

### Context

Workplace applications require direct access to machine information and commands during normal production operation.

Routing this communication through the Central Panel or RabbitMQ would introduce unnecessary latency and central dependencies.

### Decision

Workplaces communicate directly with their local Area Gateway using REST and SSE.

The Central Panel is not part of the runtime path between workplaces and machines.

### Consequences

- Local production remains operational when central services are unavailable.
- Communication latency stays local to the production area.
- Central telemetry infrastructure cannot become a dependency for normal workplace operation.
- The Area Gateway becomes a critical component within its individual area.

## ADR-003 - Isolate Industrial Protocols Behind Protocol Adapters

**Status:** Accepted

### Context

Existing machines expose heterogeneous interfaces such as OPC UA, OPC Classic, Modbus/TCP, and vendor-specific protocols.

Allowing workplace applications to communicate through these protocols directly would duplicate integration logic and tightly couple applications to machines.

### Decision

Industrial protocol communication is implemented through Protocol Adapters inside the Area Gateway.

Workplace applications use standardized application-level interfaces instead of machine-specific protocols.

### Consequences

- Industrial protocol details are isolated from consumers.
- New protocols can be introduced through additional adapters.
- Multiple workplaces can share gateway-managed machine connections.
- Protocol-specific capabilities may still require specialized adapter logic.

## ADR-004 - Use RabbitMQ for Asynchronous Central Telemetry

**Status:** Accepted

### Context

Telemetry must be collected centrally, but local gateways should not depend on the availability or processing speed of the Central Panel.

### Decision

Area Gateways publish telemetry asynchronously through RabbitMQ. A dedicated Telemetry Worker in the Central Panel consumes and persists the messages.

### Consequences

- Telemetry producers and consumers are decoupled.
- Central processing can temporarily stop while RabbitMQ retains pending messages.
- Retry, acknowledgement, and dead-letter mechanisms can be applied centrally.
- RabbitMQ becomes an additional infrastructure component that must be operated and monitored.

## ADR-010 - Use AMQP as the Common Messaging Protocol

**Status:** Accepted

### Context

The solution requires reliable asynchronous communication between Area Gateways, RabbitMQ, and the Central Panel.

Using different messaging protocols for different parts of the telemetry path, for example MQTT on the Area Gateway side and AMQP on the Central Panel side, would increase implementation and operational complexity without providing sufficient benefit for the current requirements.

### Decision

Use AMQP as the common messaging protocol for communication between Area Gateways, RabbitMQ, and the Central Panel telemetry consumer.

The same protocol is therefore used on both producer and consumer sides of the central telemetry pipeline.

### Consequences

- Only one messaging protocol needs to be implemented, configured, monitored, and maintained.
- Area Gateway and Central Panel messaging behavior remains consistent.
- Operational troubleshooting is simplified.
- The system avoids introducing an additional messaging technology without a concrete architectural need.
- If future machine or edge scenarios require another protocol such as MQTT, it can be introduced explicitly where justified.

## ADR-005 - Persist Telemetry in a Local Outbox Before Publication

**Status:** Accepted

### Context

An Area Gateway may lose connectivity to RabbitMQ or restart after receiving a machine event but before successfully publishing it.

### Decision

Telemetry is persisted to the local SQLite Telemetry Outbox before asynchronous publication begins.

The Outbox entry is removed only after RabbitMQ acknowledges successful receipt.

### Consequences

- Temporary broker and network outages do not immediately result in telemetry loss.
- Pending telemetry survives Gateway Application restarts.
- RabbitMQ and the Outbox protect different parts of the delivery chain.
- Local storage capacity must be monitored if central connectivity is unavailable for an extended period.

## ADR-006 - Provide Short-Term Workplace Event Replay

**Status:** Accepted

### Context

SSE connections between workplaces and Area Gateways may be temporarily interrupted. Without recovery support, events generated during the interruption would be missed by the Workplace.

### Decision

Replayable machine events are temporarily stored in a Recent Event Journal with ordered sequence identifiers.

After reconnecting, a Workplace provides its last successfully processed event ID and the Area Gateway replays the available missing events before continuing the live stream.

### Consequences

- Short communication interruptions can be recovered without querying machines again.
- Workplace applications can maintain event continuity across SSE reconnects.
- The journal is intentionally temporary and does not provide permanent event history.
- Recovery is possible only while the requested events remain within the configured retention period.

## ADR-007 - Centralize Configuration, but Keep the Applied Configuration Locally

**Status:** Accepted

### Context

Configuration of Areas, machines, and Area Gateways must be centrally manageable, while an Area Gateway should not stop operating if the Central Panel becomes temporarily unavailable.

### Decision

The Central Panel is the authoritative source of configuration.

Area Gateways retrieve published configuration through the Configuration API, validate it locally, and store the successfully applied version in their local SQLite storage.

A new configuration becomes operational only after successful local validation and activation.

### Consequences

- Configuration can be managed from one central location.
- Area Gateways retain their currently applied configuration during central outages.
- Invalid configuration does not immediately replace a working configuration.
- Configuration versions and their states must be tracked explicitly.

## ADR-008 - Separate RabbitMQ and the Central Panel into Different Physical Failure Domains

**Status:** Accepted

### Context

Telemetry is considered important. If RabbitMQ, the Telemetry Worker, and the Central Database were hosted on the same physical server, failure of that server would make the complete central telemetry path unavailable at once.

### Decision

RabbitMQ is deployed on Physical Server B, while the Central Panel and Central Panel Storage are deployed on Physical Server A.

### Consequences

- Failure of the Central Server does not immediately prevent Area Gateways from publishing telemetry.
- RabbitMQ can retain telemetry until the Telemetry Worker becomes available again.
- Gateway Outboxes continue protecting telemetry when RabbitMQ itself or the network toward it is unavailable.
- RabbitMQ remains a single infrastructure failure point in the initial deployment; higher availability can later be introduced through clustered broker nodes if required.

## ADR-009 - Keep Integration Data Separate from the Existing Customer Database

**Status:** Accepted

### Context

The customer's existing business database already belongs to existing applications and must remain unaffected by the integration project.

The new solution also requires dedicated persistence for telemetry, configuration, and gateway recovery mechanisms.

### Decision

The integration solution owns its own databases:

- one SQLite Gateway Storage per Area Gateway;
- one Central Panel Storage (Microsoft SQL Server) for centralized telemetry, configuration, and integration metadata.

The existing customer business database remains outside the architectural ownership of the solution.

### Consequences

- Existing database schemas and applications remain unaffected.
- Integration data has clearly defined ownership.
- Area Gateways can persist local recovery information independently.
- Synchronization between the integration system and customer business data must use explicit interfaces if such integration is required in the future.
