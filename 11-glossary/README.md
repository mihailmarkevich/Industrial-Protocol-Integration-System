# 11. Glossary

This glossary defines the most important domain-specific and architectural terms used throughout the documentation.

| Term | Definition |
|---|---|
| Area | A logical or physical production segment, for example a building or production zone, containing a group of machines and workplaces served by a dedicated Area Gateway. |
| Area Gateway | Local integration system responsible for communicating with industrial machines within an Area and providing standardized access to them for Workplace applications. It also forwards telemetry to the Central Panel. |
| Gateway Application | Main application of an Area Gateway containing machine communication, workplace integration, telemetry handling, and configuration functionality. |
| Gateway Storage | Local persistent storage owned by an Area Gateway. It contains the Telemetry Outbox, Recent Event Journal, and Applied Configuration. |
| Central Panel | Centralized part of the solution responsible for telemetry processing and storage, configuration management, monitoring, reporting, analytics, and centralized APIs. |
| Machine | Existing industrial equipment integrated with the solution through its available communication interface. Internal machine control logic remains outside the solution. |
| Workplace | Existing production application that accesses machine data and executes supported machine operations through an Area Gateway. |
| Protocol Adapter | Area Gateway component that isolates machine- and protocol-specific communication from the remaining gateway functionality. |
| Telemetry | Machine and integration data collected centrally for persistence, monitoring, reporting, analytics, and system-health observation. |
| Telemetry Outbox | Durable local storage for telemetry that has not yet been successfully accepted by the central messaging infrastructure. |
| Recent Event Journal | Short-term local storage of replayable machine events used to recover events missed by a Workplace during a temporary connection interruption. |
| Applied Configuration | Last successfully validated and activated configuration currently used by an Area Gateway. |
| Configuration Candidate | Newly retrieved configuration version that has not yet replaced the currently applied configuration and must first be validated and activated. |
