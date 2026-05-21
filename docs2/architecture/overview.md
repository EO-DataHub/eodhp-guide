---
title: 2. OVERVIEW
doc_status: unreviewed
last_reviewed:
reviewed_by:
review_notes:
tags:
  - aws
  - kubernetes
---
## 2. OVERVIEW

### 2.1 Context

Federation of national data sets, information sources and processing facilities is essential for the growth of the UK space economy. The creation of a financially and operationally sustainable EO Data Hub Platform (EODHP) will break down data silos and allow stakeholders from UK industry and research/academia to work together in a centralised manner, offering a better model for research to commercial service delivery, to support a range of sectors including green finance, energy, infrastructure, and climate change monitoring. 

The EODHP offers a solution to a well-recognised need for more findable, accessible, interoperable and re-usable (FAIR) EO, climate and other data. The number of data platforms currently used by UK stakeholders is vast and can be broadly categorised into data archives and data processing services. In time, the EODHP can act as a conduit to these existing services, massively simplifying the user journey. 

Through the EODHP, the UK has an incredible opportunity to grow its EO space economy by providing federated access to quality assured data products, stimulating collaboration across the UK, and forming international partnerships with key organisations across Europe, the Commonwealth and the USA. There is an international trend towards cloud 

hosted platforms and data lakes, which provides opportunities for collaboration within those environments. However, there is a danger that each effectively becomes an isolated silo. The challenge is therefore how to interact with other initiatives. 

In response, the EODHP design relies, where possible, on common open interface/API standards for software services, and so enables improved interoperability and federation between platforms. This approach facilitates federated data discovery & access, data processing – with authorized access to protected resources through federated user identity. 

### 2.2 High-level Design Goals

There are many different types of potential end user of the EODHP with varying levels of knowledge and understanding, including; expert science users, service developers, commercial users, government departments, satellite data providers, as well as other platforms offering compute infrastructure for users. 

To take account of these varying use-cases we identify some high-level design goals for the platform: 

- **Federated Data Discovery & Access** - Simplified and inclusive *Find* and *Access* to open and commercial data archives regardless of where they are hosted. 
- **Data-proximate User-defined Processing** - Allow the user to ‘bring their processing to the data’ e.g. by ‘bundling’ the algorithm in a form that can be deployed to the platform and executed against platform data
- **Federated Identity & Access Management** - Establish a common user authorisation model, in which resources are owned, and access to resources is protected and accounted for through federated user access. The federation of service-to-service interfaces across platform boundaries aids interoperability in a federated user environment. 
- **Intuitive user interface** - The UX design should be clear and follow an intuitive style guide to ensure user engagement and satisfaction. 
- **Data Fusion** - Establish strategies for ‘interoperable data’ for exploitation of data across domains. For example, standard discovery APIs can perform metadata mapping to facilitate the establishment of interoperable metadata interpretation amongst collaborating services.
- **Data Quality** - Establish the UK Data Hub Platform as a trusted data source, providing a toolset to cleanse, filter, decode, de-duplicate and validate to improve overall quality. Provide product quality, uncertainty and instrument health metrics and/or reports for data, offering data sets and derived information with readily interpretable quality metrics for users to assess confidence and ‘fitness for purpose’. 
- **Scalable Storage & Compute** - Rapidly scalable data storage for online, near-line and cold storage and user access to highly flexible and scalable, cloud-based compute infrastructure. 
- **Highly Available** - Supporting service providers to deliver reliable operational services, through a design that provides business continuity and disaster recovery. 
- **Accounting and Costing** - Facilitate platform uptake by provision of a clear platform cost model through resource consumption accounting. 

