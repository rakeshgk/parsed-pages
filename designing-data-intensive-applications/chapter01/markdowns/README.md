# Chapter 01 - Trade-Offs in Data Systems Architecture

When building an application, we typically integrate various software systems or services—such as databases and APIs—using application code. As developers, you have many tools available to choose from: different database systems with distinct characteristics, various caching approaches, and more. 

The critical question is: **How do you reason about trade-offs?** You must determine which tools and approaches are most appropriate for your specific task.

This book serves as a guide to help you make these decisions regarding technology selection and integration. No single approach is fundamentally superior; everything has pros and cons. This chapter compares contrasting concepts, evaluates their trade-offs, and helps you identify which approach best serves the needs of your application.

## Data Intensive vs. Compute Intensive Systems

**Data-Intensive Systems**
We classify an application as data-intensive if data management is a primary challenge during development. Key concerns typically include:
*   Storing and processing large volumes of data.
*   Managing data changes and ensuring consistency amidst failures and concurrency.
*   Maintaining high service availability.

**Compute-Intensive Systems**
We classify an application as compute-intensive if the primary challenge is parallelizing a very large computation.

## Operational vs. Analytical Systems

**Operational Systems**
Consist of backend services and data infrastructure where data is created (e.g., serving external users). In these systems, application code reads and modifies data in databases based on user actions.

**Analytical Systems**
Serve the needs of business analysts and data scientists. They contain a read-only copy of data from operational systems and are optimized for analytics-specific processing.

**Emerging Roles**
As these systems have matured, two specialized roles have emerged:
*   **Data Engineers:** Focus on integrating operational and analytical systems and managing the organization's broader data infrastructure.
*   **Analytics Engineers:** Model and transform data to make it more useful for business analysts and data scientists.

Most companies maintain operational and analytical systems separately.

### Online Transaction Processing (OLTP) vs. Online Analytical Processing (OLAP)

**OLTP**
Operational systems typically perform point queries (looking up a small number of records by key). Records are inserted, updated, or deleted based on user input. Because these applications are interactive, this access pattern is known as **Online Transaction Processing**.

**OLAP**
Users run analytical queries that scan over large numbers of records to calculate aggregate statistics (e.g., count, sum, or average) rather than returning individual records. To differentiate this pattern from transaction processing, it is called **Online Analytical Processing**.

| Property | OLTP | OLAP |
| :--- | :--- | :--- |
| **Main Read Pattern** | Point queries (fetching individual records by key) | Aggregate over large numbers of records |
| **Main Write Pattern** | Create, update, and delete individual records | Bulk import (ETL) or event streams |
| **Human User Example** | End user of web/mobile application | Internal analyst for decision support |
| **Machine Use Example** | Checking if a user action is authorized | Detecting fraud/abuse patterns |
| **Query Volume** | Lots of small queries | Few queries, each complex |

### Data Warehouses

Traditionally, companies ran both OLTP and OLAP workloads on the same database. A common trend arose for companies to separate these workloads: analytics moved to a dedicated system called a **Data Warehouse**.

1.  **ETL (Extract-Transform-Load):** Data is extracted from OLTP databases via periodic dumps or continuous streams, transformed into an analysis-friendly schema, cleaned, and then loaded into the warehouse.
2.  **ELT:** The order of Transform and Load is swapped; transformations occur *after* loading data directly into the warehouse (common with modern cloud warehouses).

**HTAP (Hybrid Transactional/Analytical Processing)**
Some database systems offer HTAP, aiming to enable OLTP and analytics in a single system without requiring ETL. However, many HTAP systems internally consist of an OLTP system coupled with a separate analytical system, hidden behind a common interface.

### Data Lakes

A data warehouse often uses a relational data model queried via SQL using specialized BI software. While this works well for business analysts, it is less suited for data scientists training ML models or extracting structured information from unstructured text (e.g., PDFs). Consequently, many data scientists prefer Python libraries like Pandas or frameworks like Spark.

To make data available to these tools, organizations use a **Data Lake**:
*   A centralized repository holding copies of data useful for analysis.
*   Data is obtained from operational systems via ETL processes.
*   Unlike a warehouse, a Data Lake simply contains files without imposing a specific file format, data model, or schema.

This flexibility makes Data Lakes cheaper than relational storage, as they can utilize commoditized object storage. While some view the Data Lake as an intermediate step leading to a Data Warehouse, it often holds raw data (the "Sushi Principle": *raw data is better*) until transformed by the consumer.

### Reverse ETL

