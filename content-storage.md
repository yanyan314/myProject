# Content Storage micro-service 
[![Build Status](https://connjenk.swg.usma.ibm.com/jenkins/buildStatus/icon?job=connections-incubator/content-storage/master)](https://connjenk.swg.usma.ibm.com/jenkins/job/connections-incubator/job/content-storage/job/master/)

This is a micro-service providing functions to upload and download files. 

Right now, the service is using loopback storage component. Files can be uploaded to filesystem or supported object storage like AWS. 

## Documentation
Check https://pages.github.ibm.com/connections-incubator/content-storage/ for API documentation.

## Project structure
Refer to [Readme of content-base](https://github.ibm.com/connections-incubator/content-base/blob/master/README.md)

## Getting started
Refer to [Readme of content-base](https://github.ibm.com/connections-incubator/content-base/blob/master/README.md)

Note that 
* Make sure `/data/storage` has 777 permission.
* Go to browser and hit `http://localhost:3002/explorer` to see your running loopback service - For more information about loopback, check https://loopback.io/doc/en/lb2/Getting-started-with-LoopBack.html

### Docker Image

#### Build
`docker build -t content-storage .`

#### Run
Run script `docs/development/dockerRunMe.sh` to pull and run this service as docker container.

Note that 
* If you want to run your locally built docker image, change `IMAGE` in the script. 
* By default, all containers are running in User Defined Network, i.e. 'connections-dev'. If it doesn't work for you, e.g. can not connect to another container due to your special network, try to comment out `NETWORK=connections-dev`.
* By default, `filesystem` is used to store data. To use an object storage, e.g. AWS S3, change `STORAGE_PROVIDER` and set proper `STORAGE_KEY` and `STORAGE_KEY_ID`.

### Configurations

Configurations are passed as environment variables. Below is a table of environment vavirables used by this service.

|Environmental Variable | Required | Description |
|----------|:----------:|-------------|
|STORAGE_PROVIDER|Y|Specify what type of storage this service connects to, such as, STORAGE_PROVIDER='filesystem', STORAGE_PROVIDER='cleversafe'. If storage is cleversafe, STORAGE_KEY, STORAGE_KEY_ID, STORAGE_PROVIDER_ENDPOINT should be set as environment variable, too. You can pass any environment variables by ANY_ENV=any_value npm start'.|
|STORAGE_KEY|N|Key issued by storage. It is only required based on storage. If `STORAGE_PROVIDER=filesystem`, it is ignored|
|STORAGE_KEY_ID|N|Key ID issued by storage.|
|STORAGE_PROVIDER_ENDPOINT|N|Endpoint issued by storage. It is only required based on storage. If `STORAGE_PROVIDER=filesystem`, it is ignored |
|TEST_SUITE|N|Used by testing. Specify what suit to be run, e.g. `live`|
|TEST_REPORT|N|Used by testing. Specify root folder for test report, e.g. `target/tests`|
|TEST_PATH|N|Used by testing. Specify root folder containing test suites, e.g. `tests`|
|TEST_FILES|N|Used by testing. Specify pattern of test files, e.g. `*.spec.js`|

Note that
* All STORAGE_PROVIDER_* variables are applied to remote storage
* STORAGE_FS_ROOT is for local storage
* STORAGE_STAGED_STORAGE indicates whether to use staged storage
* STORAGE_MOCK_FS_ROOT is used for unit testing. `mockremotestore` is provider used only by unit testing.

This service need a data volume to be attached. In production env, the data volume is shared by containers of this service. Check EXTERNAL_DATA_DIRECTORY in `docs/development/dockerRunMe.sh`.

### Coding guides
Refer to [Readme of content-base](https://github.ibm.com/connections-incubator/content-base/blob/master/README.md)

### Setting for VSCode
Refer to [Readme of content-base](https://github.ibm.com/connections-incubator/content-base/blob/master/README.md)

### Architecture
* System context of pink content platform
![System context of pink content platform](https://pages.github.ibm.com/connections-incubator/content-base/design/content-pink-system-context.png)

* Containers of pink content platform
![Containers of pink content platform](https://pages.github.ibm.com/connections-incubator/content-base/design/content-pink-containers.png)

* Components of content storage service,
![content-storage components diagram](docs/architecture/components.png)

### System Design
#### Storage
* Storage service need filesystem(NAS) as default storage. By default, all contents are saved to filesystem first. 
* Storage service can be enabled to work with object storage. When it integrates with object storage, it can 
    1. Asynchronously uploade file from filesystem to object storage. Whether files on filesystem are kept depends on consuming applications and type of files
    2. Consuming applications may ask Storage service to upload directly to object storage by skipping filesystem. 
    3. To serve file download, Storage service can either redirect users to object storage, or serve it itself
* Need support huge file.

#### Upload Function
* Multipart file upload
* Multip-phase file upload
* Resummable file upload

#### Download Function
* Download single file
* Resummably file download
* Zip download? 

#### Configurable
* Different application may have different container, rendition, what files to be kept on disk, etc. 
* Global configuration for the service. 

#### Adaptablility
* Be able to connect to different object storage
* Be able to dynamically switch between different storages

#### Rendition
* Be able to generate renditions or to trigger conversion services
* Be able to link renditions with main files

#### Authentication and Authorization
* The service is not used directly by end users, except for download. 
* Need be able to authorize different applications. 
* Might be able to privide pre-signed download url. 

#### Interservice API
Refer to [Readme of content-base](https://github.ibm.com/connections-incubator/content-base/blob/master/README.md)
* Query content in a container? 
* Get meta-data of a container? 
* Create a container? 
* Delete a container? 
* Get meta-data of a file
* Get content of small file? 
* delete a file
* Create a small file? 

#### REST API
* Upload a file
* Download a file
* Delete a file
* Get meta-data of a file? 
* Get content of a container
* Create a container? 
* Update a container? 
* Delete a container? 

#### Events publishing
Refer to [Readme of content-base](https://github.ibm.com/connections-incubator/content-base/blob/master/README.md)

* Container CRUD
* File CRUD

## Investigation stories
* Multi-phase upload/download
  - [Investigation - how Storage service provide multiphase upload API](https://github.ibm.com/connections-incubator/content-storage/issues/125)
  - [Investigation - how to upload file content in non-form type](https://github.ibm.com/connections-incubator/content-storage/issues/197)
  - [Investigation/2 - to use non-form type request to upload file part](https://github.ibm.com/connections-incubator/content-storage/issues/286)
* Security
  - [Investigate to use S3 auth spec to sign and validate download url](https://github.ibm.com/connections-incubator/content-utils/issues/59)
* Events publishing
  - [Investitation - Documentation of events](https://github.ibm.com/connections-incubator/content-storage/issues/313)
  - [Investigation -busmq workflow and redis persistence](https://github.ibm.com/connections-incubator/content-base/issues/192)
* Error handling
  - [Investigation - how to handle errors in a central place](https://github.ibm.com/connections-incubator/content-storage/issues/296)
* Scheduled tasks
  - [Investigation - solution for prioritized job/task management](https://github.ibm.com/connections-incubator/content-conversion-proxy/issues/24)
  - [Investigation: How to make schedule task](https://github.ibm.com/connections-incubator/content-storage/issues/52)
* Image Conversion
  - [Investigation - choose a suitable nodejs module to create renditions for images.](https://github.ibm.com/connections-incubator/content-storage/issues/65)
* Storage
  - [Investigation - how to make Storage service to work with CleverSafe](https://github.ibm.com/connections-incubator/content-storage/issues/30)
* Security
  - [Investigation - solutions to secure REST API](https://github.ibm.com/connections-incubator/content-storage/issues/31)

## TODO
