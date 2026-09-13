# 7. Deployment View

[PASTE Deployment Diagram - Production Environment]

The deployment is divided into multiple independent production areas and centralized infrastructure.

Area A (Building 1) and Area B (Building 2) represent physically separated production areas. Each area contains its own machines, workplace applications, and a dedicated Area Gateway Host, which may be deployed as a virtual machine or a dedicated server. Each Area Gateway communicates only with the machines and workplaces assigned to its area. This keeps local production communication independent between buildings and limits the impact of an Area Gateway failure to the affected area.

Machine communication uses the corresponding industrial protocols, while workplaces communicate with their local Area Gateway through REST and SSE.

The centralized infrastructure is distributed across two physical servers:

- **Physical Server A** hosts the Central Panel and its Microsoft SQL Server storage. The Telemetry Worker, Configuration API, Telemetry API, and UI Dashboard are deployed here.
- **Physical Server B** hosts RabbitMQ and therefore forms an independent telemetry-buffering layer.

Area Gateways publish telemetry to RabbitMQ using AMQP. Separating RabbitMQ from the Central Panel provides an additional failure boundary: if Physical Server A or the Central Panel becomes unavailable, Area Gateways can continue publishing telemetry to RabbitMQ on Physical Server B. The messages remain stored there until the Telemetry Worker becomes available again and processes them into the Central Panel Storage.

The local Gateway Storage additionally protects telemetry before it has been successfully accepted by RabbitMQ, providing another level of resilience against temporary network or broker outages.
