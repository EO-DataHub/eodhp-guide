---
title: Documentation Needs
doc_status: unreviewed
last_reviewed:
reviewed_by:
review_notes:
---
# Documentation Needs

This analyses the documentation that users may need about the system and proposes particular documentation to create.

These documentation needs have been identified through a mix of ad-hoc methods, experience and analysing the user journeys in the User Journey Schedule section of the Architecture Design Document.

These are broken down by documentation type.

## Platform Discovery

Hosting location: CMS

This is content users see on the front page or on pages which are likely search engine results or external links for new users.

Half of typical website visitors view one page and leave, time-on-page across _all_ pages is less than a minute, and we should strive to demonstrate clear relevance to target users within seconds.

We can use:

- The front page. This needs to work for all types of target user.
- Pages to help users quickly determine the capabilities of the hub linked-to from UI elements in the content of the front page.
- Pages we hope search engines will return. These can be more specific to user types.
- Pages we actively promote via links. These users have already heard of the hub and will be prepared to spend more time.
- Case studies as 'social proof' and for less technical managers who will not use the hub directly.

## Description of EODH

Hosting location: CMS
Web location: 'About' header and sub-headers

This is documentation mostly of interest to people on the 'supply' side of our industry or who want to assess for themselves its sustainability, commercial interests, compliance, etc.

## Tutorials, introductions and task-specific guides

Hosting: Mostly GitHub Pages?

Here, 'tutorial' implies task-specific documentation which includes worked examples of simple tasks designed to build an understanding that can be applied to the user's own more advanced tasks. 'Guide' implies that the tasks covered are directly useful to the user with much less adaptation than a tutorial.

### Getting Started Guide

Some possible topics:

- Registration
- Workspaces
  - What are they?
  - Joining an existing workspace.
  - Link to Account Management Guide for creating a new workspace and adding users.
- Finding data, favouriting data, visualizing (supported) data, differences between finding EO, climate and commercial data, QA information. Signpost to catalogue search using notebooks / pyeodh / pystac.
- Accessing data (eg, understanding and accessing the different assets available, differences between EO, climate and commercial data). Signpost to information about accessing data using notebooks and QGIS.
- Using your workspace stores
  - Differences between block and object stores.
  - Ways of uploading and downloading data.
  - Summary of what you can do with data in your workspace stores with links to other guides: access it from notebooks and workflows, make data public, add catalogue entries
- Using your workspace catalogue
  - Viewing it and what's in it
  - How entries are added
  - Publishing entries
- Processing data - purpose and audience of notebooks and workflows, links to their own guides, and links to example analyses.

### Account Management Guide

Some possible topics:

- What are workspaces and accounts?
- Creating an account and requesting a billing arrangement.
- Creating workspaces and managing user access.
- Paying your bill.
- How your bill is calculated with an example and a link to the pricing reference documentation.
- State how commerical data is billed and link to Commercial Data Setup Guide.
- Closing accounts and workspaces.

### Data Guide

Assistance on how to choose a suitable dataset.

### Data Publishing Tutorial

Data publishing can be used to publish data you create on the hub, that you upload to the hub or that you host elsewhere.

Some possible topics:

- Hosting data in the hub
  - Whether to add data or host it yourself. Description of the costs involved.
  - Adding data to object stores.
  - Making it public.
  - URLs for your data.
- Hosting metadata in the hub
  - How to add entries and where they appear in the catalogue.
  - Writing STAC.
- Hosting data and metadata elsewhere but making data available as a 'supported' hub dataset.
  - What might be considered?
  - Requirements on data and metadata.
  - Who to contact.

### Data Visualization Tutorial

Some possible topics:

- Visualizing data from the catalogue
  - How to visualize data.
  - Settings.
  - Using the APIs
- Visualizing your own data
  - Data requirements and preparing data for visualization

### Workflows Tutorial

Some possible topics:

- Calling workflows
- Writing workflows
- eoap-gen
- Publishing workflows
- User services

### EODH APIs Tutorial

Some possible topics:

- What the APIs can be used for (OGC APIs, finding data, running workflows, accessing workspace data, writing apps)
- Getting started
  - Getting an API key
  - Finding the documentation
- pyeodh tutorial
- Bare API tutorial

### App Development Tutorial

Some possible topics:

- What an app is and when you should create one
- Types of app (public apps - client-side web app, mobile app, desktop app; private apps - server-side web app or API)
- Integrating with EODH IdP
- Workflows and user services for apps
- Access control and billing

### Commercial Data Setup Guide

Some possible topics:

- Why link your account and how does this work?
- Signposting of how to get an account with Planet, Airbus Optical or Airbus Radar.
- How to get your Airbus account set up for use with the hub.
- How to link your hub account.
- How to unlink your hub account.
- Ordering and using data.

### Example analyses

## Reference

### Context help

Hosting: CMS

- Signup
  - Terms of use
  - What registering means (can apply to use the hub or join a workspace), how we use your information
- QA
  - Where our QA data comes from and any information about it NPL wants to make available (eg, how it's generated).
  - Detailed description of each field and its grades.

### Notebooks

Hosting: CMS

- Using alternative versions of Python
- Managing packages

### Hub APIs

Hosting: OpenAPI description and the Swagger viewer
Location: /api/api.html

Cover:

- /api/catalogue/api.html: The data, metadata and processing APIs - everything under /api/catalogue/, including STAC, TiTiler, annotations, ADES etc.
- /api/workspaces/api.html and /api/accounts/api.html: The workspaces and accounts APIs (not critical)
- /api/api.html: A landing point that shows the roots of the other APIs and links to them. Should also describe the workspace file access API.

### pyeodh API

Location: ReadTheDocs

### FAQs and/or Troubleshooting

Location: CMS

We should build these over time based on the difficulties users have in practice.

### Pricing

Hosting: CMS and/or a client-side app

We can't start this until we understand the pricing structure. This might be a simple table of items and prices.

### Data

### Commercial data and licences

This should mostly link to data providers' sites, but at least we will include the licence comparison table we already have.
