## Anchore Enterprise Planning Guide

## Table of Contents

<!--ts-->
  * [Introduction](#Introduction)
  * [Anchore Overview and Concepts](#Anchore-Overview-and-Concepts)
  * [Anchore Components](#Anchore-Components)
    * [Enterprise Services](#Enterprise-Services)
    * [Enterprise Feeds Service](#Enterprise-Feeds-Service)
  * [Capacity planning](#Capacity-planning)

<!--te-->

## Introduction

The intent of this document is to provide an overview and planning guide in preparation for an installation of Anchore Enterprise. Following reading this guide, Anchore operators should understand the following and be capable of proceeding with a successful install:

- Core components of Anchore and concepts.
- Where Anchore will be run?
- What Anchore needs to communicate with?
- Common Anchore points of integration.
- Anchore capacity planning.

## Anchore Overview and Concepts

Anchore is a Docker container image static analysis and policy-based compliance tool that automates the inspection, analysis, and evaluation of images against user-defined checks to allow software development teams to deploy containers with a high level of confidence by ensuring the workload content meets the required criteria. 

### How is Anchore delivered? 

Anchore is container-native and is built and delivered as a Docker container. You can view Anchore's dockerhub here: https://hub.docker.com/u/anchore.

### What is inside? 

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

Below lists the Anchore Enterprise services.

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

### What is required for image analysis?

Given that Anchore scans built Docker images a container registry is a hard requirement in order for Anchore to being analysis. The components of Anchore that need to communicate with the configured container registres are the Catalog and API service. Anchore can be instructed to download image from any Docker V2 compatible registry. Anchore will attempt to download images from any registry without requirement further configuration. For registries where username and password authorization is required, the registry must be added to Anchore prior to any image analysis. 

## Capacity planning