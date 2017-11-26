# Motus Version 2.0 API \
**Paul Morril**\
5 September 2017 #


Each request requires the single json={} parameter, with contents as
described.

Common Parameters:

 - **date**: REQUIRED on all requests; formatted as YYYYMMDDHHMMSS in GMT
 - **login**: optional unless authentication required on call
 - **pword**: optional unless authentication required on call
 - **fmt**: optional, defaults to `json`

Authentication is normally not required (exceptions below), but
unauthenticated requests will not return private information.

## 1. Querying Projects ##

### 1.1 List projects with basic information ###

    /api/projects

Additional Parameters: none

### 1.2 List projects with descriptions ###

    /api/projects/descriptions

Additional Parameters: none

## 2. Querying Receivers ##

### 2.1 List receivers ###

    /api/receivers

(alias: /api/receivers/list)

Additional Parameters:

 - **status**: filter on basis of deployment status (0|1|2)

### 2.2 List Receiver Deployments (nested version) ###

This version includes JSON nested structures for individual antennas (csv is
not supported)

    /api/receivers/sensordeployments

Additional parameters:

 - **projectID**: filter by a single project ID
 - **year**: filter by year deployment started
 - **serialNo**: filter by receiverID
 - **status**: filter by deployment status (0|1|2)

### 2.3 List Receiver Deployments (flat version: json or csv) ###

    /api/receivers/deployments

Additional parameters:

 - **projectID**: filter by a single project ID
 - **year**: filter by year deployment started
 - **serialNo**: filter by receiverID
 - **status**: filter by deployment status (0|1|2)

### 2.4 List Receiver Antennas ###

    /api/receivers/antennas

Additional parameters:

 - **projectID**: filter by project ID
 - **serialNo**: filter by receiverID
 - **status**: filter by deployment status

## 3. Querying Tags ##

### 3.1 List Tags ###

    /api/tags/

(alias: /api/tags/list)

Additional parameters:

 - **projectID**: filter by project ID
 - **year**: filter by year deployment began
 - **mfgID**: filter by manufacturers id

### 3.2 Search Tag Deployments ###

    /api/tags/search

Additional parameters:

 - **projectID**: filter on project id
 - **status**: filter on deployment status (0|1|2)
 - **mfgID**: filter on manufacturer id
 - **qSearchMode**: set deployment period search mode (`startsBetween`|`overlap`)
 - **tsStart**: unix timestamp, start of deployment
 - **tsEnd**: unix timestamp, end of deployment

## 4. Miscellaneous Querying ##

### 4.1 Validating a user ###

Check that credentials are valid. If so, return basic user information
and a list of projects that the user can access.

    /api/user/validate

Additional parameters:

 - **login**: users login name
 - **pword**: users password

### 4.2 List Species ###

    /api/species

Additional parameters:

 - **qstr**: a search string from common or scientific name.
 - **qgroup**: filter on a group (`bird`|`mammal`|..)
 - **qlang**: search common name for this language (`en` |  `fr` | `cd`)

### 4.3 List receiver status ###

    /api/receiverstatus

## 5. Registering Tags, Receivers and Projects ##

### 5.1 Register a tag ###

    /api/tag/register

See api version 1.0 documentation for parameters

### 5.2 Register a receiver (sensor) ###

    /api/receiver/register

Additional parameters:

 - **secretkey**: sensor hash key
 - **receiverType**: eg: LOTEKSRX800
 - **serno**: the receiver id (eg: `SG-1234BBBK678`, `Lotek-153`)

### 5.3 Register a project ###

    /api/project/register

Additional parameters:

 - **projectName**: a name for the project
 - **projectCode**: a short code for the project (not available yet on live
server)

### 5.4 Deploy a tag (untested) ###

    /api/tag/deploy

See api version 1.0 documentation for parameters

### 5.5 Undeploy a tag (untested) ###

    /api/tag/undeploy

See api version 1.0 documentation for parameters (deletetagdeployment)
