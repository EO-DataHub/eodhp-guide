---
title: 3.5 Resource Catalogue Contents
doc_status: ok
last_reviewed:
reviewed_by:
review_notes:
tags:
  - stac
  - data-catalogues
---
### 3.5 Resource Catalogue Contents

#### 3.5.1 Catalogue Paths

The catalogue has a hierarchical structure and houses objects of a variety of types (STAC Catalogs, STAC Collections, STAC Items, OGC API Processes Processes and Jobs, etc). It’s useful to have a way to refer to the location of an object in a hierarchy which is unambiguous and unique, and which easily corresponds to API locations. This is an EODH ‘catalogue path’ or cat-path. 

Where an object is available via the catalogue APIs the cat-path is always the same as the API path relative to the catalogue root – for example, `/catalogs/public/catalogs/ceda/collections/sentinel1-ard.` The object types (‘catalogs’ and ‘collections’ here) are included to avoid having multiple URLs for the same object whilst adhering to standards and conventional REST layouts. For example, /catalogs returns a list of sub-Catalogs of the root Catalog so, for a conventional API structure, `/catalogs/my-cat` should be the URL for sub-Catalog my-cat. 

Cat-paths differ from API paths because they can also be applied to objects not available via the API. Objects harvested from external sources are assigned a source cat-path by the harvester – eg, upstream location `https://api.stac.ceda.ac.uk/collections/cmip6` is assigned cat-path `/catalogs/ceda-stac-catalog/collections/cmip6` (ceda-stac-catalog is the ID the CEDA STAC API gives for its root Catalog). Part of the job of a transformer is to translate the source cat-path into the destination cat-path. This is typically done by replacing a prefix of the source cat-path with the destination cat-path. 

#### 3.5.2 Catalogue Structure

The STAC catalog will start with some initial structure. Relative to the STAC catalog root, `https://eodatahub.org.uk/api/catalogue/stac/`:

- **/** is the root Catalog (self, parent and root links all match),
- **/catalogs/public** contains a sub-Catalog for freely available datasets hosted by the platform. This is fully public and contains collections from the CEDA STAC Catalogue such as /catalogs/public/catalogs/ceda-stac-catalogue/collections/ukcp. 
- **/catalogs/commercial** contains a sub-Catalog for each commercial data provider collaborating with EODH. This metadata is public but the associated data will only be available for purchase. 
- **/catalogs/user** contains a sub-Catalog for each workspace. These are the ‘workspace catalogues’ and are under the control of the members of their owning workspace. Only the workspace has write access, but workspaces can choose to make parts of their workspace catalogue public for read access. 
- **/catalogs/user/catalogs/\<workspace-name\>/catalogs/commercial-data** will be used to store metadata for commercial data previously ordered by a particular workspace. 
- **/catalogs/user/catalogs/\<workspace-name\>/catalogs/processing-results** will be used by default to store the outputs of workflows run within a particular workspace. 

Workspaces are constrained to write to their workspace catalogue and its sub-Catalog. However, it’s intended that this could be made more flexible so that special-purpose workspaces could be used for harvesting public and commercial data. This would require additional access control in the transformers. 

Access policies can be defined for Catalogs and Collections in this structure, currently a choice of public vs private. If a policy is applied to a Catalog then it applies by default all descendent Catalogs and Collections that do not have their own policy. Items always inherit their Collection’s policy. If any object is visible to a user due to its access policy then its ancestor Catalogs also become visible (but not other contents of those Catalogs). 

#### 3.5.3 Data Streams

A data stream is a source of data which may be used by EODH users using platform functionality. Data streams are not a specific component, but rather place requirements on a number of other components which must work together to make data available to users and to define the interface to data streams. 

Data streams may separately 

- be commercial or open, 
- require account linking or not, meaning that a user must first authorize their EODH workspace to use the user’s account at the data provider for data access, 
- have an order-based interface (users make an asynchronous request for data) or a direct file-based interface (users follow HTTPS or S3 links provided as STAC assets), - use direct or adaptor-based upstream retrieval from the data provider, 
- be platform-published or published by a user, 
- and be hosted inside or outside the hub. 

