---
title: Services
doc_status: needs-verification
last_reviewed:
reviewed_by:
review_notes:
tags:
  - stac
  - data-catalogues
  - needs-verification
---
Overview diagram:
```mermaid
flowchart TB
   GHCatRepo[GitHub catalogue repo]
   UpstreamSTAC[Upstream STAC API]

   subgraph Harvesters
     githarvester[Git Harvester]
     stacharvester[STAC Harvester]
     otherharvest[...]
   end

   GHCatRepo --> githarvester
   UpstreamSTAC --> stacharvester

   transformer[Transformers]

   githarvester --> transformer
   stacharvester --> transformer
   otherharvest --> transformer

   subgraph ingester[Ingesters]
        searchingest((Search\nIngest))
        anningest((Annotations\nIngest))
        otheringest((...))
    end

    transformer --> searchingest
    transformer --> anningest
    transformer --> otheringest
    
    searchdb[stac-fastapi elasticsearch]
    anndb[Annotations DB]
    otherdb[...]

    searchingest --> searchdb
    anningest --> anndb
    otheringest --> otherdb
```


More detailed diagram:
```mermaid
flowchart TB
    GHCatRepo[GitHub catalogue repo]
    PulsarH[Pulsar 'harvested' Topic]
    PulsarT[Pulsar 'transformed' Topic]

    subgraph githarvester[Git Harvester]
       direction TB
       clone((Clone/\npull))
       repocache[Repo Cache]
       scan((Scan\nchanges))
       githarvested[Git Harvester S3]

       clone --> repocache --> scan --> |Harvested files| githarvested
    end

    GHCatRepo --> clone
    scan --> |Change message\n&lpar;points to files&rpar;| PulsarH


   UpstreamSTAC[Upstream STAC API]
   subgraph stacharvester
       stacharvest[STAC Harvest]
       harvestedstac[Harvested STAC\n&lpar;if upstream private&rpar;]
       stacharvest --> harvestedstac
   end
   UpstreamSTAC --> stacharvest
   stacharvest --> |Change messages\npoints to S3 or original STAC| PulsarH

    subgraph transformer[Transformers]
        direction TB
        transformerdispatch((Transformer\ndispatch))
        transformer1t1((STAC\nrewrite))
        transformer2t1((...))
        transformer1t2((Other format))
        transformeroutput((Transformer\noutput))
        transformed[Transformer Output S3]

        transformerdispatch --> |STAC| transformer1t1 --> |STAC| transformer2t1 --> |STAC| transformeroutput --> |output file| transformed
        transformerdispatch --> |file type 2| transformer1t2 --> transformeroutput
    end

    PulsarH --> |Change message| transformerdispatch
    githarvested --> |Changed files| transformerdispatch
    stacharvest --> |Changed files| transformerdispatch
    transformeroutput --> |Change message| PulsarT


    subgraph ingester[Ingesters]
        searchingest((Search\nIngest))
        anningest((Annotations\nIngest))
        otheringest((...))
    end

    PulsarT --> |Change messages| searchingest
    PulsarT --> |Change messages| anningest
    %%PulsarT --> |Change messages| otheringest
    transformed --> |Changed files| searchingest
    transformed --> |Changed files| anningest
    %%transformed --> |Changed files| otheringest


    searchdb[stac-fastapi elasticsearch]
    anndb[Annotations DB]
    otherdb[...]
    searchingest --> searchdb
    anningest --> anndb
    otheringest --> otherdb
```


### Services

```puml
@startuml

package WebPresence {
  [Catalog Browser] as CatB
  [Wagtail]
  CatB -> Wagtail : served by
}


node "Static catalogue - S3" as S3 {
  [Private Metadata Catalogue] as NonSTACC
}

package "Search Service" as SearchService {
  [STAC API\n(stac-fastapi)] as Search
  [Records API\n(pygeoapi)] as Records
  [Elasticsearch] as ES
  Search --> ES
  Records --> ES
}

[Annotations Service] as Annotations
package "Data Access Services" as DAS

CatB --> Search : Dynamic STAC API
CatB --> Records : OGC API - Records
CatB --> Annotations : Annotations API
CatB --> DAS : "WMTS"
'SearchService -> Annotations

[Other EODHP Service] as EODHPService
NonSTACC <-- EODHPService
SearchService --> NonSTACC

@enduml
```


### STAC Harvester configuration ingest

This configures persistent 'harvesters' into the system based on a configuration file in Git. The configuration is stored persistently in the form of a Kubernetes custom resource, STACHarvester.

```mermaid
flowchart TB
   subgraph GHCatRepo[GitHub catalogue repo]
      harvesterconfigfile[STAC Harvester config file]
   end


   githarvester((Git Harvester))
   transformer((Transformers))
   harvesterconfigingester((Harvester\nconfig\ningester))

   subgraph Kubernetes
      subgraph WSNamespace[Workspace Namespace]
         stacharvestercr[STACHarvester CR]
      end
   end


   harvesterconfigfile --> |harvest config| githarvester --> |Pulsar msg + config| transformer  --> |Pulsar msg + config| harvesterconfigingester  --> |creates/manages| stacharvestercr
   

   UpstreamSTAC[Upstream STAC API]
   stacharvester[STAC Harvester]
   stacharvesternext[...]

   stacharvestercr --> |harvest config| stacharvester --> stacharvesternext
   UpstreamSTAC --> stacharvester
```
