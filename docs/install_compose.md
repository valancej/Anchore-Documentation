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

Anchore Enterprise Feeds exposes a RESTful API by default on port 8228, however this port can be remapped. 

Anchore Enterprise Feeds require access to the upstream data feeds from the following supported distributions and package registries over port 443:

- linux.oracle.com (Oracle Linux Security Feed)
- github.com (Alpine Linux Security database)
- redhat.com (Red Hat Enterprise Linux security feed)
- security-tracker.debian.org (Debian security feed)
- salsa.debian.org (Debian security feed)
- replicate.npmjs.com (NPM Registry package data)
- s3-us-west-2.amazonaws.com (Ruby Gems data feed (stored in Amazon S3)
- static.nvd.nist.gov (NVD Database)
- launchpad.net/ubuntu-cve-tracker (Ubuntu data)
- data.anchore-enterprise.com (Snyk data)


**Air-gapped installs will differ.**

### Database requirements

Anchore Enterprise uses PostgreSQL object-relation database to store data. Before beginning install, determine whether you will be using the PostgreSQL database container that is automatically install or an external PostgreSQL instance. 

**See configuring external DB instance for more info.**

### PostgreSQL versions

The PostgreSQL container that is automatically installed with Anchore Enterprise is postgres:9. 

Anchore Enterprise supports PostgeSQL version 9 or higher

### Installation

- Approved Dockerhub username is required to pull Anchore Enterprise images.
- A valid Anchore Enterprise license.yaml file.

#### Step 1: Create installation location

Create a directory to store the configuration files and license file.

`mkdir ~/aevolume`

#### Step 2: Copy configuration files

Download the latest Anchore Enterprise container image which contains the necessary docker-compose and configuration files needed. In order to download the image, you'll need to login to docker using the dockerhub account that you provided to Anchore when you requested your license.

`docker login`

Enter username and password.

`docker pull docker.io/anchore/enterprise:latest`

Next, copy the included docker-compose.yaml file into the directory you created in step 1.
```
docker create --name ae docker.io/anchore/enterprise:latest

docker cp ae:/docker-compose.yaml ~/aevolume/docker-compose.yaml

docker rm ae
```

Next, copy the license.yaml file that provided into the directory you created in step 1.

`cp /path/to/your/license.yaml ~/aevolume/license.yaml`

Once these steps are completed, your Anchore directory workspace should look like the following:

`cd ~/aevolume`

```
find .
.
./docker-compose.yaml
./license.yaml
```



