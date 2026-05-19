# Workflow Output Processing

This proposes a way to map from the STAC records produced by workflows to records created in the catalogue in a way which allows simple workflows not to care about catalogue structure and more complex ones to control where exactly the output will go. Note that this proposal is not currently fully implemented.

OGC Application Packages are required to produce STAC outputs along with data files. EODHP saves the data files into a workspace store and the STAC into the resource catalogue, usually to the workspace's catalogue. Some workspaces may be able to write to other areas in the catalogue, such as 'official' workspaces being used to produce data which is being published as hub-curated datasets.

An access check always occurs so that only authorized workspaces can write to areas outside their own workspace catalogue. Output processing uses the credentials of the calling workspace. This means that output files are written to the caller's workspace stores and output STAC is written to the caller's workspace catalogue, or another area of the catalogue that the caller is authorized to write to.

## STAC Records Written to the Catalogue

Exactly what STAC is written and where in the catalogue it is written to depend on what the workflow produces. Where STAC is written to a location in the catalogue which doesn't exist - for example to `catalogs/users/catalogs/my-workspace/catalogs/non-existent-catalog/collections/workflow-output-collection` - intermediate Catalogs will be automatically generated.

### Workflow produces files and no STAC

Whilst this is technically a breach of the requirements on OGC Application Packages, to provide a better experience to users EODHP generates STAC in this case. The resulting structure is:

- Catalog: user-datasets
  - Catalog: `my-workspace`
    - Catalog: `Workflow Outputs`
      - Catalog: `<workflow name>`
        - Collection: `Run <id>`
          - Item: `Output Files`. Each output file is an asset.

### Workflow produces a top-level Catalog, a Collection or an Item with no parent link

This includes the case of the workflow producing a Catalog or Collection with other STAC entities inside it (and so with parent links). This case applies when the top-level of the STAC returned has no parent.

In the case of a Catalog, the resulting structure is:

- Catalog: user-datasets
  - Catalog: `my-workspace`
    - Catalog: `Workflow Outputs`
      - Catalog: the Catalog produced by the workflow

for a Collection:

- Catalog: user-datasets
  - Catalog: `my-workspace`
    - Catalog: `Workflow Outputs`
      - Catalog: `<workflow name>`
        - Collection: the Collection produced by the workflow

and for an Item

- Catalog: user-datasets
  - Catalog: `my-workspace`
    - Catalog: `Workflow Outputs`
      - Catalog: `<workflow name>`
        - Collection: `Run <id>`
          - Item: the Item produced by the workflow

If the `id` of the Catalog, Collection or Item matches matches another one in its containing entity then it overwrites that entry. If no `id` is given then we generate one based on the workflow job ID.

If a Catalog or Collection from the workflow is overwriting an existing entity then existing children are normally preserved.

TODO: Is there any sensible way to specify when existing children should be removed?

### Workflow produces a top-level Catalog, a Collection or an Item with a parent link

The parent link must identify a location in the catalogue (a Catalog or Collection) which the workflow's caller can write to. For example, if it's this `https://eodatahub.org.uk/api/stac/catalogs/public/catalogs/eodh-hosted/` then this identifies an `eodh-hosted` Catalog in the `public` Catalog, with the `eodh-hosted` Catalog being auto-created if it doesn't exist. Normal workspaces can only write in their workspace catalogue but special-purpose ones may be given greater privileges.

The resulting Catalog, Collection or Item is inserted as a child of the specified parent. If its `id` matches an existing child of that parent then it overwrites that entry. If no `id` is given then we generate one based on the workflow job ID.

### Use of the STAC Version extension

If the STAC Version extension is used and the `version` property is set then the processing is slightly different. The following will happen:

- The entry will be added to the catalogue under the id `<id>-<version>`, overwriting any existing entry if the version number has been reused. If no overwriting occurred then a `rel=predecessor-version` link is added pointing to the previous version if one existed (this link can be found as the link with `rel=latest-version` in the pre-update version of the file created/updated in the next step). A `rel=latest-version` link is added pointing to itself. If the version number was reused then the `predecessor-version`, `successor-version` and `latest-version` are copied from the pre-update copy of the entry.
- The entry will also be copied as an entry under the id `<id>` . The `latest-version` link still points to the `<id>-<version>` entry.
- The old entry has its `latest-version` and `successor-version` links updated.
