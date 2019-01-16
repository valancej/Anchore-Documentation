## Anchore Enterprise Planning Guide

## Table of Contents

<!--ts-->
  * [Introduction](#Introduction)
  * [Anchore Overview](#Anchore-Overview)
  * [Anchore Components](#Anchore-Components)
    * [Enterprise Services](#Enterprise-Services)
    * [Enterprise Feeds Service](#Enterprise-Feeds-Service)
    * [Registry communication](#Registry-communication)
  * [How is Anchore typically used?](#How-is-Anchore-typically-used?)
  * [Image analysis lifecycle](#Image-analysis-lifecycle)
  * [Capacity planning](#Capacity-planning)
  * [Troubleshooting](#Troubleshooting)

<!--te-->

## Introduction

The intent of this document is to provide an overview and planning guide in preparation for an installation of Anchore Enterprise. Following reading this guide, Anchore operators should understand the following and be capable of proceeding with a successful install:

- Anchore Overview
- Anchore Components
- Where Anchore will be run?
- What Anchore needs to communicate with?
- Common Anchore points of integration.
- Anchore capacity planning.

## Anchore Overview

Anchore is a Docker container image static analysis and policy-based compliance tool that automates the inspection, analysis, and evaluation of images against user-defined checks to allow software development teams to deploy containers with a high level of confidence by ensuring the workload content meets the required criteria. 

### How is Anchore delivered? 

Anchore is container-native and is built and delivered as a Docker container. You can view Anchore's dockerhub here: https://hub.docker.com/u/anchore.

### What Anchore contains? 

Anchore is a collection of services that can be deployed co-located or fully distributed or anything in-between, as such it can be scaled out horizonally to increase analysis throughput. The only external system required is a PostgreSQL database (version 9.6 or higher) that all services will connect to. 

### How Anchore works?

1. Fetch the Docker image content and extract it, but never execute it.
2. Analyze the image by running a set of Anchore analyzers over the image content to extract and categorize as much metadata as possible. 
3. Save the resulting analysis data in the database for future use and audit.
4. Evaluate policies against the resulting analysis data, including vulnerability matches on the artifact discovered within the image. 
5. Update the latest external data used for policy evaluation and vulnerability matches, and automatically update image analysis results against any new data found upstream.
6. Notify users of changes to policy evaluations and vulnerability matches.
7. Repeat steps 5 & 6 on intervals to ensure latest external data and updated image evaluations. 

## Anchore Components? 

Below lists the components of Anchore Enterprise.

### Enterprise Services

#### Client Tier

- Enterprise UI
- Anchore CLI

#### API Tier

- External API
- Enterprise RBAC Manager API
- Enterprise RBAC Authorizer (internal)

#### State Tier

- Catalog
- SimpleQueue
- Policy Engine
- Enterprise Feeds Service

#### Worker Tier

- Analyzer

### Enterprise Feeds Service

Anchore Enterprise Feeds service is an on-premise service that supplies os and non-os vulnerability data and package data for consumption by Anchore Engine. 

Anchore Enterprise Feeds have three components: 

1. Drivers: Responsible to downloading raw data from external sources and normalizing it. 
2. Database: Normalized vulnerability and package data is persisted in the database. In addition, the execution state and updates to the data set are tracked in the database.
3. API: Anchore Enterprise Feeds exposes a RESTful API for interabtion with the service. The API layer serves the normalized data from the database based on the client requests. The Policy Engine service uses this API to sync the feed data down to the Anchore Engine database.

Anchore Enterprise Feeds require access to upstream data feeds from the following supported supported distributions and package registries over port 443:

| Host | Port | Description |
| :---- | :---- | :----------- |
| linux.oracle.com | 443 | Oracle Linux Security Feed |
| github.com | 443 | Alpine Linux Security Database |
| redhat.com | 443 | Red Hat Enterprise Linux Security Database |
| security-tracker.debain.org | 443 | Debian Security Feed |
| salsa.debian.org | 443 | Debian Security Feed |
| replicate.npmjs.com | 443 | NPM Registry Package Data |
| s3-us-west-2.amazonaws.com | 443 | Ruby Gems Data Feed |
| static.nvd.nist.gov | 443 | NVD Database |
| launchpad.net/ubuntu-cve-tracker | 443 | Ubuntu Data |
| data.anchore-enterprise.com | 443 | Snyk data |

### Anchore Databases and Persistence

#### Anchore Engine Database

Anchore is built around a single Postgresql database, using the default public schema namespace. This is the standard open-source installed db and contains tables for all necessary services. The services do not communicate through the db, only through explicit API calls, but the database tables are consolidated for easier management operations.

#### Anchore Enterprise Database

Anchore Enterprise has its own database tables and uses a separate anchore_enterprise schema namespace in the same postgresql database as the open-source installed tables. This schema has its own version tracking and upgrade mechanisms and includes the data for the RBAC systems as well as the Feed Service (if configured to use the same postgresql instance).

#### Feed Service Database 

The feed service uses the same namespace/schema as the other enterprise components but can be configured to use an entirely different database instance if desired in order to isolate performance and load. This is particularly useful for air-gapped installations. The feed service uses the common Enterprise db upgrade mechanisms, if you install and configure the feed service to use its own db instance you will still see all the enterprise tables present, but they will not be used in that specific case.

#### Redis

Redis is a requirement of the Enterprise UI and is used for session state and caching. It is currently not expected to be persisted, only served from memory since that data is small.

### Registry communication

Given that Anchore scans built Docker images a container registry is a hard requirement in order for Anchore to being analysis. The components of Anchore that need to communicate with the configured container registres are the Catalog and API service. Anchore can be instructed to download image from any Docker V2 compatible registry. Anchore will attempt to download images from any registry without requirement further configuration. For registries where username and password authorization is required, the registry must be added to Anchore prior to any image analysis. 


## How is Anchore typically used? 

Anchore Enterprise is typically deployed as a running service and is commonly used in the following ways:

- Iteractive Mode - Use the APIs to explicitly request an image analysis, get a policy evaluation and content reports, bu the engine only performs operations when specifically requested by a user or tooling. 
- Watch Mode - Use the APIs to configure Anchore to poll specific registries and repositories/tags to watch for new images added and automatically pull and evaluate them, emitting notifications when a given tag's vulnerability or policy evaluation state changes.

### CI/CD integration

- It is common to scan Docker images after they are built and prior to deployment to a production registry. In this case, a staging registry is used to push the built images, Anchore then fetches the images from the staging registry, conducts analysis, and depending on the result of the policy evaluation, users can choose to promote these scanned images to a production registry. 
- The build pipeline must be able to access the Anchore engine-api endpoint. 

## Image analysis lifecycle

Anchore image analysis is performed as a distinct, asynchronous, and scheduled task driven by queues that analyzer workers periodically poll. Image records have a small state-machine as follows:

![alt text](https://s3.amazonaws.com/cdn.freshdesk.com/data/helpdesk/attachments/production/36021354861/original/YSqCPgSG77OXMHOjx7P2Do9Bld4Zar5gVw.jpg?1542049669)

The analysis process is composed of several steps and utilizes several system components. The, basic flow of that task is as follows:

![alt text](https://s3.amazonaws.com/cdn.freshdesk.com/data/helpdesk/attachments/production/36020808611/original/9W4uwUyT5suYarn83yOGsWPynNnxtPJKdw.jpg?1541503856)

## Capacity planning

Before allocating designated resources for Anchore, users should ask themselves the following questions:

1. What are the total number of images per day they will likely be analyzing with Anchore?
2. What are the total number of analyzed images they would like to keep within Anchore? Is an audit trail of older images important? Is there an interval of time where analyzed image data can be thrown out? 
3. What is the largest image that Anchore will be conducting analysis on?

*Load characteristics will vary by size of images, contents within the images, system throughput needs, policy evaluations, and user API interactions.*

### Component performance attributes

DB | Primary Resource Consumption | Scaling Metric | Recommended Resources (in AWS) |
| :---- | :---- | :----------- | :----------------------------- |
| External API | CPU (+ Mem for large responses) | Concurrent User API load | m4.large |
| Catalog | CPU & Network IO | User API load and Image Analysis load | m4.xlarge |
| Policy Engine | CPU + Memory | Image loads | c5.xlarge-4xlarge |
| Simple Queue | CPU | Number of workers | t2.medium |
| Analysis Worker | CPU + Scratch Disk IO | Image analysis queue | t2.medium/large |
| Feed Service | IO (+ Mem) | N/A | m5.large |
### Database recommendations

 DB | Purpose | Scaling Metric | Storage | Recommended Resources (in AWS) |
| :---- | :---- | :----------- | :------ | :----------------------------- |
| Engine DB | Backs all engine services | API Load and Image ingress load | 50-100GB initially ~50MB per image analyzed (avg) | db.r4.large or xlarge |
| Feed Service DB | Feed data | API Load | 20GB (grows with feed data size, slowly) | db.t2.large |
| Feed Service RubygemsDB | Scratch space for a Rubygems driver | None | 10GB | local postgresSQL container (ephemeral), 4GB Memory reserved |

## Troubleshooting

Troubleshooting Anchore is typically done by interactions with the Anchore CLI or by executing into a container and looking at the logs of particular services. 