All current data streams are either 

- commercial streams with account linking, order-based interfaces, adaptor-based retrieval, platform-published and hosted outside the hub (Planet and Airbus), or 
- open streams with no account linking, direct file-based interfaces, direct retrieval, platform published and hosted outside the hub (CEDA STAC). 

Each stream is allocated a sub-Catalog within the relevant top-level Catalog (‘public’ or ‘commercial’). Typically, this will contain one Collection for each of the stream’s datasets but there is no limitation preventing a more complex structure of sub-Catalogs being created. 

Special-purpose workspaces may be created for data streams. This is useful to apply special privileges (as required by commercial data streams to retrieve linked-account credentials) or if the stream will use significant resources because the accounting system can be used to track them. It will also be useful if user-managed harvesting features are added, such as access to harvester status and logs in the UI, as system operators can be added to the workspace to allow managing the stream. 

#### 3.5.4 Data Streams and the Catalogue

Each data stream must provide catalogue metadata which can be harvested by EODH into the platform catalogue. This can be provided in a number of ways: 

- **STAC harvesting**: by providing a STAC API which the STAC harvester can harvest,
- **custom harvester**: by providing a proprietary or other non-STAC API and having the platform developers write a custom harvester, 
- **delegated search**: by providing an API and having the platform developers write a custom proxy to make it appear within the EODH API hierarchy without harvesting,
- **Git**: by writing STAC into a Git repository, 
- **Workflow**: with a harvester workflow. 

##### 3.5.4.1 STAC Harvester-Based Harvesting

In many cases, data stream metadata is already provided as a STAC catalogue which EODH should harvest. The STAC Harvester is a containerized CLI-based tool which can be configured to be run on a schedule, such as once per day. This can be done in one of two ways: 

- By creating a CronJob Kubernetes resource using ArgoCD, 
- By creating a harvester configuration in a harvested Git repo. 

In the latter case a STACHarvester custom resource is created in a workspace, along with the required Argo Events EventSource and Sensor to trigger harvesting on a schedule. There is currently little advantage to this second option which exists as a step towards user managed harvesters, but it is the method used for harvesting the CEDA catalogue. 

##### 3.5.4.2 Custom Harvester

In some cases a custom harvester may be necessary. This parallels the STAC Harvester, being configured in the same way and producing compatible messages, but can talk to any API required, be triggered using any mechanism required and perform conversion to STAC. 

The Airbus custom harvester is configured as four separate harvests (one per dataset) run once per day. These are configured using ArgoCD, which creates Argo Events calendar based EventSources and associated Sensors which run a CLI harvest tool. 

##### 3.5.4.3 Delegated Search

In scenarios where harvesting a data catalogue is technically challenging, such as for Planet due to its size, a delegated search method can be used instead of harvesters. With this method, the Catalog at `/catalogs/commercial/catalogs/planet` and the Collections at `/catalogs/commercial/catalogs/planet/collections/\<collection-id\>` are served by the usual EODH STAC API. However, `/catalogs/commercial/catalogs/planet/search` and all of the sub-paths of the collections are served by a custom proxy which makes on-the-fly upstream connections to the Planet APIs. 

Where delegated search is used, the Catalogs and Collections can be found through the usual global EODH STAC APIs but the Items cannot (eg, /search will not return them). 

A future enhancement could be to provide full federated search by integrating upstream search results into the global search. This is potentially difficult to achieve whilst still providing a useful order for search results. 

##### 3.5.4.4 Git-Based Harvesting

This method may be suitable for small manually constructed datasets and for publishing workflows or other similar resources. It can also serve as a way to introduce version controlled configuration into the harvest pipeline – its only current use is to harvest STAC harvester configuration for the CEDA harvest. Future potential use in this way could include harvesting access policies and harvesting annotations. 

