# Installing Anchore Enterprise with Docker Compose

## Anchore Support

Join community Slack channel: https://anchore.com/slack

## Getting started

This document will detail the necessary requirements for installing Anchore Enteprise with Docker Compose. 

### Hardware requirements

The following details the minimum hardware requirements needed to run a single instance of all containers:

- 2 CPUs
- 8 GB RAM
- 50 GB disk space

**Increased CPUs and RAM is recommend for better performance**

### Docker requirements

Anchore Enterprise is delivered as a Docker container, so a Docker comptabile runtime is a requirement. 

Anchore Enterprise supports Docker runtime versions 1.12 or higher and Compose version 2.x.

### Operating System requirements

- Ubuntu 16.04x or higher
- CentOS 7.3 or higher
- RHEL 7.3 or higher
- Amazon Linux 2

### Software requirements

The Anchore Enterprise UI is a web application with an HTML interface. Accessing the user interface is done via a web browser.

- Chrome
- Firefox
- Safari

### Network requirements

Anchore Enterprise requires the following ports to be externally accessible:

- 

**Air-gapped installs will differ**

### Database requirements

Anchore Enterprise uses PostgreSQL object-relation database to store data. Before beginning install, determine whether you will be using the PostgreSQL database container that is automatically install or an external PostgreSQL instance. 

**See configuring external DB instance for more info**

### PostgreSQL versions

The PostgreSQL container that is automatically installed with Anchore Enterprise is postgres:9. 

Anchore Enterprise supports PostgeSQL version 9 or higher