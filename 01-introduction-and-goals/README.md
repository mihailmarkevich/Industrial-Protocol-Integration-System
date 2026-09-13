# 1. Introduction & Goals

## 1.1 Introduction

The Industrial Protocol Integration System is intended to provide a standardized integration layer between existing industrial machines, local workplace applications, and central monitoring and configuration services.

The production environment consists of multiple machines from different vendors. These machines already exist and expose different industrial communication protocols, such as OPC UA, OPC Classic, Modbus/TCP, or vendor-specific interfaces.

Existing workplace applications communicate with these machines in order to read production data, receive status information, and, where required, execute commands or write values.

Historically, these integrations were implemented individually for particular machines and workplaces. As the number of machines, protocols, and workplace applications increased, this approach resulted in duplicated integration logic, strong dependencies between applications and machines, and limited possibilities for centralized telemetry and monitoring.

The goal of the new architecture is therefore not to replace the existing machines or workplace applications, but to introduce a common integration layer around them.

## 1.2 Existing System and Architectural Problems

Before introduction of the Industrial Protocol Integration System, workplace applications communicate directly with industrial machines or use individually developed machine-specific integrations.

Each integration may therefore depend on a particular industrial protocol, machine model, vendor interface, or data representation.

[PASTE Existing System Overview Diagram]

### Problems of the Existing Architecture

| Problem | Consequence |
|---|---|
| Protocol-specific workplace integrations | Workplace applications require knowledge about machine-specific protocols, interfaces, or data structures. |
| Tight coupling between workplaces and machines | Changes to a machine, protocol, address structure, or vendor interface can require modifications to workplace software. |
| Too many connections to a single machine | Multiple workplace applications may establish separate connections to the same machine. This increases the number of sessions, consumes machine and network resources, may exceed machine-specific connection limits, and reduce communication performance. |
| Duplicated integration logic | Connection handling, conversion, error handling, reconnect behavior, and similar functionality may be implemented independently by multiple applications. |
| No common centralized telemetry path | Machine states, measurements, communication failures, and integration health cannot be collected and evaluated consistently from one central location. |
| Limited observability | Troubleshooting requires investigation of individual workplace applications and machine connections instead of a standardized integration layer. |
| Increasing maintenance complexity | The number of direct dependencies grows as additional workplaces, machines, and protocols are introduced. |

## 1.3 Goals of the New Architecture

The new architecture shall introduce a standardized integration layer while preserving the existing production environment.

The main architectural goals are:

1. **Decouple workplace applications from industrial protocols.**  
   Workplace applications should communicate through standardized interfaces instead of implementing OPC UA, Modbus, OPC Classic, or other machine-specific protocols directly.

2. **Centralize machine communication within production areas.**  
   Machine-specific communication and protocol handling should be moved out of individual workplace applications into a common integration layer. Multiple workplace applications should be able to consume machine data through this shared layer, reducing the number of direct connections established to individual machines.

3. **Support heterogeneous industrial protocols.**  
   The architecture should allow different machine protocols to coexist behind a common integration model.

4. **Preserve existing workplace applications.**  
   Existing workplaces should require only the minimum changes necessary to use the new integration interfaces and should not need to be completely rewritten.

5. **Provide centralized telemetry.**  
   Machine measurements, events, communication states, and gateway health information should be transferable to a Central Panel for monitoring and analysis.

6. **Keep local production communication independent from central services.**  
   Communication between local workplaces and machines should not require the Central Panel or the central telemetry infrastructure to be continuously available.

7. **Improve extensibility.**  
   Additional machines, protocols, workplaces, and production areas should be introducible without redesigning the complete system.

8. **Improve fault isolation.**  
   Failures within one production area should have minimal influence on other production areas.

9. **Improve maintainability and observability.**  
   Integration behavior, failures, connection states, and message processing should be easier to understand, monitor, and troubleshoot.

## 1.4 Quality Goals

The following quality goals have the highest architectural priority.

| Priority | Quality Goal | Description |
|---:|---|---|
| 1 | Interoperability and Extensibility | Different industrial protocols and machine types can be integrated through clearly separated protocol-specific adapters without requiring changes to consuming workplace applications. |
| 2 | Reliability and Fault Isolation | Temporary failure of central services or another production area should not unnecessarily interrupt local machine-to-workplace communication. |
| 3 | Maintainability | Machine communication, protocol handling, local workplace communication, telemetry, configuration, and persistence should remain separated into understandable architectural responsibilities. |
| 4 | Scalability | Additional machines, workplaces, and production areas should be supported primarily by adding or configuring integration components rather than redesigning the architecture. The number of direct connections to an individual machine should remain controlled as the number of workplace applications increases. |
| 5 | Operability and Observability | Communication status, failures, stale connections, connection counts, and relevant telemetry should be observable in a consistent manner. |

Detailed and testable quality scenarios are specified in Section 9 - Quality Requirements.

## 1.5 Scope of the Architectural Improvement

The architecture focuses on improving the integration between existing machines and software systems.

It does not attempt to replace the industrial machines, redesign their internal control logic, or completely replace existing workplace applications.

The main architectural change is the introduction of a standardized integration layer that separates machine-specific communication from workplace applications, controls the number of connections established to each machine, and provides an additional path for centralized telemetry and configuration.

The resulting target architecture is described in the following sections, beginning with the architectural and technical constraints in Section 2 and the system context and boundaries in Section 3.
