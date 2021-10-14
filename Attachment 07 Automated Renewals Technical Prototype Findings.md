# Prototype Findings

A summary of findings, lessons learned and additional questions from our work on the initial technical prototype. 

## Summary
The purpose of our prototyping was to get a deeper understanding how the parts of the existing Aries System to be updated as a part of Automated Renewals work. Given the complexity of the system, a full scale prototype was not possible given constraints. Instead, we chose to a handful of prototypes on the parts of ARIES we think will need to be updated as a part of developing Automated renewals. These included WebSphere Application Server, WebSphere Operational Decision Manager, Mule Community Edition, HP Extream (a.k.a. OpenText) and Quartz Scheduler.

## Components

### DHSS controlled Systems

- **Websphere App Component** – The Websphere app component for the ARIES application is a j2EE application leveraging the MVC architecture pattern currently run using Websphere 8. The app component is responsible for handling the web requests from the users and processing the request to gather or calculate the requested information for the user. Front end pages use a database framework with the required information for each page and relevant links to other pages, all being managed through Postgres Enterprise Manager to modify the database.

- **Service Bus** – Mule is a middleware and messaging application used to create APIs and data flows. Mule applications connect systems, services, APIs, and devices using API-led connectivity instead of point-to-point integrations. Mule applications provide functionality for message routing, data mapping, orchestration, reliability, security, and scalability. For the purposes of the prototype and future development with Mule, we use Mule Runtime v3.9.0 and Anypoint Studio v6.6, the integrated IDE shipped with Mule.

- **Correspondence** - We shadowed the M&O team while they updated an existing notice to address defects, and in the process created a custom notice. Aries Automated Renewals will likely leverage the existing Correspondence Framework and data stored in our Postgres Enterprise Database. The main components are the Rules Engine (IBM WebSphere Operational Decision Manager), Batch (Quartz), and Notice Generation (HP Exstream). Details on the Rules Engine will be covered separately, but in the context of Correspondence, understand the Rules Engine contains the logic of who is/not eligible for Medicaid and on what basis the determination was made, which must be included in the correspondence.

- **Rules** - For rules, we used IBM WebSphere Operational Decision Manager (WODM). We created a rule flow based on a simple Java Class to create a BOM/XOM that was deployed to a test Rules Execution Server. We were able to call the rule to validate results was done using SOAP UI using the WSDL generated when BOM/XOM was created.

- **Batch** - The job scheduling of the Websphere application is done nightly, and is implemented using [Quartz Scheduler](http://www.quartz-scheduler.org), which leverages our [EnterpriseDB Postgres Database](https://www.enterprisedb.com/). Communications with external partners is currently done via SOAP webservices and file transfers (ftps).

- **DevOps pipeline info and work management** - We use Azure DevOps/TFS in conjunction with Git for our build and release development pipeline.

### External Services

- **CMS HUB** – For the prototype we connected to two different CMS Hub services - the Hub Connectivity Service and the Verify Current Income Service. The Hub Connectivity Service returns a 'Success' message if a connection is created and authentication is validated. 

# Key findings

## Websphere App Component
 
We explored adding additional pages in the ARIES Worker Portal Environment in order to update or add pages to the front end. During the investigation, it was found that there is strong documentation for the app component, but we no longer have access to all of tools specified by the vendor that were originally used for page creation. This software is performing SQL operations on the database to perform desired changes to the front end. These SQL queries can be manually repeated in order to accomplish the same goal, however all of the proper data must be present in the database for the front end to be properly instantiated. It was found to be possible to create and update pages to our current application. However, it was cumbersome due lack of the original tools. 

## Service Bus

We created a sample service using Mulesoft's Anypoint Studio IDE and the Mule 3.9 CE runtime. The service mimics the ARIES Mule services as much as possible, which were created using Mule 3.3.1 CE (which is not available through Mulesoft any longer). 

The flow of data through the service is as follows: 
SOAP Request -> HTTP Listener -> Proxy Service (CXF) -> Proxy Client (CXF) -> HTTP Requester -> CMS Hub Endpoint.

The proxy service is configured with the CMS Hub endpoint's WSDL. The HTTP request is configured to use both a Trust Store and Key Store, TLSv1.2 protocol, and TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384 Cipher Suite for authentication with CMS. Additionally, the original SOAP request to the service must include a username/password in digest format along with a timestamp.

In this prototype, a Mule application was created to proxy and add authentication/security to SOAP messages between DHSS and CMS Hub's HubConnectivityService and VerifyCurrentIncomeService. 

## Correspondence

While investigating correspondence, it was found that even though we use older technologies, the process of creating new notices is fairly straightforward. The potential challenges may be a limitation on licensing for HP Extreme. Updating existing, or creation of, new notice templates requires using HP Exstream (supported by OpenText) tools for creation or updates of notices. If new data is needed to be presented to the forms, this may require doing complementary updates to code for the WebSphere applications, Quartz batch jobs and WebSphere Rules server. The biggest limitation found is the fact that we are running older versions of software, which makes getting support challenging. In addition, for formal development, the processes for developing and deploying these components are still being put into place.

## Rules

Using WODM, we were able to create our own custom rules. Updating and editing rules, with the version of IBM ODM we are running, requires the installation of Eclipse along with the Rules Designer Extension added. To setup an environment for developing/testing we use the installer for "IBM WebSphere Operational Decision Management V8.0" (launchpad.exe) which is included in installer image downloaded from IBM. 

The Rules code has not formally been migrated into TFS from the original SVN Repository that was given to us from the previous contractor. As a result there is currently no automated build pipeline in place for Rules. This work is not currently defined. The manual method to test Rules described in IBM documentation and the previous contractor's documentation, for the version we currently use (using Microsoft Excel), is not possible. We think running an older version of Office may solve this.

Given our prototype did not include an actual update/addition to the existing ARIES ruleset/flows, there may be additional steps/concerns not documented here. There was a detailed overview of how Rules are developed and deployed, provided by the vendor during handover.

## Batch

Batch prototyping was not performed.
