# Eureka BPO ESB

Table of contents
- [Key features](#key-features)
- [Usage](#usage)
    + [eurekabpo/esb:1.0.0-mongodb](#eurekabpoesb100-mongodb)
    + [eurekabpo/esb:1.0.0-mysql](#eurekabpoesb100-mysql)
- [Configuration file](#configuration-file)
  * [Simple configuration](#simple-configuration)
  * [Full configuration](#full-configuration)
- [Request parameters](#request-parameters)
- [Response parameters](#response-parameters)
    + [Headers of immediately responses](#headers-of-immediately-responses)
    + [Headers of deferred responses](#headers-of-deferred-responses)

## Key features
Eureka BPO ESB is **message-oriented middleware** with **guarantee delivery** as a key feature. Functioning as **enterprise service bus** for **HTTP messages** Eureka BPO ESB provides a reliable intermediary between applications.

Here are several features of Eureka BPO ESB:
1. Guarantee of delivery incoming request to receiver and receiver's response to sender.
1. Full logging of requests, responses, exceptional situations.
1. Possibility to make queue for separate objects.
1. Possibility to manage state of requests "on the fly".
1. Flexible configuration at request, target system or global level.
1. Easy configuration and deployment.

## Usage
Application Eureka BPO ESB is distributed as docker image. Depending on database you want to use you can choose one of next releases:

- eurekabpo/esb:1.0.0-mongodb - uses MongoDB as storage
- eurekabpo/esb:1.0.0-mysql - uses MySQL as storage

#### eurekabpo/esb:1.0.0-mongodb

Uses MongoDB 4.4 as storage.

Environment variables for configuring:

Variable name | Required/optional | Description
--------------|-------------------|------------
MONGO_DATABASE | required | Database name
MONGO_HOST | required | host
MONGO_PORT | required | port
MONGO_USER | optional | username
MONGO_PASSWORD | optional | password

See [docker-compose.yml](docker-compose-mongodb.yml) to configure Eureka BPO ESB with MongoDB.

#### eurekabpo/esb:1.0.0-mysql

Uses MySQL 8.0 as storage

Environment variables for configuring:

Variable name | Required/optional | Description
--------------|-------------------|------------
MYSQL_DATABASE | required | Database name
MYSQL_HOST | required | host
MYSQL_PORT | required | port
MYSQL_USER | optional | username
MYSQL_PASSWORD | optional | password

See [docker-compose.yml](docker-compose-mysql.yml) to configure Eureka BPO ESB with MySQL.

## Configuration file

Configuration of Eureka BPO ESB stored in separate [YAML](https://en.wikipedia.org/wiki/YAML) file

### Simple configuration
```yaml
---
Target systems:
  system-one:
    baseUrl: http://www.example.com/test
```

Description

 Configuration parameter | Description
-------------------------|---------------
Target systems: | Header for list of target systems
system-one: | Logical name of target system, which can receive requests. You can configure one or more target systems, without limit
baseUrl: http://www.example.com/test | URL for requests

### Full configuration
Note: this configuration file contains all supported settings.
Some settings are conflicted with other.
Do not try to use confifuration in exactly this form.
```yaml
---
Options:
  HttpConnectTimeout: 2
  HttpReadTimeout: 2
  RetryPause: 10
  DeferredStatus: 202
Target systems:
  system-one:
    baseUrl: http://www.example.com/test
    successCodes: [200,201,202,203,204,205,206,207,208,209]
    successRegexp:
    - .+success.+
    errorCodes: [500,501,502,503]
    errorRegexp:
    - .+error.+
    ignoredUrls:
    - "/ignored/.*"
    extractors:
    - type: HEADER
      options: "Object-Id"
    - type: URL_REGEX
      options: ".*/([\\d]+)$"
    - type: BODY_JSONPATH
      options: "$.id1"
    - type: BODY_XPATH
      options: "outer/id1/text()"
    HttpConnectTimeout: 30
    HttpReadTimeout: 20
    RetryPause: 120
    DeferredStatus: 286
  system-two:
    .....
```

Description

 Configuration parameter | Description
-------------------------|---------------
Options: | Header for list of global settings. Global settings are applied if they are not specified for system (<sup>note 1</sup>)
HttpConnectTimeout | Time of connect timeout, in seconds
HttpReadTimeout | Time of read timeout, in seconds
RetryPause | Time between attempts, in seconds (<sup>note 2</sup>)
DeferredStatus | Http Status Code, which senders will receive in case of request cannot be evaluated immediately
successCodes | List of Http status codes that are accepted for success response
successRegexp | List of regexp that are accepted for success response. Applied to response body
errorCodes | List of Http status codes that are accepted for failure response
errorRegexp | List of regexp that are accepted for failure response. Applied to response body
ignoredUrls | List of regexp that are ignored by Eureka BPO ESB. Applied to request's URL
extractors | extractors are using to organize queues of requests (<sup>note 3</sup>)

Notes:
1. Where is a sequence, in which parameters are resolved for every request:
    1. Request header value
    1. System settings
    1. Global settings
    1. Default values
1. RetryPause must be bigger than HttpConnectTimeout + HttpReadTimeout
1. With help of extractors it is possible to organize queue for separate objects. Requests from this queue can be evaluated only after previous request for same object. Id of object can be extracter from URL, header or body of http request.

Value of type of extractor | Object Id extracted from | Meaning of options of extractor
------------------|--------------------------|---------------------------------
HEADER | header | name of header
URL_REGEX | URL | regex for url string
BODY_JSONPATH | http message body | jsonpath-expression
BODY_XPATH | http message body | xpath-expression

## Request parameters
Every request can contain headers that have influence on Eureka BPO ESB behaviour. Hier are all these headers with their description. All headers are optional.

Header name | Description
------------|--------------
Eureka-BPO-ESB-Ignorable | "true" if this request must be ignorable, otherwise in accordance with system configuration
Eureka-BPO-ESB-Back-Url | Url for delivery the system response in the event of system response cannot be obtained immediately

## Response parameters
Usually target system process requests immediately. But in the event of target system unaccessible or some error was acquired during processing on target system response can be deferred.

#### Headers of immediately responses
Immediately responses can have these Eureka BPO ESB specific headers.

Header name | Description
------------|---------------
Eureka-BPO-ESB-Transaction-Id | Unique identifier of request. To every request (except ignorable) unique identifier is given.

#### Headers of deferred responses
Deferred responses have the same headers as immediately and some additional:

Header name | Description
------------|---------------
Eureka-BPO-ESB-Original-Status-Code | Http status code of response target system
Eureka-BPO-ESB-Original-Status-Text | Http status text of response target system
