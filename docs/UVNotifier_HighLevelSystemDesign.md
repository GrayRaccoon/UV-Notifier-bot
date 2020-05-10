# UVNotifier high level system design

## Content index
- [Definitions and Acronyms](#definitions-and-acronyms)
- [Changelog](#changelog)
- [Abstract](#abstract)
- [Goal/Objectives](#goalobjectives)
    - [StakeHolders](#stakeholders)
- [Assumptions](#assumptions)
- [Limitations & Unknowns](#limitations--unknowns)
- [Out of Scope](#out-of-scope)
- [Proposal](#proposal)
    - [Architecture](#architecture)
        - [Phase 1: description](#phase-1-description)
        - [Phase X: next steps](#phase-x-next-steps)
        - [Architecture Diagram](#architecture-diagram)
        - [External components](#external-components)
        - [Owned components](#owned-components)
        - [End-to-End flow sequence diagram](#end-to-end-flow-sequence-diagram)
    - [Data Structures](#data-structures)
- [Implementation plan](#implementation-plan)
- [Costs](#costs)


## Definitions and Acronyms

**Note: some of these terms require knowledge on Heroku Platform 
and its Products, for more information refer to 
[Heroku Home](https://www.heroku.com/home)**

- Paas: Platform As A Service
- [Heroku](https://www.heroku.com/home): Paas that enables developers to easily build, run, and operate applications entirely in the cloud.
- UV: Ultraviolet
- [OpenUV](https://www.openuv.io/): Open UV Index API, Global Real-Time UV Index Forecast API.
- [Spring Cloud](https://spring.io/projects/spring-cloud): Spring Cloud provides tools for developers to quickly build some of the common cloud patterns in distributed systems.
- [RabbitMQ](https://www.rabbitmq.com/): RabbitMQ is the most widely deployed open source message broker.
- [Message Broker](https://www.ibm.com/cloud/learn/message-brokers): Message brokers are an inter-application communication technology to help build a common integration mechanism to support cloud native, microservices-based, serverless and hybrid cloud architectures.
- TODO: add more as required.


## Changelog 
- revision 1:
    - Initial Version of high level system design.


## Abstract
This project aims to consume data provided by OpenUV, 
and use this data to send periodic notifications to a given user 
in a daily basis and/or near-real time fashion.
The system is designed to be cloud based and 
its notifications may contain information considered as relevant for the end users. 

As OpenUV is a private paid service there is no need of including 
source attribution(s) in the notifications messages.

TODO: Refine abstract once full high level design is done.


## Usage Overview
A user subscribes to the system events 
by starting following an online account (Twitter/Facebook) 
so that, every morning user can receive updates 
about UV levels in all supported locations in his or her daily feed news.


## Goal/Objectives

Provide a suitable system architecture 
Cloud Native based which would freely provide daily basis 
and/or near-realtime notifications 
regarding UV levels in the supported locations.

Additionally, this project targets the open-source software development community, to share the code of this project, as long as guidance/practices/methodologies used in its development.

### Stakeholders
- Project owner: [@Heriberto Reyes Esparza](https://github.com/herychemo) 
- Project co-owners: [@Gray Raccoon Members](https://github.com/GrayRaccoon)
- Users: Anyone interested using this system with the intention of querying or being notified about UV levels notifications in any of the supported locations.
- Open-source community: anyone insterested in using this software for learning purposes or those allowed by MIT License.


## Assumptions
- This is a **high level** design, is not ment to hold specifics of implementation of the presented components on this system.
- Each "supported location" represents a manually configured pair of Latitude, Longitude in the system.
- Adding one or more new locations (becoming a "supported location") is treated as separate from this design, assume this is done manually.
- Data obtained from the OpenUV programmatic API is considered the **source of truth**
- Data retrieval frequency is by default **everyday at 7:00AM**, it is subject to change.
- For Rate limits at the "publisher", Twitter's limits will be considered, 
    see [Twitters Limits](https://developer.twitter.com/en/docs/basics/rate-limits). 
    As of 04/29/2020, the Tweet limit is 300 requests per user/per app within a 3 hour window.


## Limitations & Unknowns
- Twitter needs to approve a development account. ETA is not known.
- How does Twitter detect Spam. Should there be any considerations?
- At this phase a user will see all the updates even the ones 
    from the supported locations that might not be relevant for him or her.


## Supported use-cases
- Specific rules that determine whether or not a notification should be published, it should prevent spam (Avoid publishing same notification more than once and/or publishing too frequently).
- Users are notified via twitter(users should follow a determined Twitter account to receive messages)
- System is able to publish text messages as tweets on a determined twitter account with uv levels info in plain text.
- In case of failure, message notifications must be discarded, it is subject to change. 
- TODO: Complement using app selected features!


## Out of scope
- Any form of user data collection: no data will be collected in this project.
- Search/Lookup functionality
- Email notifications are out of scope
- Automation of setup for adding new supported locations.
- Error handling
- Retry mechanisms
- Notifications with UV levels with a nice friendly chart.
- TODO: Complement using app selected features!


## Proposal
### Architecture
The proposed system will be a **Spring boot application** deployed as a Heroku App.
Architecture might be developed in multiple **phases**, 
This document version covers the Phase 1, 
and potential future phases might contain or not some items from the out of scope list.

#### Phase 1 description
Relies on Spring Scheduled Tasks to trigger the process of consuming the source of truth, 
and it uses Spring services to validate and delivery the outcome of the process as system notifications.
TODO: Complement using app selected features!

#### Phase X (next steps)
The following are features considered for upcoming development.

- Display notification information as a friendly chart image.
- Ease the addition of new supported locations.

#### Architecture diagram
The following diagram describes the system components and its external dependencies.

##### 1.- System Context Diagram
High Level System's context.
![uvNotifier-systemContext](https://gitlab.com/GrayRaccoon/UV-Notifier-bot/-/raw/master/docs/UVNotifier-SystemContextDiagram.svg)

##### 2.- Container Diagram
High level system containers responsibilities.
![uvNotifier-systemContext](https://gitlab.com/GrayRaccoon/UV-Notifier-bot/-/raw/master/docs/UVNotifier-ContainerDiagram.svg)

##### 3.- Component Diagram
Main Backend container components interaction definition.
![uvNotifier-systemContext](https://gitlab.com/GrayRaccoon/UV-Notifier-bot/-/raw/master/docs/UVNotifier-ComponentDiagram.svg)


##### External Systems:
- **OpenUV API  System**: Source of the system data through Rest API. It is limited to 50 request per day.
- **Twitter System**: Platform where a "Twitter Publisher" event handler would post tweets.
- **Other**: Any other platform(like sms, email, etc..) that would require it's own event handler or mechanism in order to send in notifications.


#### Owned components:
- **Process Scheduled Task**: Triggered everyday at 7:00AM by a programmatic Spring scheduled task.
    This task will invoke the whole system process.
- **Fetch Data Service**: Triggered by the **Process Scheduled Task**, it will consume the *Source of truth and 
    deliveries uv level details as Cloud Stream Events.
- **OpenUV API Client**: Invoked by **Fetch Data Service**, it will perform a set of API requests to *OpenUV API* 
    in order to fetch UV level details for each supported location, and then aggregates this info into a domain object back to its caller.
- **Database**: Stores the required info about all the Supported locations.
- **Message Broker**: Receives Cloud Stream Events and deliveries each of them to all the components listening the queue.
- **Twitter Publisher**: Receives events from the *Message Broker* about UV levels and then builds and publishes the events as notifications in Twitter account.


#### End-to-End flow sequence diagram
![uv-notifier-sequence-diagram](https://gitlab.com/GrayRaccoon/UV-Notifier-bot/-/raw/master/docs/UVNotifier-SequenceDiagram.svg)


### Data structures

#### Sample OpenUV API ${ext_api_path} response
TODO: Add up to 3 api calls as required.
```json
{
    "status": "ok"
}
```

#### Any other DTO definition
```json
{
    "key": "value"
}
```

**Note: Specific process implementation structures in this project are not part of high level design.**


## Implementation plan
See **assumptions** As this is high level design, We'll keep it simple
- Steps to complete


## Costs
### Monthly Cost (30 day) (Phase 1)
(DESIGN NOTE: Add cost considerations)
