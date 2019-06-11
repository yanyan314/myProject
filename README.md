# myProject
# Content-Base micro-service

[![Build Status](https://connjenk.swg.usma.ibm.com/jenkins/buildStatus/icon?job=connections/content-base/master)](https://connjenk.swg.usma.ibm.com/jenkins/view/CNext/job/connections/job/content-base/job/master/)
[![Unit Test Coverage](https://connjenk.swg.usma.ibm.com/jenkins/job/connections/job/content-base/job/master/lastSuccessfulBuild/artifact/reports/coverage/coverage.svg)](https://connjenk.swg.usma.ibm.com/jenkins/job/connections/job/content-base/job/master/lastSuccessfulBuild/artifact/reports/coverage/lcov-report/index.html)
[![code style: prettier](https://img.shields.io/badge/code_style-prettier-ff69b4.svg)](https://github.com/prettier/prettier)

Base micro-service is a core part of content service, which stores data about a content item, including basic meta-data, ACL, versions, and extensible attributes.

## Project structure

1. `src` folder with source code.

2. `tests` folder with test code.

   * `tests/unit` folder with unit tests.
   * `tests/live` folder with tests which requires the service to be live, e.g. connecting database, Redis, etc.
   * `targets` - folder with all test reporting assets (unit and coverage html reports).

3. `deployment` folder with documents to deploy the service to kubernetes.

4. `scripts` folder with basic bash script that a common pipeline will call.

## Getting started

* NVM (Node Version Manager) for your OS https://github.com/creationix/nvm and use NodeJS 8 LTS
* Latest `docker-engine` https://docs.docker.com/engine/installation/
* IDE of choice (VSCode)
  * Ensure you have eslint linter plugin installed to see potential errors inline as you code.
* Ensure you have permission to https://artifactory.swg.usma.ibm.com/artifactory

### Development Mode

#### Install node pacakages

* Create .npmrc from .npmrc.template. Set registry to be v-ess-npm-dev
* Run `npm install`. Some modules may not be found from v-ess-npm-dev. You can install them individually by `npm install <module-name> --registry https://registry.npmjs.org`.
* For some modules which are not found in registry.npmjs.org, you can search https://artifactory.swg.usma.ibm.com/artifactory and use the vitural repository name to replace v-ess-npm-dev, e.g. 'https://artifactory.swg.usma.ibm.com/artifactory/api/npm/v-ess-npm'

#### Install Mongo and Redis ( run as a container in Docker )

##### Install Docker

* Install docker following guides on https://www.docker.com/.

##### Install Mongo

* Run script `docs/development/dockerRunMongo.sh` to pull and run mongo as docker container.

##### Install Redis

* Run script `docs/development/dockerRunRedis.sh` to pull and run redis as docker container.

#### Launch service

* Run `npm start` to launch this service.

### Production Mode

* `npm run build` transpiles JS and puts them to /lib folder
* `npm run start-prod` runs this service in production mode

Go to browser and hit `http://localhost:3000` to see your running loopback service - For more information about loopback, check https://loopback.io/doc/en/lb2/Getting-started-with-LoopBack.html

### Other targets

* `npm test` runs unit tests in `tests/unit`. Coverage reports are created in `target/tests/unit`.
* `npm run lint` generates lint report.
* `npm test:live` runs live tests in `tests/live`. Coverage reports are created in `target/tests/live`.

### Docker Image

#### Build

`docker build -t content-base .`

#### Run

Run script `docs/development/dockerRunMe.sh` to pull and run this service as docker container.

Note that

* If you want to run your locally built docker image, change `IMAGE` in the script.
* By default, all containers are running in User Defined Network, i.e. 'connections-dev'. If it doesn't work for you, e.g. can not connect to another container due to your special network, try to comment out `NETWORK=connections-dev`.

### Configurations

Configurations are passed as environment variables. Below is a table of environment vavirables used by this service.

| Environmental Variable          | Required | Description                                                                                                                 |
| ------------------------------- | :------: | --------------------------------------------------------------------------------------------------------------------------- |
| DB_HOST                         |    Y     | Hostname of database service. It can be name of database container if all services are running in docker's private network. |
| DB_PORT                         |    Y     | Port of database service                                                                                                    |
| INTER*SERVICE_CHANNEL*{service} |    N     | Name of channel provided by {service} for inter-service API call, e.g. `content-share-rpc`.                                 |
| EVENTS*QUEUE*{service}          |    N     | Name of events queue provided by {service}, e.g. `connections.events.content-share`.                                        |
| TEST_SUITE                      |    N     | Used by testing. Specify what suit to be run, e.g. `live`                                                                   |
| TEST_REPORT                     |    N     | Used by testing. Specify root folder for test report, e.g. `target/tests`                                                   |
| TEST_PATH                       |    N     | Used by testing. Specify root folder containing test suites, e.g. `tests`                                                   |
| TEST_FILES                      |    N     | Used by testing. Specify pattern of test files, e.g. `*.spec.js`                                                            |

### APIs

| Type          | Endpoint/Channel                             | Description                                                                                                                                       |
| ------------- | -------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------- |
| Rest          | /api/v1/                                     | Rest API for resources, e.g. `GET /api/v1/entities`                                                                                               |
| Inter-service | content-base-rpc                             | Used by other micro-services                                                                                                                      |
| Events        | <nobr>connections.events.content-base</nobr> | Events queue where this service publish events. (`connections.events-internal.content-base` is for internal events. But this might not be needed) |

### Connections development guides

* https://pages.github.ibm.com/connections/connections-planning/development/

### Setting for VSCode

#### Configure editor

* `editor.tabSize": 2` (Go to Code->Preferences->Settings)
* Install ESLint extension

#### Enable debugging

Add "runtimeExecutable" to .vscode/launch.json as below

```
  {
    "type": "node",
    "request": "launch",
    "name": "Launch Program",
    "protocol": "inspector",
    "program": "${workspaceRoot}/src/server/server.js",
    "runtimeExecutable": "${workspaceRoot}/node_modules/.bin/babel-node"
  },
```

Switch to Debug view and click F5.

### Architecture

* System context of pink content platform
  ![System context of pink content platform](docs/design/content-pink-system-context.png)

* Containers of pink content platform
  ![Containers of pink content platform](docs/design/content-pink-containers.png)

* Components of content base service
  ![Components of content base service](docs/design/components.png)

### System Design

#### Interservice API

Basic framework is setup via [Setup interservice communication between content-base and content-share](https://github.ibm.com/connections/content-base/issues/23). Need following up work

* Finalize API signature w/ other pink services

#### Events publishing

Basic framework is setup via [Micro-service need be able to publish events](https://github.ibm.com/connections/content-base/issues/22). But Connections Pink is using Redis Lists, (see details in issue 16). The following work is

* Choose between Redis List or BusMQ or RSMQ
* Implement message payload following Activity Stream spec
* For events consumer, how to avoid to get duplicated old events(e.g. to refactor this `queue.consume({remove: false, index:0})`

## Investigation stories

* Asynchronously compute to keep data eventually consistent

  * [Investigation 3 - an event-driven pattern to be used in micro-service](https://github.ibm.com/connections/content-base/issues/219)

* Enforce unique constraint

  * [Investigation II - enforce uniqueness by application](https://github.ibm.com/connections/content-base/issues/207)

* Trash

  * [Investigation - Trash](https://github.ibm.com/connections/content-base/issues/287)

* Search
  * [Investigation - building content-search micro-service using elastic search](https://github.ibm.com/connections/content-base/issues/276)
    ### References
* [Use of Pub Sub for Object Retrieval](https://github.ibm.com/connections/connections-planning/blob/master/docs/AD1-pubsub/PubSubForObjectsRetrieval.md)
* [Pink Middleware Design](https://apps.na.collabserv.com/communities/service/html/communityview?communityUuid=71676fab-07a6-420a-97c5-600359a8b773#fullpageWidgetId=W25b2035e7d79_4557_ae9e_e48296475d91&file=c77b25ed-ad4e-4487-8068-65d45bff64e0.)
