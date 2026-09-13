# 5. Building Block View

The Building Block View describes the internal structure of the two main software systems of the Industrial Protocol Integration Solution:

1. Area Gateway
2. Central Panel

## 5.1 Area Gateway (Whitebox)

### 5.1.1 Container View

[PASTE Area Gateway Container Diagram]

The Area Gateway consists of two containers.

| Container | Technology | Responsibility |
|---|---|---|
| Gateway Application | .NET Application | Communicates with industrial machines, maintains machine connections, exposes the Workplace API, creates telemetry, and exchanges information with central services. |
| Gateway Storage | SQLite | Provides local persistence for the Telemetry Outbox, Recent Event Journal, and Applied Configuration. |

### Rationale

Machine communication and application-level integration are concentrated in one Gateway Application so that workplace applications do not need to maintain their own industrial-protocol implementations or machine connections.

Local SQLite storage provides the Area Gateway with the persistence required for telemetry delivery, workplace recovery, and configuration retention without introducing an additional database server within every production area.

The Gateway Application communicates with:

- Industrial Machines through their existing industrial protocols;
- Workplace Applications through REST and SSE;
- RabbitMQ for centralized telemetry delivery;
- Central Panel for central processing and configuration.

### 5.1.2 Gateway Application - Component View

[PASTE Area Gateway Component Diagram]

The Gateway Application is decomposed into the following architecturally relevant components.

| Component | Responsibility |
|---|---|
| Protocol Adapters | Implement communication with the supported industrial protocols and provide access to machine data and operations. |
| Worker (Core) | Coordinates the main runtime behavior of the gateway and the communication between machine integration and workplace functionality. |
| API | Provides REST operations and SSE streams to workplace applications. |
| Telemetry Service | Receives machine telemetry, persists it temporarily in the Telemetry Outbox, and forwards it through RabbitMQ. |
| Configuration Service | Receives configuration provided by the Central Panel, stores the current configuration locally, and provides it to the gateway runtime. |

#### Protocol Adapters

Protocol Adapters encapsulate communication with industrial machines using protocols such as OPC UA, Modbus, and OPC Classic.

They isolate protocol-specific communication from the remaining gateway functionality.

#### Worker (Core)

The Worker represents the central runtime logic of the Gateway Application.

It coordinates machine communication through the Protocol Adapters and provides the functionality required by the Workplace API. It operates according to the configuration supplied by the Configuration Service.

#### API

The API is the communication boundary between the Area Gateway and existing workplace applications.

It provides:

- REST for request/response operations;
- SSE for continuous machine-data delivery.

Recent machine events are temporarily stored in the Recent Event Journal to support recovery of workplace communication.

#### Telemetry Service

The Telemetry Service handles the central telemetry path.

Telemetry is stored in the local Telemetry Outbox as early as possible and then forwarded to RabbitMQ. Outbox entries remain available until successful acknowledgement by RabbitMQ.

This separates telemetry creation from the availability and processing speed of the Central Panel.

#### Configuration Service

The Configuration Service handles configuration supplied by the Central Panel.

Received configuration is stored as the current Applied Configuration in Gateway Storage and is made available to the gateway runtime.

This allows the Area Gateway to retain its currently valid configuration locally.

### 5.1.3 Gateway Storage

Each Area Gateway has one local SQLite database.

The Component Diagram shows its three logical responsibilities separately to make their architectural purposes explicit.

| Logical Storage | Purpose |
|---|---|
| Telemetry Outbox | Temporarily stores telemetry until successful acknowledgement by RabbitMQ. |
| Recent Event Journal | Temporarily stores recent machine events required for workplace recovery. |
| Applied Configuration | Stores the current configuration used by the Area Gateway. |

All three logical storage areas belong to the same Gateway Storage / SQLite container shown in the Container Diagram.

## 5.2 Central Panel (Whitebox)

The Central Panel provides the centralized part of the Industrial Protocol Integration Solution.

It receives and permanently stores telemetry from Area Gateways, provides centralized configuration management, exposes telemetry to other applications, and offers a common dashboard for monitoring and administration.

### 5.2.1 Container View

[PASTE Central Panel Container Diagram]

The Central Panel consists of the following containers.

| Container | Technology | Responsibility |
|---|---|---|
| Telemetry Worker | Worker Service | Consumes telemetry delivered through RabbitMQ, processes and normalizes incoming data, and persists it in the Central Panel Storage. |
| Central Panel Storage | Microsoft SQL Server | Provides persistent central storage for telemetry, configuration, and integration-related metadata. |
| Telemetry API | API Application | Provides access to persisted telemetry and related information for the UI Dashboard and external applications. |
| Configuration API | API Application | Manages configuration of machines, Areas, and Area Gateways and persists this configuration centrally. |
| UI Dashboard | Web Application | Provides the main user interface for telemetry visualization, business analytics, system-health monitoring, and integration configuration. |

### Rationale

Telemetry processing is separated from interactive API and UI functionality through a dedicated Telemetry Worker. This allows incoming telemetry to be processed independently of dashboard usage or requests from other applications.

The Central Panel Storage acts as the authoritative persistent data store for the centralized part of the solution. Applications consume historical and current centralized information from this database instead of communicating directly with industrial machines.

Telemetry and configuration functionality are exposed through separate APIs because they serve different responsibilities and can evolve independently.

### 5.2.2 Telemetry Worker

The Telemetry Worker is responsible for the central telemetry ingestion path.

Telemetry produced by Area Gateways is delivered through RabbitMQ. The Telemetry Worker consumes these messages, processes and normalizes their content, and persists the resulting information in the Central Panel Storage.

The worker operates independently from the Central Panel UI and APIs.

### 5.2.3 Central Panel Storage

The Central Panel uses a dedicated Microsoft SQL Server database for persistent central data.

The database stores:

- normalized telemetry and machine history;
- machine and integration configuration;
- Area and Area Gateway configuration;
- metadata required for centralized monitoring and management.

This database belongs exclusively to the Central Panel and is separate from the customer's existing business database.

### 5.2.4 Telemetry API

The Telemetry API provides application-level access to telemetry stored by the Central Panel.

It is used by:

- the Central Panel UI Dashboard;
- reporting and visualization applications;
- analytics and BI applications;
- other applications requiring centralized machine or telemetry information.

These consumers can therefore access persisted machine information without establishing direct connections to industrial machines or Area Gateways.

### 5.2.5 Configuration API

The Configuration API provides centralized management of integration configuration.

It manages configuration related to machines, Areas, and Area Gateways and stores the resulting configuration in the Central Panel Storage.

The API is used by the UI Dashboard for administrative configuration operations and provides the centralized source of configuration required by Area Gateways.

### 5.2.6 UI Dashboard

The UI Dashboard is the main user-facing application of the Central Panel.

It uses the Telemetry API and Configuration API to provide a centralized view of the integration environment.

The dashboard supports:

- telemetry visualization;
- business analytics and reporting;
- machine and Area Gateway status;
- system-health monitoring;
- configuration of machines, Areas, and Area Gateways.

The dashboard does not communicate directly with industrial machines. Machine communication remains the responsibility of the corresponding Area Gateways.
