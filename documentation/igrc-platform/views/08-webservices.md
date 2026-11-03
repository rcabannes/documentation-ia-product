---
title: "Views and WebServices"
description: "Configure and use WebServices to retrieve data from Identity Analytics"
---

# Views and WebServices

All of the data of the Identity Ledger is accessible from the Web interfaces, two interface families exist:

- The Web portal in order to publish results and analyses to the security and clearance stakeholders
- The Web services in order to facilitate the dissemination of information to the third-party technical services

The web services in particular allow us to "close the loop" of the PDCA approach initiated with Identity Analytics, it is effectively a question here, for example, of offering the list of deviations to be corrected to a third-party service.

An example of implementation and use of the web services is the following:

A weekly loading is accomplished with the help of Identity Analytics, all of the repositories are loaded, as well as the HR database. On the basis of all of this data, Identity Analytics identifies the people having left the company as well as the access accounts that have remained open. A WebService is published listing all of the accounts that are in this state. A third-party service contacts the Identity Analytics WebService at regular intervals (every week in our case), and starts the remediation workflows in the company's ticket management system (ex: Jira, Remedy,...)

Another example of implementation is the following:

A log analysis system has been set up, correlating various information about the accounts and detecting abnormal situations on accesses to the infrastructure and to the file systems. Summary reports are consolidated by the solution. The SIEM solution can then contacts the Identity Analytics WebServices in order to enrich the information present, notably by adding all the information relative to the user context (who is the account owner, what are the account's rights,...)  
All of the data being available, it is even possible to develop one's own browsing or consultation interfaces, for example, in order to enrich one's own interfaces with information from Identity Analytics.

This note presents the use of the WebServices in Identity Analytics.

## Basic principles

The WebServices follow the REST norm (cf [wikipedia](http://www.google.com/url?q=http%3A%2F%2Ffr.wikipedia.org%2Fwiki%2FRepresentational_state_transfer&sa=D&sntz=1&usg=AFrqEzf1kipVVsK4vgtufB0O48EAo2k9qA)), the URL indicates both the operation and the basic element on which the operation is performed. The results are returned in the JSON format (cf [wikipedia](http://www.google.com/url?q=http%3A%2F%2Ffr.wikipedia.org%2Fwiki%2FJson&sa=D&sntz=1&usg=AFrqEzdsUx9kd8ytDTLZXHJAYE2yYSoQ6A)) and are therefore directly interpretable from standard Javascript, or any programming language with a JSON parser.

> [note!] You can configure the WebServices to return XML content instead of JSON content by using the `_format=xml` parameter. The XML format is Microsoft Office compliant.|

The WebServices allow access to views, whose parameters are set in the project, so it is a simple matter to create your own WebServices, you have only to set parameters for a view in the project. You can refer to the corresponding section of the documentation for more information about [Views](index.md).

WebServices are accessible from the web portal and are activated by default when you deploy a portal. The URL to access the WebServices is protected so that you must identify yourself to the Web portal in order to access the Web Services. The base URL for the webservices resembles `http(s)://server:port/projet/ws/` where:

- server:port represents the host on which the portal is deployed
- project represents the name of the deployed project

## Presentation of the available WebServices

### Retrieve the list of results of a view

The corresponding Web service is **"result"**, the REST syntax is:  
`/ws/results/[view identifier]`

For example to request the results of the view `br_identity` (element `/views/identity/identity.view` of your project) you would type the following url  
`http://localhost:8080/demonstration/ws/results/br_identity`

The results are as follows:

```json
{ "total":862,
"limit":1000,
"success" : true,
"start":0,
"data" : [ { "employeetypecode" : "Contractor",
"employeetypedisplayname" : "Contractor",
"employeetyperecorduid" : 9,
"employeetypeuid" : "CONTRACTOR_1349174669639_1897",
"fullname" : "Abdel BOUCHEZ",
"givenname" : "Abdel",
"internal" : false,
"mail" : "ABOUCHEZ11@brainwave.fr",
"recorduid" : 955,
"repositorycode" : "Contractors",
"repositorydisplayname" : "Contractors",
"repositoryextractdate" : "Tue Sep 04 10:51:30 CEST 2012",
"repositorymedia" : "R:/git/demonstration/demonstration/importfiles/two/contractors.csv",
"repositoryrecorduid" : 10,
"repositorytype" : "I",
"repositoryuid" : "CONTRACTORS_1349174669670_1898",
"surname" : "BOUCHEZ",
"titlecode" : "Mr",
"titledisplayname" : "Mr",
"titlerecorduid" : 3,
"titleuid" : "MR_1349174640974_137",
"uid" : "BOUCHEZ_ABDEL_1349174669895_1905"
},
...
```

The attributes in the header of the response provide information on the execution of the webservice:

- `success`: True if the request has results, otherwise false
- `total`: Total number of results
- `limit`: Maximum number of results returned
- `start`: Index of the first result returned. this is useful when windowing displayed results
- `data`: The elements of the response

It is possible to perform some windowing in order to retrieve a subset of results, with the help of the view parameters:

- `_start`: starting index (0 by default)
- `_limit`: number of results returned (1000 by default)
- `_timeslot`: restrict the results to the chosen timeslot (`_timeslot=<value>`)

For example:  
`http://localhost:8080/demonstration/ws/results/br_identity?_start=10&_limit=2`

The results are as follows:

```json
{
  "limit": 2,
  "start": 10,
  "success": true,
  "total": 862,
  "data": [
    {
      "employeetypecode": "Contractor",
      "employeetypedisplayname": "Contractor",
      "employeetyperecorduid": 9,
      "employeetypeuid": "CONTRACTOR_1349174669639_1897",
      "fullname": "Danièle DULAC",
      "givenname": "Danièle",
      "internal": false,
      "mail": "DDULAC20@brainwave.fr",
      "recorduid": 962,
      "repositorycode": "Contractors",
      "repositorydisplayname": "Contractors",
      "repositoryextractdate": "Tue Sep 04 10:51:30 CEST 2012",
      "repositorymedia": "R:/git/demonstration/demonstration/importfiles/two/contractors.csv",
      "repositoryrecorduid": 10,
      "repositorytype": "I",
      "repositoryuid": "CONTRACTORS_1349174669670_1898",
      "surname": "DULAC",
      "titlecode": "Ms",
      "titledisplayname": "Ms",
      "titlerecorduid": 4,
      "titleuid": "MS_1349174641389_141",
      "uid": "DULAC_DANIELE_1349174670409_1963"
    },
    {
      "employeetypecode": "Contractor",
      "employeetypedisplayname": "Contractor",
      "employeetyperecorduid": 9,
      "employeetypeuid": "CONTRACTOR_1349174669639_1897",
      "fullname": "Fatia PHILIBERT",
      "givenname": "Fatia",
      "internal": false,
      "mail": "FPHILIBE16@brainwave.fr",
      "recorduid": 1799,
      "repositorycode": "Contractors",
      "repositorydisplayname": "Contractors",
      "repositoryextractdate": "Tue Sep 04 10:51:30 CEST 2012",
      "repositorymedia": "R:/git/demonstration/demonstration/importfiles/two/contractors.csv",
      "repositoryrecorduid": 10,
      "repositorytype": "I",
      "repositoryuid": "CONTRACTORS_1349174669670_1898",
      "surname": "PHILIBERT",
      "titlecode": "Ms",
      "titledisplayname": "Ms",
      "titlerecorduid": 4,
      "titleuid": "MS_1349174641389_141",
      "uid": "PHILIBERT_FATIA_1349174669911_1907"
    }
  ]
}
```

### Choose which attributes must be displayed

The corresponding Web service is “views”, the REST syntax is:  
`http://localhost:8080/demonstration/ws/results/[view]?_columns=uid,givenname,surname`

### Retrieve the list of views available

The corresponding Web service is "**views**", the REST syntax is:  
`/ws/views`

For example:  
`http://localhost:8080/demonstration/ws/views`

The results are as follows:

```json
{ "success" : true,
  "total" : 273,
  "views" : [ { "description" : "Rule details",
        "displayname" : "Rule details",
        "entity" : "SearchLog",
        "name" : "br_controlrule"
      },
      { "description" : "Permission rule evolution on the last 3 months",
        "displayname" : "Permission rule evolution on the last 3 months",
        "entity" : "Permission",
        "name" : "br_sharedfolderruleevol"
      },
      { "description" : "delta review people who changed organisation and who where in a given organisation in the prvious timeslot",
        "displayname" : "delta review people who changed organisation and who where in a given organisation in the prvious timeslot",
        "entity" : "Identity",
        "name" : "deltareview_hrteam_movementsoldorganisation_1"
      },
      { "description" : "Organisation search filtered by a root OU",
        "displayname" : "Organisation search filtered by a root OU",
        "entity" : "Organisation",
        "name" : "organisationsearchou"
      },
...
```

### Retrieve the list of columns returned by a view

The Web service to return specific columns of a view is **"columns"**, the REST syntax is:  
`/ws/columns/[view identifier]`

For example:  
`http://localhost:8080/demonstration/ws/columns/br_identity`

The results are as follows:

```json
{  "success" : true,
   "columns" : [ { "label" : "Identifiant unique interne de l'identité",
        "name" : "recorduid",
        "type" : "Integer"
      },
      { "label" : "Identifiant intemporel de l'identité",
        "name" : "uid",
        "type" : "String"
      },
      { "label" : "Identifiant unique interne du référentiel d'identités",
        "name" : "repositoryrecorduid",
        "type" : "Integer"
      },
      { "label" : "Identifiant intemporel du référentiel d'identités",
        "name" : "repositoryuid",
        "type" : "String"
      },
      { "label" : "Code du référentiel d'identités",
        "name" : "repositorycode",
        "type" : "String"
      },
      { "label" : "Nom d'affichage du référentiel d'identités",
        "name" : "repositorydisplayname",
        "type" : "String"
      },
      { "label" : "Média du référentiel d'identités",
        "name" : "repositorymedia",
        "type" : "String"
      },
      { "label" : "Date d'extraction du référentiel d'identités",
        "name" : "repositoryextractdate",
        "type" : "Date"
      },
      { "label" : "Type de référentiel d'identités",
        "name" : "repositorytype",
        "type" : "String"
      },
      { "label" : "Matricule de l'identité",
        "name" : "hrcode",
        "type" : "String"
      },
      { "label" : "Alias de l'identité",
        "name" : "nickname",
        "type" : "String"
      },
      { "label" : "Prénom de l'identité",
        "name" : "givenname",
        "type" : "String"
      },
      { "label" : "Second prénom de l'identité",
        "name" : "middlename",
        "type" : "String"
      },
      { "label" : "Nom de famille de l'identité",
        "name" : "surname",
        "type" : "String"
      },
      ...
    ]
}
```

### Retrieve the list of parameters accepted by a view

The Web service that allows you to list the available parameters of a view is **"parameters"**. The REST syntax is:  
`/ws/parameters/[view identifier]`

For example:  
`http://localhost:8080/demonstration/ws/parameters/br_identity`

The results are as follows:

```json
{
  "success": true,
  "parameters": [
    { "label": "Mail", "name": "mail", "type": "String" },
    { "label": "Record UID", "name": "recorduid", "type": "Integer" },
    { "label": "Given Name", "name": "givenname", "type": "String" },
    { "label": "Surname", "name": "surname", "type": "String" },
    { "label": "Internal", "name": "internal", "type": "Boolean" },
    { "label": "HR Code", "name": "hrcode", "type": "String" }
  ]
}
```

### Specify the displayed data format

To choose in which format the data will be displayed (example here is xml):  
`http://localhost:8080/demonstration/ws/results/[view]?_format=xml`

### Retrieve the list of available timeslots

The Web service that allows you to list the available timeslots is **"timeslots"**, the REST syntax is:  
`/ws/timeslots`

For example:  
`http://localhost:8080/demonstration/ws/timeslots`

The results are as follows:

```json
{
  "total": 2,
  "limit": 2147483647,
  "start": 0,
  "success": true,
  "timeslots": [
    {
      "commitdate": "20121002131318",
      "displayname": "Import started Oct 2, 2012 1:08:05 PM",
      "importdate": "20121002130805",
      "status": "A",
      "uid": "20121002130805_2"
    },
    {
      "commitdate": "20121002125001",
      "displayname": "Import started Oct 2, 2012 12:43:23 PM",
      "importdate": "20121002124323",
      "status": "C",
      "uid": "20121002124323_1"
    }
  ]
}
```

#### Managing errors

When an error appears, the engine sends back a response with the attribute `success = false` in the header of the results, for example:

```json
{
  "success": false,
  "message": "java.lang.NumberFormatException: For input string: \"99999999999\""
}
```

The type of error is also displayed allowing you to debug you webservice.

### Security Settings for Service Access

#### Manage access authentication

The WebService service relies on the access security of the Tomcat container and thus on the access security as defined in the configuration file of the Web application under `/WEB-INF/web.xml`.  
By default the section is as follows, note that the URL `ws/*` is protected the same way as the others:

```xml
<security-constraint>
    <web-resource-collection>
        <web-resource-name>Portal</web-resource-name>
        <url-pattern>/*</url-pattern>
    </web-resource-collection>
    <auth-constraint>
        <role-name>user</role-name>
    </auth-constraint>
</security-constraint>

<security-constraint>
    <web-resource-collection>
        <web-resource-name>Portal Login</web-resource-name>
        <url-pattern>/</url-pattern>
        <url-pattern>/login/*</url-pattern>
    </web-resource-collection>
</security-constraint>

<login-config>
    <auth-method>FORM</auth-method>
    <realm-name>UserDatabase</realm-name>
    <form-login-config>
        <form-login-page>/login/login.html</form-login-page>
        <form-error-page>/login/login.html</form-error-page>
    </form-login-config>
</login-config>

<security-role>
    <description>Utilisateur</description>
    <role-name>user</role-name>
</security-role>
```

In order to activate the web service call, you have to ensure that the authentication is performed prior to the URL access. Most of the time, as webservices systems have a hard time handling FORM authentication, you should deploy a Single Sign On system (SSO). If you don't have one, contact Professional Services or Support to obtain the Identity Analytics SSO valve for Tomcat. These libraries are not available for direct download.

> [!warning] Please note that this Valve is provided as an openSource component along with its source without any support.|

To activate Identity Analytics's Tomcat SSO Valve (`com.brainwave.tomcat.valve.PassthroughAuthenticationValve`), copy the JARs below into the `/lib` directory of your Tomcat folder.

**4.x (recommended)** — Tomcat 9.0, Java 8 or later:

- `tomcat-addons-valve-passthrough-4.x.x.jar`
- `tomcat-addons-common-4.x.x.jar`

**5.x (recommended)** — Tomcat 11.0 (Jakarta), Java 21 or later:

- `tomcat-addons-valve-passthrough-5.x.x.jar`
- `tomcat-addons-common-5.x.x.jar`

**3.x (not recommended)** — Tomcat 9.0. One JAR: `bw-tomcat-9.0-addons.java1.8-3.x.x.jar` on Java 8, or `bw-tomcat-9.0-addons.java17-3.x.x.jar` on Java 17 or later.

Then:
- Edit your `/conf/server.xml` file and add the following section between the \<Host\> tags :

```xml
<Valve className="com.brainwave.tomcat.valve.PassthroughAuthenticationValve"
  config="sso.xml"
  refreshRate="5"
  urlPattern="^/[a-z,A-Z,0-9,_,-]+/ws/.*$" />
```

By doing so, the SSO valve will be actived by default whatever the web application. You can restrict its activation to a subset of URLs by using the urlPattern parameter (standard regex).  
The SSO valve points to an XML configuration file. Put it in `$CATALINA_BASE/conf/sso.xml`. From **3.4.0**, **4.1.0** and **5.1.0**, a bare `config` such as `sso.xml` is read from that directory. An absolute path, or a path relative to `$CATALINA_BASE`, also works. It can be edited on the fly (the valve configuration is updated every "5" seconds if needed in the upper example).

- Create `$CATALINA_BASE/conf/sso.xml`

`$CATALINA_BASE/conf/sso.xml` contains the SSO tokens and their user configuration. Here is an example:

```xml
<tokens>
    <token id="HsCnykb8RtMdGpKlu7W7SlJs" login="ABOURGET18" IPpattern="^.*$">
        <role>user</role>
    </token>
</tokens>
```

This means that every time the "HsCnykb8RtMdGpKlu7W7SlJs" token will be presented along with a HTTP request, an authentication will be performed on the fly with the following login "ABOURGET18" and the following roles (you can add more than one \<role\> tag). This will be valid only if the request is made from the IPpattern source IP address (standard regex). You are strongly encouraged to enforce security by properly configuring the IPpattern parameter.

> [!warning] By default the login defined in the files `sso.xml` must correspond to a reconciled account as the default view used to log into the portal is `br_portalidentity`.

In order to use your token you can either add an "ssokey" custom HTTP request header with the token value or add an `_ssokey` URL parameter with the token value. Authentication will be done on the fly. Note that custom HTTP header is the preferred method as URL parameters are most of the time stored in HTTP access logs, this can lead to security issues.

Here is an example of an URL:  
`http://localhost:9090/PortalReports/ws/results/br_identity?_ssokey=HsCnykb8RtMdGpKlu7W7SlJs`

Here is also an Example of a HTTP header which is the prefered method:

```txt
GET /PortalReports/ws/views HTTP/1.1
Host: localhost:9090
ssokey: HsCnykb8RtMdGpKlu7W7SlJs
...
```

### Manage access rights

The management of access rights to the different services is handled externally, by setting access strategies according to the URLs accessed. The REST notion allows fine-tuning of accesses, so it is possible to define the access rules view by view.  
Practically speaking, it is common to make a first layer filter of access by source IP (communication with WebServices being most often machine to machine type), then to define dedicated roles if necessary to filter the accessible services according to user roles.

The [Remote Address Filter](http://tomcat.apache.org/tomcat-9.0-doc/config/filter.html#Remote_Address_Filter) for `Tomcat` allows access controls by source IP.

Setting access rights by role according to the URL is done with the `Security Constraint` sections, for example:

```xml
<security-constraint>
    <display-name>Web Services Access</display-name>
    <web-resource-collection>
        <web-resource-name>Web Services</web-resource-name>
        <url-pattern>/ws/results/getaccountstodisable</url-pattern>
        <url-pattern>/ws/results/getaccountstodelete</url-pattern>
        <url-pattern>/ws/results/getidentityinformation</url-pattern>
    </web-resource-collection>
    <auth-constraint>
        <role-name>iam_engine</role-name>
    </auth-constraint>
</security-constraint>
```
