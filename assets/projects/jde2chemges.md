# JDE to ChemGes: Seamless Integration Between Oracle JD Edwards ERP and ChemGes SDS Software

When the company I worked, decided to adopt ChemGes for managing Safety Data Sheets (SDS) and ensuring we met regulatory compliance, we quickly faced a challenge. Our primary ERP system was Oracle JD Edwards E1, but ChemGes operated on a completely different data model and used file-based storage. This created a barrier since there was no native way for the two systems to exchange information. To overcome this obstacle, I took the lead in designing and developing a synchronization platform that would serve as the communication link between Oracle JD Edwards and ChemGes.

Rather than connecting the two systems directly, I created an intermediate SQL Server database that acted as a staging area for all the data coming in and going out. This approach allowed both systems to work independently while providing a synchronization process that was reliable, traceable, and recoverable.

The integration transferred all information required for ChemGes to generate and maintain regulatory data. Once ChemGes completed its calculations, the generated regulatory information was synchronized back into Oracle JD Edwards, ensuring that both systems remained consistent while each continued to perform the tasks it was designed for.

Because the two applications had fundamentally different data structures, much of the project focused on designing transformation rules rather than simply copying data. The synchronization process included extensive validation, duplicate detection, dependency checks and detailed logging to guarantee data integrity.

Reliability was a major design goal. Every synchronization produced execution statistics, progress information and detailed logs showing which records had been processed successfully and which required attention. Failed records were isolated and retried without interrupting the remaining synchronization process, allowing the system to continue operating even when individual records contained issues.

The architecture was designed around a clear separation of responsibilities. Oracle JD Edwards remained the authoritative source for logistics and product information, while ChemGes handled regulatory calculations and generated the compliance information required by the business. The synchronization platform ensured that each system exchanged only the information relevant to its role.

The project required approximately five months of architecture design, implementation, testing and optimization before being deployed into production and I was responsible for the complete solution.

The integration has remained reliable in production, significantly reducing manual work while allowing both enterprise systems to operate together as a single business process.

## Technologies

* C#
* .NET
* ADO.NET
* SQL Server
* T-SQL
* Windows Service
* Oracle JD Edwards E1
* ChemGes
