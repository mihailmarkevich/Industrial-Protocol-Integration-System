# 3. Context & Scope

This section describes the environment of the Industrial Protocol Integration Solution and its interfaces with existing production systems and users.

The solution consists of two major software systems:

- **Area Gateway** - provides standardized access to industrial machines for local workplace applications and forwards relevant information to central services.
- **Central Panel** - provides centralized telemetry, reporting, visualization, configuration, and system monitoring.

Both systems belong to the Industrial Protocol Integration Solution. Their internal architecture is intentionally not shown in this section and is described later in the Building Block View.

Existing machines, workplace applications, reporting applications, and users remain outside the solution boundary.

## 3.1 Business Context

The Business Context describes who interacts with the Industrial Protocol Integration Solution and for what purpose, independently of concrete communication technologies.

[PASTE Business Context Diagram]

| Communication Partner | Interaction with the Solution | Purpose |
|---|---|---|
| Industrial Machines | Exchange machine data and supported machine operations with the Area Gateway | Provide production values, status information and machine functionality required by workplace applications |
| Workplace Applications | Access machine data and execute supported machine operations through the Area Gateway | Allow existing production workplaces to interact with machines without implementing machine-specific communication independently |
| Worker | Uses an existing Workplace Application | Operates the workplace as part of the production process |
| Reporting / Visualization Applications | Consume centralized telemetry through the Central Panel | Display or analyse production and telemetry information |
| Administrator | Uses the Central Panel | Configures integrations and monitors telemetry and overall integration health |

The Area Gateway and Central Panel have different responsibilities but exchange information as part of the same integration solution.

The Area Gateway provides the Central Panel with:

- telemetry data;
- integration and communication status;
- health information.

The Central Panel provides centralized configuration information required by the integration environment.

Local workplace-to-machine communication is handled by the Area Gateway and is therefore separated from centralized telemetry and management functionality.

## 3.2 Technical Context

The Technical Context describes how the communication relationships shown in the Business Context are technically realized.

It focuses on system boundaries and communication technologies. Internal implementation details such as databases, internal services, protocol adapter structure, or message-processing components are described in later sections.

[PASTE Technical Context Diagram]

| Communication | Technical Interface | Description |
|---|---|---|
| Industrial Machines ↔ Area Gateway | OPC UA, OPC Classic, Modbus/TCP, vendor-specific TCP-based protocols | The Area Gateway communicates with existing machines using the protocols already provided by the industrial equipment |
| Workplace Applications ↔ Area Gateway | HTTPS: REST API + Server-Sent Events (SSE) | REST is used for request/response operations and supported machine commands; SSE provides continuous machine-data updates to workplace applications |
| Area Gateway → Central Panel | RabbitMQ over AMQP | Telemetry and integration-related information are transferred asynchronously from Area Gateways toward central processing |
| Reporting / Visualization Applications → Central Panel | REST API over HTTPS | External applications consume centralized telemetry and reporting data through the Central Panel API |
| Administrator → Central Panel | Web UI over HTTPS | Administrators access configuration, telemetry, and system-health functionality through the Central Panel web interface |

## 3.3 System Boundary

The Industrial Protocol Integration Solution is responsible for the integration between existing industrial equipment, workplace applications, and centralized telemetry and management functionality.

The solution includes:

- machine communication and protocol integration;
- standardized access to machine data and supported operations;
- distribution of machine information to workplace applications;
- centralized telemetry collection;
- integration configuration;
- integration-health monitoring;
- interfaces for reporting and visualization applications.

The following elements remain outside the solution boundary:

- industrial machine hardware and internal machine control logic;
- PLC and safety-control logic;
- existing workplace business logic;
- existing reporting and visualization applications;
- physical network infrastructure;
- customer-specific infrastructure and business systems not directly owned by the integration solution.

The diagrams in this section show the logical context of the solution. They intentionally contain only one generic Area Gateway.

A production installation may contain several Area Gateway instances serving different groups of machines and workplaces. The concrete distribution of gateways, machines, workplaces, and central infrastructure is described in Section 7 - Deployment View.
