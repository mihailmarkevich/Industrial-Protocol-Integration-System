# 2. Constraints

The architecture of the Industrial Protocol Integration System is influenced by several technical, organizational, and operational constraints originating from the existing production environment.

These constraints limit the available solution space and must be respected by the target architecture.

## 2.1 Existing workplace applications must be preserved

The existing workplace applications are already used in production and cannot be completely rewritten or replaced as part of this project.

The integration architecture must therefore allow existing applications to continue fulfilling their current responsibilities while requiring only limited adaptations for communication with the new integration layer.

## 2.2 Existing machines cannot be replaced or fundamentally modified

The industrial machines are already installed and operational.

The project cannot assume modifications to:

- machine firmware;
- PLC programs;
- vendor-specific communication interfaces;
- supported industrial protocols;
- internal machine control logic.

The integration system must adapt to the interfaces already provided by the machines.

## 2.3 Existing machine communication protocols must be supported

Machines in the production environment use different industrial communication technologies, including protocols such as:

- OPC UA;
- OPC Classic;
- Modbus/TCP;
- vendor-specific interfaces.

The architecture cannot require all machines to expose one common protocol.

Protocol differences must therefore be handled by the integration system.

## 2.4 Production operation must remain independent from central services where possible

Local communication between workplaces and machines is part of the production process.

Temporary unavailability of central monitoring, telemetry, or management services should therefore not unnecessarily interrupt communication required for local production operation.

The architecture must distinguish between communication required locally for production and functionality that can depend on central infrastructure.

## 2.5 Production environments may contain multiple independent machine groups

Machines and workplaces are distributed across different parts of the production environment.

Communication architecture must therefore support logical or physical separation of machine groups and must not assume that all industrial communication can or should be routed through one central runtime instance.

## 2.6 Network connectivity cannot be assumed to be permanently available

Connections between production infrastructure and central services may temporarily become unavailable because of:

- network failures;
- maintenance;
- service restarts;
- infrastructure outages.

The architecture must tolerate temporary connectivity interruptions where technically reasonable.

## 2.7 Direct machine connections must remain controlled

Industrial machines may support only a limited number of simultaneous client connections or sessions.

The architecture should therefore avoid uncontrolled growth of direct machine connections as additional workplace applications are introduced.

Multiple consumers should be able to use machine information without each consumer necessarily establishing an independent machine connection.

## 2.8 Machine-specific behavior cannot be completely eliminated

Although the system should provide standardized interfaces to consumers, individual industrial protocols and machines may expose capabilities that cannot be represented identically across all machine types.

The architecture must therefore allow protocol-specific or machine-specific behavior where necessary instead of assuming complete technical uniformity.

## 2.9 Existing customer systems remain outside the ownership of the integration platform

Existing customer applications, databases, and production systems that are not part of the Industrial Protocol Integration System remain externally owned systems.

The new architecture may integrate with them but should not assume responsibility for redesigning or replacing their internal data models or business logic.

## 2.10 The system operates in an industrial environment

The solution is intended to run as part of an existing production infrastructure.

Deployments, updates, failures, and restarts must therefore consider the impact on active production processes.

Architectural solutions that require unnecessary interruption of machine communication should be avoided.

## 2.11 Hard real-time control is outside the project scope

The Industrial Protocol Integration System is designed for machine integration, data distribution, telemetry, configuration, and application-level communication.

It is not intended to replace PLC-level or other deterministic real-time control mechanisms.

Functions requiring strict deterministic timing or hard real-time guarantees must remain sufficiently close to the machine or within dedicated industrial control systems.

## 2.12 Machine safety logic remains outside the integration system

Safety-critical machine behavior must remain implemented within the appropriate industrial control and safety systems.

The integration platform must not become the sole mechanism responsible for safety-critical machine control.

## 2.13 Existing production databases are not redesigned by this project

Where customer applications already maintain their own operational or business databases, those databases remain outside the architectural ownership of the integration platform unless an explicit integration interface is required.

The system may collect its own telemetry, configuration, and integration-related data without replacing the customer's existing business data storage.
