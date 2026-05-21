---
title: 3. ARCHITECTURAL DESIGN
doc_status: unreviewed
last_reviewed:
reviewed_by:
review_notes:
---
## 3. ARCHITECTURAL DESIGN

This section presents our architectural design in the form of a high-level overview followed by subsections that provide details of the architecture components. 

Our solution is guided by the following core principles: 

- The re-use of open-source component solutions, including the EOEPCA Reference Implementation, and other open-source components, most of which are widely used both in the EO community and within other TPZ-UK projects; 
- Loose coupling, scalability and extensibility through the use of the Apache Pulsar messaging system to combine these components; 
- Cloud-native computing, using Kubernetes, object stores and microservices; - Upgrade paths towards a highly scalable, available, interconnected and widely used system are considered and mapped; 
- Gitops-based management of the system, its configuration and some of its contents. 

