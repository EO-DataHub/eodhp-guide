---
title: "ADR-002: Bulk download of item assets in the browser"
doc_status: unreviewed
tags:
  - rc-ui
  - stac
  - workspaces
last_reviewed:
reviewed_by:
review_notes: "Written alongside eodhp-rc-ui PR #621, which implements it."
---
# ADR-002: Bulk download of item assets in the browser

**Status:** Accepted. Implemented in [eodhp-rc-ui#621](https://github.com/EO-DataHub/eodhp-rc-ui/pull/621), in review.
**Date:** 2026-10-21
**Applies to:** the Resource Catalogue UI (`eodhp-rc-ui`). No backend service changes.

## Summary

Users can download many files of one catalogue item in one action. The browser downloads each file separately. No zip is built and no new backend service is added. For large or repeated downloads, the page also gives a ready-made Python or command-line script, and suggests a browser download manager.

This solves the reported problem now, at no running cost, and leaves a backend bulk-download service open as a later option.

## Context

A catalogue item can have many files. An Airbus SAR scene, for example, has 30 to 50. Before this change, users downloaded them one at a time from the Assets tab, which is slow and easy to get wrong. The request is tracked in [platform-bugs#205](https://github.com/EO-DataHub/platform-bugs/issues/205).

A "bulk download service" is also listed as a possible phase 3/4 architecture change in [eodhp-sparkgeo-pm#180](https://github.com/EO-DataHub/eodhp-sparkgeo-pm/issues/180). This ADR records why the immediate problem was solved without one.

Three facts shape the design:

- **Files are large.** Earth observation files are often hundreds of megabytes, and some are several gigabytes.
- **Some files are private.** Purchased data and workflow outputs sit in a user's workspace, behind a login. Public data needs no login.
- **The links already exist.** Every file in an item already has a direct download link that works today, so no new storage or API is needed to reach it.

## Options considered

| Option | Outcome | Reason |
|---|---|---|
| A backend service that builds a zip of the item | Not chosen | It needs a new service that reads private workspace data, builds large archives, and stores or streams them. That adds compute, storage and data-transfer cost, plus a new component to secure and operate. |
| Build a zip in the browser | Not chosen | The browser must hold every file in memory before it can save the zip. For the items that need bulk download most, this runs out of memory |
| Download each file separately in the browser, plus scripts and a download-manager tip | **Chosen** | It uses the existing links and logins. Each file goes to disk on its own. Scripts cover the cases the browser handles badly |

## Decision

**The browser downloads each selected file separately.** The user ticks the files they want and clicks **Download all** or **Download selected**. Public files start one after another, half a second apart, so the browser does not drop them.

**Private files are fetched with the user's workspace login, one at a time.** A private file's link leads to a login page, and the browser cannot follow that login in the background. The page fetches these files with the user's workspace token, holds each one in memory, and saves it. Files are fetched one at a time.

**The workspace token goes only to DataHub servers.** Some items link to files on other organisations' servers, for example a supplier's preview image. Those files are downloaded without the token, so the user's login never leaves the platform. The generated scripts follow the same rule.

**Every item gets a linkable download page.** The page is at `/download/<collection>?item=<id>`. Purchased data, saved sets and workflow results link to it, and the Data Finder shows the same view in a new **Download** tab. Users can share or bookmark the link.

**Scripts are the route for large or repeated downloads.** The page writes a Python script (using the `pyeodh` library) and a command-line script (using `curl`) for the selected files. For private items, the user pastes in their own API token.

## Excluded from this phase

| Excluded | Consequence |
|---|---|
| A single zip file | Users get one download per file. A download manager or script gathers them into one folder |
| Downloading several items at once, for example a whole saved set | Users open the download page for each item, or run a script per item |
| A backend download service | No server-side progress, retries or resumable downloads. The scripts and download managers provide these |

## Consequences

**What this makes possible.** Users can download a whole item in one action, from any page where they find it. There is no new service to run, monitor or pay for. Usage is measured with three new Google Analytics events (`view_item_downloads`, `download_all_item_imagery`, `copy_download_script`), which will show whether a backend service is worth building.

**What it costs.** The browser is in control, and browsers treat many downloads from one page with suspicion. Chrome asks the user to allow multiple downloads after the first file, and may block some. Some browsers open images and metadata files instead of saving them. The page warns users about this and points them to the scripts and to the [DownThemAll!](https://www.downthemall.org/) browser extension.

**What it does not do.** It does not make very large private files practical in the browser. Each private file passes through browser memory, so a multi-gigabyte private file can fail. The page tells users to use the scripts for these.

If a backend service is built later, the download page is where a "Download as zip" option would go. Nothing in this design needs to be removed to add it.

## Risks

**1. Browsers may tighten their rules on multiple downloads.** The **Download all** button depends on browsers allowing several downloads from one page once the user agrees. If a browser stops allowing this, the button stops working for more than one file. The scripts and download managers are not affected.

**2. A click before the workspace token loads sends private files the wrong way.** If the user clicks **Download all** in the moment before their workspace token has loaded, private files are sent through the public route and fail. This was left as it is on purpose.

## Where the detail lives

- [eodhp-rc-ui#621](https://github.com/EO-DataHub/eodhp-rc-ui/pull/621): the implementation, with notes for reviewers
- [platform-bugs#205](https://github.com/EO-DataHub/platform-bugs/issues/205): the original request
- [eodhp-sparkgeo-pm#180](https://github.com/EO-DataHub/eodhp-sparkgeo-pm/issues/180): the possible phase 3/4 bulk download service
