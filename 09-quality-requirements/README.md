# 9. Quality Requirements

The most important quality characteristics of the Industrial Protocol Integration Solution are reliability, fault isolation, interoperability, extensibility, maintainability, scalability, and operability.

The initial deployment does not aim to provide full high availability. Instead, the architecture is designed to preserve local production communication and telemetry data during common temporary failures and to allow additional availability mechanisms to be introduced later if required.

## 9.1 Quality Tree

```text
Quality
│
├── Reliability
│   ├── Telemetry durability
│   ├── Recovery after communication failures
│   └── Workplace event recovery
│
├── Fault Isolation
│   ├── Area independence
│   └── Independence from central services
│
├── Interoperability & Extensibility
│   ├── Multiple industrial protocols
│   ├── New protocol adapters
│   └── Standardized workplace interface
│
├── Scalability
│   ├── Additional Areas
│   ├── Additional machines and workplaces
│   └── Controlled machine connection count
│
├── Maintainability
│   ├── Separation of responsibilities
│   └── Independent evolution of integration components
│
└── Operability
    ├── Health visibility
    ├── Failure diagnosis
    └── Central monitoring
```

## 9.2 Quality Scenarios

| ID | Quality Attribute | Scenario | Expected Response / Measure |
|---|---|---|---|
| QR-01 | Reliability | The Central Panel or Physical Server A becomes temporarily unavailable. | Local machine-to-workplace communication continues in every unaffected Area. Area Gateways continue publishing telemetry to RabbitMQ while it is available. Stored messages are processed after the Central Panel returns. |
| QR-02 | Reliability | RabbitMQ or the network connection between an Area Gateway and RabbitMQ becomes temporarily unavailable. | Telemetry already received by the Area Gateway remains in the local Telemetry Outbox and is published after connectivity is restored. |
| QR-03 | Reliability | The Gateway Application restarts after telemetry has been stored locally but before it has been published. | Pending Outbox messages survive the restart and are processed after the Gateway Application starts again. |
| QR-04 | Reliability | The Telemetry Worker receives a message but fails before successfully persisting it in the Central Panel Storage. | The message is not acknowledged as successfully processed and remains available for retry. Repeatedly failing messages can be isolated through the DLQ. |
| QR-05 | Reliability | A Workplace loses its SSE connection for a period shorter than the configured Event Journal retention period. | After reconnecting with its last processed event ID, available missed events are replayed in sequence before normal live delivery continues. |
| QR-06 | Fault Isolation | An Area Gateway in Area A fails. | Machine/workplace communication in Area A is affected, but Area B and other Areas continue operating independently. |
| QR-07 | Fault Isolation | Central telemetry or management infrastructure is unavailable. | Local communication between machines and workplaces does not depend on the Central Panel or RabbitMQ and therefore continues within each operational Area Gateway. |
| QR-08 | Interoperability | A new machine uses an already supported industrial protocol. | The machine can be integrated primarily through configuration without changes to workplace applications. |
| QR-09 | Extensibility | A machine using a new industrial protocol must be supported. | A new Protocol Adapter can be introduced without changing the standardized Workplace API or requiring protocol-specific logic in workplace applications. |
| QR-10 | Scalability | A new production building or Area is introduced. | A new Area Gateway can be deployed and configured without architectural changes to existing Areas. |
| QR-11 | Scalability | Additional workplaces start consuming information from an already integrated machine. | The number of direct industrial-protocol connections to the machine should not grow proportionally with the number of workplaces; machine connections are managed and shared by the Area Gateway where supported by the protocol. |
| QR-12 | Maintainability | Telemetry processing or reporting functionality changes. | Changes to central telemetry functionality should not require modifications to machine protocol adapters or existing local Workplace interfaces unless the underlying integration contract changes. |
| QR-13 | Maintainability | A machine-specific communication implementation must be changed. | The change should remain primarily isolated to the corresponding Protocol Adapter and related machine integration logic. |
| QR-14 | Operability | A machine, Area Gateway, or central processing component becomes unavailable or unhealthy. | Its health or communication state should be observable through the Central Panel and/or the customer's existing monitoring infrastructure. |
| QR-15 | Configuration Reliability | A newly published Area Gateway configuration is invalid. | The candidate configuration is rejected during validation and does not replace the currently applied working configuration. |

## 9.3 Performance and Timing

The system is intended for industrial integration and application-level machine communication, but it is not designed for hard real-time or deterministic control.

Local workplace communication is intentionally kept within the production Area to avoid unnecessary dependency on central infrastructure and to minimize additional network latency.

No strict end-to-end latency SLA is defined for the initial architecture. Concrete timing limits should be derived from the requirements of the machines and workplace applications during implementation and acceptance testing.

Functionality requiring deterministic or safety-critical response times must remain within the machine, PLC, or another dedicated real-time control system.

## 9.4 Availability

The initial architecture improves resilience but does not provide complete high availability.

In particular:

- each Area initially contains a single Area Gateway instance;
- RabbitMQ is initially deployed as a single broker instance;
- the Central Panel and Central Panel Storage reside on one physical Central Server.

The architecture limits the consequences of these failures through Area isolation, local persistence, the Telemetry Outbox, RabbitMQ persistence, and separation of RabbitMQ from the Central Server.

If stronger availability requirements are introduced, possible extensions include:

- redundant or active/standby Area Gateways;
- clustered RabbitMQ with replicated queues;
- redundant Central Panel instances;
- high-availability database deployment.

These mechanisms are considered evolution options rather than requirements of the initial deployment.

## 9.5 Security

The initial architecture assumes deployment inside the customer's controlled industrial and enterprise network.

The system should use authenticated and authorized access for Central Panel administration and protected application-level communication where required. Existing customer network-security, host-security, and infrastructure-monitoring mechanisms remain part of the surrounding environment.

A dedicated high-security or zero-trust architecture is not part of the current project scope.

## 9.6 Quality Boundaries

The architecture cannot guarantee recovery of information that was never successfully received by an Area Gateway.

For example, a machine event may be lost during a communication interruption between the machine and the Area Gateway. Recovery in such cases depends on capabilities of the corresponding machine protocol or server, such as historical-data support.

Similarly, temporary event replay for workplaces is limited by the configured retention period of the Recent Event Journal.

These limitations define the boundary of the reliability guarantees provided by the integration solution.
