# Building a Synchronization Platform Between an ERP and a Regulatory Compliance System

During one of my previous roles, the company adopted a specialized regulatory compliance platform for generating Safety Data Sheets (SDS) and managing product compliance information. The existing ERP system remained the primary source of operational data, but the two applications had fundamentally different data models and storage mechanisms. Since there was no native integration between them, I took the lead in designing and developing a synchronization platform that bridged the gap.

Rather than connecting the systems directly, I introduced an intermediate SQL Server database that acted as a staging area for all incoming and outgoing data. This approach allowed both applications to operate independently while providing a synchronization process that was reliable, traceable, and recoverable.

The integration transferred all information required by the regulatory platform to generate and maintain compliance data. Once the regulatory calculations were completed, the resulting information was synchronized back into the ERP, ensuring both systems remained consistent while each continued to perform the responsibilities it was designed for.

Because the two applications used completely different data structures, much of the project focused on designing transformation and mapping rules rather than simply copying data between systems. The synchronization process included extensive validation, duplicate detection, dependency checks, and detailed logging to guarantee data integrity.

Reliability was one of the primary design goals. Every synchronization produced execution statistics, progress information, and detailed logs showing which records had been processed successfully and which required attention. Failed records were isolated and retried without interrupting the remaining synchronization process, allowing the system to continue operating even when individual records contained issues.

The architecture followed a clear separation of responsibilities. The ERP remained the authoritative source for business and logistics data, while the regulatory platform handled compliance calculations and generated the required regulatory information. The synchronization layer ensured that each system exchanged only the information relevant to its own domain.

The project required approximately five months of architecture design, implementation, testing, and performance optimization before being deployed into production. I was responsible for the complete solution, including the architecture, synchronization engine, desktop application, Windows Service, validation logic, logging, and performance optimization.

The integration significantly reduced manual work while allowing the two enterprise systems to operate together as a unified business process. The solution remains in production today, continuing to synchronize data reliably between both systems.

## Technologies

- C#
- .NET
- ADO.NET
- SQL Server
- T-SQL
- Windows Service
