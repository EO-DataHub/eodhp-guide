---
title: Pulsar Messages
doc_status: needs-verification
last_reviewed:
reviewed_by:
review_notes:
tags:
  - pulsar
  - needs-verification
---
# Pulsar Messages

Python definitions for some messages are present in eodhp-utils, in `eodhp_utils/pulsar/messages.py`. These are suitable for use with Pulsar Schemas.

Topic URLs all begin `persistent://public/default/` followed by the name given in here.

## Accounting (topic `billing-events`)

A schema is enforced on this topic.

The Python class `eodhp_utils.pulsar.messages.BillingEvent` is the authoritative definition.

| Field                    | Type                                       | Definition                                                                                                                                                                         |
| ------------------------ | ------------------------------------------ | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| uuid                     | string (UUID)                              | Unique ID for the event                                                                                                                                                            |
| event_start              | string (ISO datetime, always UTC)          | Billing events cover a period of time and this is the start of this period. Where appropriate, event_start and event_end may be equal. The period should never cross midnight UTC. |
| event_end                | string (ISO datetime, always UTC)          | End time (see event_start)                                                                                                                                                         |
| sku (stock-keeping unit) | string                                     | A human-readable identifier, such as WFLOW-CPU, which identifies the BillingItem ('product') being billed for.                                                                     |
| user                     | string (UUID)                              | Where applicable, the UUID of the relevant user. This may be null, for example for the cost of workspace storage.                                                                  |
| workspace                | string                                     | A workspace name the charge will be applied to.                                                                                                                                    |
| quantity                 | double (schema type - stored as a decimal) | The quantity of the product consumed in the units defined in the BillingItem.                                                                                                      |

## Harvest Pipeline - Catalogue Change Messages (topics `harvested`, `harvested-annotations`, `harvested_bulk`, `harvested_stac`, `harvested_workflows`, `transformed`, `transformed-annotations`, `transformed_bulk`, `transformed_stac`)

Catalogue Change Messages follow a common format but the type of metadata being handled and the stage of processing varies by topic. Messages with the same stage of processing and type of metadata should be compatible with each other, regardless of which component generated them.

A harvest message doesn't contain the harvested metadata itself. Instead it contains only a reference to it in an S3 bucket. The locations in the bucket are significant and follow the following format:

`s3://<bucket>/<harvest-component-type>/<harvester-id>/<cat-path>`

for example

`s3://catalogue-population-eodhp-dev/stac-harvester/ceda-harvester5/catalogs/stac-fastapi/collections/cmip6.json`

Note that the layout in the bucket may have no relationship to the layout of files or URL paths in the original source, if such a thing even exists. Apart from the files referenced in messages, everything under the `s3://catalogue-population-eodhp-dev/<harvest-component-type>/` prefix is private to the named harvest pipeline component and may contain harvester state, caches, etc.

### Definitions

- Harvester type: the STAC harvester, the workflow harvester, the file harvester, the Git harvester, etc. Each is implemented by different code and services/jobs.
- Harvest component type: any type of harvest pipeline component including the harvester types, the transformer and all of the ingesters.
- Harvester or harvester instance: the configuration of (usually recurring) harvesting from a specific source for a specific target in the catalogue. eg, STAC harvesting from CEDA or the harvesting of the outputs of a particular workflow job.
- Harvest run or harvest: everything triggered by a single fetch of some amount of metadata by a harvester, eg the fetching of one set of updated from Airbus, a single re-fetch of the contents of a STAC catalogue, or the harvesting of all outputs from a workflow job during stag-out.
- Cat path: a location within the catalogue hierarchy. For STAC entries this corresponds to the end of the path in the EODH STAC APIs.

### Message Format

| Field                    | Type                                                                                                                                                                                                                                                                                                                                                                                                                   | Definition                                                                                                                                                                                                                                                                                                                                                                                                                       |
| ------------------------ | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| id                       | string                                                                                                                                                                                                                                                                                                                                                                                                                 | The harvester ID, which uniquely identifies the (potentially recurring) harvest process. eg, sh-default-workspace-api-stac-ceda-ac-uk. For workflows this is the job id                                                                                                                                                                                                                                                          |
| workspace                | string                                                                                                                                                                                                                                                                                                                                                                                                                 | The name of the workspace under whose identity and permissions the harvesting is being done. This should be given no other significance.                                                                                                                                                                                                                                                                                         |
| source_uri               | string (URI)                                                                                                                                                                                                                                                                                                                                                                                                           | (Not yet implemented - nneded for annotations) A URI which uniquely identifies the origin of the metadata. This need not be dereferenceable and it's not necessary possible to retrieve the source metadata a second time, although in many cases both with be true. This may be a Git URL pointing to a repo the metadata came from, an HTTPS URL pointing to a STAC API or static STAC catalogue, the URL of an ADES Job, etc. |
| bucket_name              | The S3 bucket used to store the metadata this message is about.                                                                                                                                                                                                                                                                                                                                                        |
| added_keys, updated_keys | Keys of objects in `bucket_name` which contain metadata known to exist in the source. The message only guarantees that, at the time of generation, this metadata existed in the source and nothing more. The distinction between added and updated metadata should be uesd as a hint only, and a piece of metadata may be new to one component (such as a recently added replica or ingester type) and not to another. |
| deleted_keys             | Keys of objects which are known not to exist in the original source and which no longer exist in S3. Receivers of such messages must use knowledge of the S3 URL format above to know what to delete. There is no guarantee that the catalogue entity being referenced actually exists and receivers should be prepared to see this twice.                                                                             |
| source                   | The root of the harvest in the upstream source. Only the catalogue entry identified and its descendents are being harvested. Commonly this will be `/`.                                                                                                                                                                                                                                                                |
| target                   | The location in the EODH catalogue hierarchy the harvest is targetting. Normally, the catalogue entity at `source` in the upstream catalogue becomes the entity at `target` in the EODH catalogue.                                                                                                                                                                                                                     |

### `harvested` stage

Topics beginning `harvested` relate to metadata that has been retrieved from its original source and has had little or no processing. It's our copy of the source material and can be used for re-transform and re-ingest even if the source is unavailable. This is stored into our catalogue population bucket with the cat-path part of the S3 key being the complete cat-path in the upstream source. This means that the cat-path part of these keys always begins with the `source` field in the message the harvester will send. Where there is no concept of a 'catalogue path' in the source, `source` should always be `/` and the objects can be layed out in any way which is most helpful to the transformer.

Outputs from this stage can be re-injected into the transformer for reprocessing, for example because the transformer or ingester code has changed or there is a new ingester instance.

### `transformed` stage

Topics beginning `transformed` relate to metadata that has been transformed into the correct form for EODH and has passed all access checks. This is stored into our catalogue population bucket with the cat-path part of the S3 key being the complete cat-path the entries have in EODH. This always begins with the value of the `target` field.

In many cases the EODH cat-path is the source cat-path with its `source` prefix stripped and replaced with `target`. This is not guaranteed and is not the case with annotations data.

### Metadata types

`harvested_stac` and `harvested_bulk` both contain messages pointing to STAC, with `_bulk` being used for lower-priority bulk harvests that would otherwise block the pipeline for a long period.

`harvested` contains other types of metadata such as access policies.

`harvested_workflows` contains workflow definitions defined in harvest sources (currently just Git).

`harvested-annotations` contains annotations with the NPL QA workflow outputs being the only supported format.
