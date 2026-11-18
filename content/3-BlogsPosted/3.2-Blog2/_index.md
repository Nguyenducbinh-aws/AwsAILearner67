---
title: "Blog 2"
date: 2026-11-18
weight: 2
chapter: false
pre: " <b> 3.2. </b> "
---

### MOVING MICROSOFT SYSTEMS TO AWS: 30% LOWER LICENSE COST AND 60% LOWER STORAGE COST

**Published on:** AWS Study Group (Facebook)

Moving an old .NET Framework and SQL Server system to the cloud with a lift-and-shift approach does not solve the problems of license cost and scalability. A real example from TEKsystems Global Services (TGS) shows how the whole Microsoft platform of a university was modernized into a cloud-native architecture on AWS.

![Cloud-native architecture on AWS: Route 53, WAF, CloudFront, API Gateway, ECS, Aurora, CloudWatch](../../images/3-BlogsPosted/3.2-Blog2/cloud-native-architecture.png)

*Source: AWS Blog (see the references below).*

**Key points:**

- **System analysis:** the monolith had 785,000 lines of code, 342 tables and 63 stored procedures. The team used CAST Highlight (code risks and barriers to the cloud), CAST Imaging (visualizes data flow and dependencies, cutting analysis time by 80%) and Babelfish Compass (checks how well SQL Server can move to PostgreSQL).
- **Analysis result:** only about 20,000 lines of code sit in the database layer, so replacing SQL Server was practical.
- **Monolith to microservices:** AWS Microservices Extractor for .NET analyzes the application and splits it into independent services.
- **.NET Framework to .NET 8:** Porting Assistant and AWS Transform for .NET, running on Linux and AWS Graviton (Arm64).
- **Database and storage:** SQL Server moves to PostgreSQL, and Amazon FSx moves to Amazon S3.
- **New architecture based on Well-Architected:** Amazon ECS runs Linux containers, Amazon ELB and Route 53, Amazon S3 and CloudFront, AWS WAF and Amazon Cognito, Amazon API Gateway, Amazon CloudWatch, and AWS Organizations for central management and audit logs.
- **Two cost results:** 30% lower commercial license cost and 60% lower storage cost.

**Conclusion and thoughts:**

Cloud modernization is not just moving servers to the cloud. Restructuring the application, choosing the right platforms and optimizing the new architecture are what reduce cost, improve scalability and make operations more efficient.

**References:**

1. AWS Blog: How TEKsystems modernized a university's Microsoft workloads on AWS — [aws.amazon.com/blogs/modernizing-with-aws](https://aws.amazon.com/blogs/modernizing-with-aws/how-teksystems-modernized-a-universitys-microsoft-workloads-on-aws)
2. AWS Transform for .NET (AWS Documentation, User Guide).
3. AWS Microservices Extractor for .NET (AWS Documentation).
4. Modernize Windows Workloads — [aws.amazon.com/vi/transform/windows](https://aws.amazon.com/vi/transform/windows/)

**Link:** [Post on AWS Study Group](https://www.facebook.com/groups/660548818043427/?multi_permalinks=2340011250097167)
