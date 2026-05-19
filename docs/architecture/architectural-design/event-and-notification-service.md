### 3.11 Event and Notification Service (ENS)

Argo Events and Argo Workflows are installed as part of the incomplete ENS. Using Kubernetes custom resources, event sources and sensors can be configured which receive event triggers and trigger downstream actions. The possible event sources include GitHub and GitLab webhooks, Pulsar messages and calendar events. The actions that can be triggered include Argo Workflows and Kubernetes Jobs. 

Currently, event sources are used to detect changes to Git repos containing harvestable files, and to trigger on a daily schedule. The sensors attached to these are used to trigger Jobs containing harvest pipeline harvesters. This is used for harvesting the Airbus and 

CEDA catalogues each day, and for harvesting platform workflow definitions from a Git repository. 

This is intended to be extensible in various ways. 

Firstly, support for Pulsar messages could be used to receive catalogue change messages and trigger downstream actions, particularly to run workflows. A user might configure this in a UI or with a harvestable file so that a workflow is run on all matching new data. 

Secondly, the Git support could be used to allow users to link workspaces to their Git repositories and trigger actions when changes are made. This could include the harvesting of catalogue files stored there, like STAC, and the auto-deployment of workflow definitions. 

Finally, it could be used to trigger harvesting from other sources such as upstream STAC APIs, using user-configured settings. This would allow users to manage their own harvesters to import data from other STAC catalogues. These users could include EODH system operators and data providers. Argo Workflows provides a UI to Argo Events which could be used to monitor the state of this harvesting without needing to use more general tools like kubectl and Kibana. 