##### 3.5.4.5 Harvester Workflows

A harvester workflow is a custom workflow which harvests metadata from the data stream provider, converts it to STAC and emits it as a workflow output. This will then be processed by the Workflow Harvester, resulting in it being added to the catalogue. Such workflows can use workspace stores for keeping state, such as records of what has and has not been previously harvested. 

The location to which a workflow can harvest data is currently limited and must be within the owning workspace’s sub-Catalog. Future work would be needed to allow write access to public and commercial Catalogs, most likely by adding workspace configuration to list writable possible destinations. This type of harvester is, however, the only type that unprivileged users can currently create. 

At present these workflows can only be triggered by an API call so an external mechanism is needed to make them run at the correct times. This may change if further work is done on the ENS to allow workflows to be triggered on a schedule. 

#### 3.5.5 Data Streams and Data Access

Data access varies depending on the attributes listed in section 3.5.3, particularly order based vs direct access and adaptor-based vs direct retrieval. Only the first two options outlined here are currently used. 

##### 3.5.5.1 File-based Access with File-based Retrieval

This is the simplest data access scenario and the most likely one for open data. In this scenario, the user uses the Resource Catalogue to find a STAC Item and follows its STAC Asset links. These are HTTPS links which work directly. 

The data provider must ensure that all data is available at HTTPS URLs. These may be hosted inside or outside the hub. 

Account linking is not required or supported and data may be published either as platform published data or user-published data. 

Support for commercial data using this access method is not planned. 

##### 3.5.5.2 Order-based Access with Adaptor-based Retrieval

This is the method used for Planet and Airbus retrieval and is the most likely method for commercial data. It could be used for open data published using order-based interfaces, such as ECMWF’s ERA5 and other data in the Copernicus Climate Data Store. 

In order-based access scenarios, a user places an order for data using the data ordering UI or API. The order is placed against a particular STAC Item in the catalogue, but this STAC Item typically doesn’t represent specifically the data that will be returned. This is because the user will usually need to specify processing and subsetting options to reduce the size and cost of the data and to obtain the most convenient result. In any case, the initial STAC Item does not contain asset links and cannot be directly used. For these reasons, the order generates a new STAC Item which is written to the commercial-data sub-Catalog in the ordering workspace’s catalogue. 

This STAC Item uses the STAC Order extension to initially identify that the order is in progress. The user must then poll this Item in the catalogue API. Once the order completes the assets are written to the workspace object store and the STAC Item is updated with a new status and with the assets. 

The user must then wait for order completion by polling. Once the order is complete, the ordered data is written into a workspace store belonging to the user and a catalogue entry pointing to it is created inside the workspace catalogue (in a dedicated sub-Catalog for this purpose). 

See section 3.9.4 Adaptors for information on how order-based retrieval is implemented. 

This scenario is not available for data published from the hub and is not currently expected for user-published data. 

##### 3.5.5.3 Order-based Access with File-based Retrieval (future)

In this scenario, the user must place an order for data but retrieval from the upstream source uses simple URL-following. This is not currently needed but could be used if a data stream requires that only a limited number of retrievals are in progress at a time. 

##### 3.5.5.4 File-based Access with Adaptor-based Retrieval (future)

This scenario is not currently planned but could be supported in the future if a suitable dataset is added to EODH. From a user’s perspective, the user follows an Asset link from a STAC Item and the required data is returned – although some functionality such as range requests may not be available and latency may be higher. These requests are directed to a special EODH data access service which invokes an Adaptor, returning data on completion. 

This scenario is only useful where the data stream can assure EODH that retrievals are low-latency. To provide good performance, the service may need a caching layer and it may be necessary to allow for persistently-running Adaptors (which the User Account Service calls directly) or to integrate Adaptors into the data access service. 

#### 3.5.6 Future Evolution

Should Workflow-startup latency become problematic, for example when supporting interactive applications, persistently running adaptors which listen directly for retrieval messages may be required.