Outputs from analytical systems are often fed back into operational systems. For example, an ML model trained on data in an analytical system may be deployed to production to generate recommendations for end users.

### System of Record vs. Derived Data

**System of Record (Source of Truth)**
Holds the authoritative or canonical version of data. When new data arrives, it is written here first. If a discrepancy exists between systems, the System of Record value is considered correct.

**Derived Data**
The result of existing data from another system being transformed or processed in some way. Because derived data duplicates information, it is technically redundant but essential for high-performance read queries. You can derive multiple datasets from a single source to view the data from different perspectives.

> **Note:** Most databases are just tools; they are not inherently systems of record or derived systems. How you use them determines their role in your architecture.

## Cloud vs. Self-Hosting

When building software, two decisions matter: who builds it and who deploys it.

**Build Decisions**
1.  Build in-house.
2.  Use off-the-shelf (open-source) software.
3.  Use external vendor software accessible via web interface or API.

**Deploy Decisions**
1.  Deploy in the cloud.
2.  Deploy on-premises.

Whether cloud services are cheaper and easier than self-hosting depends on your skills and workload predictability:
*   **Predictable Workloads:** If you possess the experience to operate necessary systems and loads are predictable, buying your own machines and running software yourself is often cheaper.
*   **Unpredictable/Complex Workloads:** If you lack expertise in deploying specific infrastructure or if load varies significantly (e.g., analytical scans), adopting a cloud service is easier and often more cost-effective.

### Cloud-Native System Architecture

The term "Cloud-Native" describes architecture designed specifically to take advantage of cloud services. Unlike traditional software that relies on generic computing resources (CPUs, RAM, filesystems) typically found in VMs (IaaS), cloud-native systems build upon **lower-level cloud services**.

*   **Benefits:** Better performance per hardware unit, faster failure recovery, scalable computing resources matching load, and support for larger datasets.
*   **Example:** Snowflake relies on S3 (object storage) rather than managing its own underlying disks.

### Separation of Storage and Compute

In traditional computing, disk storage is regarded as durable. To tolerate hard disk failure, **RAID** (Redundant Array of Independent Disks) is used to maintain copies of data across multiple physical disks attached to the same machine. 

In cloud-native systems:
*   **Compute Instances:** Typically have local disks treated as ephemeral caches (access is lost if the instance fails).
*   **Persistent Storage:** Uses virtual disk services (e.g., Amazon EBS, Azure Managed Disks) that emulate disk behavior but sit on a network layer.

Relying on virtual disks makes applications sensitive to network glitches, as every I/O operation becomes a network call. Consequently, cloud-native services often separate storage and compute to use dedicated storage services optimized for specific workloads, rather than relying on local ephemeral disks or network-emulated virtual disks.

### Operations in the Cloud Era

Traditionally, organizations employed Database Administrators (DBAs) or System Administrators (SysAdmins) to manage server-side infrastructure. In modern cloud-native teams, roles are integrating: software development and operations share responsibility for backend services and data infrastructure.

The shift emphasizes moving from individual machines to **Services**. Capacity planning becomes financial planning, and performance optimization becomes cost optimization.

## Distributed vs. Single-Node Systems

A system involving several machines communicating via a network is called a **Distributed System**. Each participating process is called a **node**.

### Microservices and Serverless

**Service-Oriented Architecture (SOA)**
One way to distribute a system is dividing it into clients and servers. The same process can be both a client and a server making requests.

**Microservices**
Refined from SOA, a microservice has one well-defined purpose, exposes APIs for network access, and is managed by one team. In a pure microservice architecture, each service typically manages its own database.

**Serverless (Function as a Service - FaaS)**
Management of infrastructure is outsourced to a cloud vendor.
*   **VMs:** You explicitly start/stop instances.
*   **Serverless:** The provider allocates and frees hardware resources automatically based on incoming requests (metered billing). You pay only for runtime, not provisioned capacity.
*   *Note:* Services may suffer from cold starts when a function is first invoked.

### Cloud Computing vs. Supercomputing

**Supercomputing (High-Performance Computing - HPC)**
Typically used for computationally intensive scientific tasks (weather forecasting, climate modeling). Supercomputers run large batch jobs that checkpoint state to disk. If a node fails, the cluster stops, repairs the node, and resumes from the last checkpoint. Nodes are geographically close together.

**Cloud Computing**
Tends to be used for online services and business data systems serving user requests with high availability. Cloud nodes can be distributed across multiple geographic regions, unlike supercomputers which assume local proximity. Stopping a cluster is undesirable in cloud-native environments because the service needs to continuously serve users.
