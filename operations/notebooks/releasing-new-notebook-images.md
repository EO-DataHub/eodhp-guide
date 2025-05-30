# Releasing New Notebook Images

## Purpose

Over time it may be necessary to update the notebook images available in Jupyter Hub. This operation describes how to update or add new images to the Jupyter Hub profile list.

## When to Use

Use this procedure when you need to update an existing or add a new notebook image to the available Jupyter Hub images.

## Operation

The default image for Jupyter Hub notebook is version controller in https://github.com/EO-DataHub/eodh-jupyter-images GitHub repo. While it is possible to use stock images for Jupyter Hub profile images, it is suggested that this repo is used to version control any custom images for the EO DataHub.

This procedure assumes you are updating the default notebook image but many of the steps are relevant to using a stock image or adding a new custom image.

1. Checkout https://github.com/EO-DataHub/eodh-jupyter-images repo.
2. Make required modifications to default/Dockerfile (follow any required Git procedures for update, e.g. make changes on branch and create PR before merging to main).
3. Use default/Makefile to publish image `make publish`. Ensure you update the version in the Makefile for a stable release. For release candidates, recommended best practice is to suffix proposed release version with `-rcX`, where X is the release candidate number, e.g. `make publish version=python-3.12-1.2.3-rc1`.
4. Tag commit used to release image with tag. Since there are multiple images version controller in this repo, the tag should include the image name (the parent directory of the image, for clarity) and the version tag used, e.g. `default-python-3.12-1.2.3`.
5. Checkout https://github.com/EO-DataHub/eodhp-argocd-deployment repo.
6. In apps/jupyter/base/values.yaml, modify `hub.singleuser`.

   ```yaml
   hub:
     singleuser:
       image:
         name: public.ecr.aws/eodh/eodh-default-notebook
         tag: python-3.12-0.2.9
       profileList:
         - display_name: EO DataHub
           description: A Python 3.12 notebook with some added EO DataHub utils.
           default: true
         - display_name: Python 3.12
           description: Python 3.12 notebook.
           kubespawner_override:
             image: quay.io/jupyter/base-notebook:python-3.12
         - display_name: R 4.4
           description: R 4.4 notebook.
           kubespawner_override:
             image: quay.io/jupyter/r-notebook:r-4.4.2
   ```

   1. To update the default image, modify `hub.singleuser.image.name` (if changing the image) and `hub.singleuser.image.tag` (when updating the version). Make sure the description in `hub.singleuser.profileList[0]` is still correct, this is the metadata for the default image (indicated by `hub.singleuser.profileList[0].default=true`).
   2. To update an alternative image, or add a new image, modify or add the relevant profile item in `hub.singleuser.profileList`, specifying the `kubespawner_override.image` attribute to the desired image (this should specify the version as the image tag, e.g. `quay.io/jupyter/base-notebook:python-3.12`). Update any metadata in the profile item, as appropriate.

7. Once changes are complete, commit changes and push to remote. Follow usual Git procedures for merging. Once changes are merged to environment branch the config in ArgoCD will be update. However, Jupyter has been disabled from restarting on confgiuration changes to avoid unplanned restarts, which can cause connection issues for users with notebooks open. Jupyter Hub `hub` deployment must be manually restarted to pick up the configuration changes. This should be planned at a period of minimal user usage to avoid negative user experiences.
8. To restart Jupyter Hub `hub` deployment, use either ArgoCD UI or `kubectl`.
   1. Log into ArgoCD UI, restart `hub` deployment in `jupyter` app
   2. Or use `kubectl -n jupyter rollout restart deployment/hub` to restart via `kubectl`.
9. Test that changes have been successful by starting desired notebook from Jupyter Hub and checking image by executing `env | grep JUPYTER_IMAGE` in the notebook terminal. You should see the desired image printed to the terminal.

## Requirements

- Access to ArgoCD UI or `kubectl` access to cluster
- Write permission to https://github.com/EO-DataHub/eodhp-argocd-deployment GitHub repo
- [Optional] Write access to https://github.com/EO-DataHub/eodh-jupyter-images GitHub repo (required if updating custom images in this repo)

## Useful Information

- If creating a new notebook image, the recommended version format is `<base-image>-<image-version>`, e.g. `python-3.12-1.2.3`. `python-3.12` because the Dockerfile is based on stock image `quay.io/jupyter/base-notebook:python-3.12` and `1.2.3` because that is the latest version of the Dockerfile.
