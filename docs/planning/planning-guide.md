## Anchore Planning Guide

## Table of Contents

<!--ts-->
  * [Introduction](#Introduction)
  * [Anchore Overview and Concepts](#Anchore-Overview-and-Concepts)

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

Anchore is a collection of services that can be deployed co-located or fulled distributed or anything in-between, as such it can be scaled out horizonally to increase analysis throughput. The only external system required is a PostgreSQL database (version 9.6 or higher) that all services will connect to. 

### How Anchore works?

1. Fetch the Docker image content and extract it, but never execute it.
2. Analyze the image by running a set of Anchore analyzers over the image content to extract and categorize as much metadata as possible. 
3. Save the resulting analysis data in the database for future use and audit.
4. Evaluate policies against the resulting analysis data, including vulnerability matches on the artifact discovered within the image. 
5. Update the latest external data used for policy evaluation and vulnerability matches, and automatically update image analysis results against any new data found upstream.
6. Notify users of changes to policy evaluations and vulnerability matches.
7. Repeat steps 5 & 6 on intervals to ensure latest external data and updated image evaluations. 

