# Motus Version 2.0 API #

**Paul Morril**,
5 September 2017

Each request requires the single json={} parameter, with contents as
described.

Common Parameters:

 - **date**: REQUIRED on all requests; formatted as YYYYMMDDHHMMSS in GMT
 - **login**: optional unless authentication required on call
 - **pword**: optional unless authentication required on call
 - **fmt**: optional, defaults to `json`

Authentication is normally not required (exceptions below), but
unauthenticated requests will not return private information.

| Table of Contents  |
| ------------------ |
| [1. Querying Projects](#1-querying-projects) |
| [1.1 List projects with basic information](#11-list-projects-with-basic-information) |
| [1.2 List projects with descriptions](#12-list-projects-with-descriptions) |
| [2. Querying Receivers](#2-querying-receivers) |
| [2.1 List receivers](#21-list-receivers) |
| [2.2 List Receiver Deployments (nested version)](#22-list-receiver-deployments-nested-version) |
| [2.3 List Receiver Deployments (flat version: json or csv)](#23-list-receiver-deployments-flat-version-json-or-csv) |
| [2.4 List Receiver Antennas](#24-list-receiver-antennas) |
| [3. Querying Tags](#3-querying-tags) |
| [3.1 List Tags](#31-list-tags) |
| [3.2 Search Tag Deployments](#32-search-tag-deployments) |
| [4. Miscellaneous Querying](#4-miscellaneous-querying) |
| [4.1 Validating a user](#41-validating-a-user) |
| [4.2 List Species](#42-list-species) |
| [4.3 List receiver status](#43-list-receiver-status) |
| [5. Registering Tags, Receivers and Projects](#5-registering-tags-receivers-and-projects) |
| [5.1 Register a tag](#51-register-a-tag) |
| [5.2 Register a receiver (sensor)](#52-register-a-receiver-sensor) |
| [5.3 Register a project](#53-register-a-project) |
| [5.4 Deploy a tag](#54-deploy-a-tag-untested) |
| [5.5 Undeploy a tag](#55-undeploy-a-tag-untested) |
| [Changelog](#changelog) |

## 1. Querying Projects ##

### 1.1 List projects with basic information ###

    /api/projects
(aliases: /api/project, /api/projects/list, /api/project/list)

**Parameters:**
 - **fmt**: string, defaults to "json", accepts "jsonp" and "csv"
 - **showAll**: boolean, defaults to false; when false and the user is authenticated shows full information on projects the user is authorized for, otherwise shows third-party-visible information on all projects
 - **login**: string, optional
 - **pword**: string, optional

 **Returns:**

 JSON-formatted object with these fields:

  - **version**: string; version number, e.g. "2.0"
  - **data**: array of objects with these fields:
    - **id**: integer; motus project ID
    - **name**: string; descriptive name of project
    - **code**: string; short project code, e.g. for plots
    - **tagPermissions**: integer, 0..3; permission level for tag registration (low # means restricted, high # means permissive)
    - **sensorPermissions**: integer, 0..3; permission level for receiver registration (low # means restricted, high # means permissive)

e.g.

``` json
{
    "version":"2.0",
    "data":[
        {
            "id":1,
            "name":"Motus Ontario Array",
            "code":"Motus",
            "tagPermissions":2,
            "sensorPermissions":2
        },
        {
            "id":2,
            "name":"Motus Atlantic Array",
            "code":"MotusATL",
            "tagPermissions":2,
            "sensorPermissions":2
        }
    ]
}
```

### 1.2 List projects with descriptions ###

    /api/projects/descriptions
(alias: /api/project/descriptions)

**Parameters**:
 - **fmt**: string, defaults to "json", accepts "jsonp" and "csv"
 - **showAll**: boolean, defaults to false; when false and the user is authenticated shows full information on projects the user is authorized for, otherwise shows third-party-visible information on all projects
 - **login**: string, optional
 - **pword**: string, optional

**Returns:**

 JSON-formatted object with these fields:

  - **version** string; version number, e.g. "2.0"
  - **data** data; array of objects with these fields:
    - **projectID**; integer; motus project ID
    - **projectName**; string; descriptive name of project
    - **projectCode**; string; short project code, e.g. for plots
    - **descriptionShort**; string; short summary of the project (a few sentences)
    - **descriptionLong**; string; long summary of the project (a few paragraphs)

e.g.

``` json
{
    "version":"2.0",
    "data":[
        {
            "projectID":1,
            "projectName":"Motus Ontario Array",
            "projectCode":"Motus",
            "descriptionShort":"Array of 70+ towers maintained by Bird Studies Canada in support of all projects.",
            "descriptionLong":"Array of 70+ towers maintained by Bird Studies Canada in support of all projects."
        },
        {
            "projectID":2,
            "projectName":"Motus Atlantic Array",
            "projectCode":"MotusATL",
            "descriptionShort":"Array of 50+ towers maintained by Bird Studies Canada in support of all projects.",
            "descriptionLong":"Array of 50+ towers maintained by Bird Studies Canada in support of all projects."
        }
    ]
}
```

## 2. Querying Receivers ##

### 2.1 List receivers ###

    /api/receivers
(aliases: /api/receivers/list, /api/recv, /api/recv/list, /api/receiver, /api/receiver/list)

**Parameters:**
 - **date**: string, required, "YYYYMMDDhhmmss" UTC
 - **fmt**: string, defaults to "json", accepts "jsonp" or "csv"
 - **login**: string, optional, some receivers are only visible to authenticated users
 - **pword**: string, optional
 - **projectID**: integer, optional, only return receivers from the given project (and users authorized with the project may be able to see more than is visible to third parties)
 - **status**: integer, optional, only return receivers whose most recent deployments have the same status code (0|1|2)

**Returns:**

 JSON-formatted object with these fields:

  - **version**: string, version number, e.g. "2.0"
  - **data**: array of objects with these fields:
    - **receiverID**: string, receiver serial number
    - **motusRecvID**: integer
    - **deviceID**: integer
    - **macAddress**: string (currently always "0")
    - **receiverType**: string
    - **dtStart**: string, most recent deployment start date/time
    - **deploymentStatus**: string
    - **deploymentName**: string

e.g.

``` json
{
    "version":"2.0",
    "data":[
        {
            "receiverID":"SG-5113BBBK0173",
            "motusRecvID":383,
            "recvProjectID":1,
            "deviceID":251,
            "macAddress":"0",
            "receiverType":"SENSORGNOME",
            "dtStart":"2016-11-19 00:00:00 +00:00",
            "deploymentStatus":"active",
            "deploymentName":"Crysler Park Marina"
        },
        {
            "receiverID":"SG-5113BBBK0324",
            "motusRecvID":384,
            "recvProjectID":1,
            "deviceID":252,
            "macAddress":"0",
            "receiverType":"SENSORGNOME",
            "dtStart":"2016-11-22 00:00:00 +00:00",
            "deploymentStatus":"terminated",
            "deploymentName":"Binbrook_Conservation_Area"
        }
    ]
}
```

### 2.2 List Receiver Deployments (nested version) ###

This version includes JSON nested structures for individual antennas (csv is
not supported)

    /api/receiver/sensordeployments

Additional parameters:

 - **projectID**: filter by a single project ID
 - **year**: filter by year deployment started
 - **serialNo**: filter by receiverID
 - **status**: filter by deployment status (0|1|2)

Returned fields:

 - id: integer; unknown motus ID #
 - serno: string; receiver serial number; e.g. Lotek-123, SG-1234BBBK4321
 - receiverType: string; SENSORGNOME, LOTEKSRX600, LOTEKSRX800, ...
 - deviceID: integer; motus ID # for this device
 - macAddress: string; not used
 - status: string; "active", "terminated", ...?
 - deployID: integer; ID for this deployment
 - name: string; (short) name for this deployment, e.g. for labelling graphs
 - fixtureType; string
 - latitude; double; decimal degrees N
 - longitude; double; decimal degrees E
 - elevation; double; metres ASL
 - isMobile; boolean
 - tsStart; double; deployment start timestamp, in seconds since 1 Jan 1970 GMT
 - antennas; list with one row per antenna and these fields:
    - port; integer
    - antennaType; string
    - bearing; double; degrees clockwise from local magnetic north
    - heightMeters; double; height of antenna mast above elevation quoted above
    - cableLengthMeters; double; length of coax cable between antenna and radio

### 2.3 List Receiver Deployments (flat version: json or csv) ###

    /api/receiver/deployments

Additional parameters:

 - **projectID**: filter by a single project ID
 - **year**: filter by year deployment started
 - **serialNo**: filter by receiverID
 - **status**: filter by deployment status (0|1|2)

### 2.4 List Receiver Antennas ###

    /api/receiver/antennas

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
 - **searchMode**: set deployment period search mode (`startsBetween`|`overlap`)
 - **tsStart**: unix timestamp, start of deployment
 - **tsEnd**: unix timestamp, end of deployment
 - **tsLastModified**: unix timestamp, threshold for change of metadata; if specified, the reply
   only includes records for which metadata were changed at or after this time.  This is
   intended to allow on-demand update of tag metadata records by the data processing server.

Reply has these fields:
 - tagID
 - tagProjectID **this is called `projectID` in v 1.0**
 - mfgID
 - dateBin
 - lifespan
 - tagDeployID
 - deploymentStatus
 - dtStart
 - dtEnd
 - dtStartAnticipated
 - tsStart
 - tsEnd
 - tsStartAnticipated
 - deferSec
 - speciesID
 - speciesName
 - motusScientificName
 - motusEnglishName
 - motusFrenchName
 - bandNumber
 - markerNumber
 - markerType
 - latitude
 - longitude
 - elevation
 - comments

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

**Parameters**:

**Name** | **Required** | **Default value** | **Description**
---|---|---|---
fmt | No | false | Supported output formats: json, jsonp
projectID | Yes | n/a | Motus numeric project ID.
mfgID | Yes | n/a | ID printed on tag by manufacturer. If a single project is registering more than one tag with the same mfgID, (e.g. three tags with mfgID=123), the 2nd tag is called 123.1, the third tag is called 123.2, and so on. (i.e. the first tag is 123.0 = 123). For most tags, the only additional labeling we can reasonably expect users to attach is one or more indelible dots. So we tell people to add one dot to the first tag duplicating a given ID, two dots to the second tag duplicating a given ID, and so on.  This should be a character string, as some later tags will have manufacturer IDs consisting of 8 hexadecimal digits.
dateBin | Yes | n/a | Field representing a definite time period (e.g. a quarter) of when the tag registration occurred, and that is mean to discriminate tags with the same mfgID across years or seasons within the same project. For now, this is done by combining the year (YYYY) and quarter (1-4) of the time of registration (e.g. 2015-3), but this may change to some other rule later (e.g. YYYY-MM). This field should be generated by a script based on registration time, and not be user-provided.
type | No | n/a | Type of tag (BEEPER or ID)
codeSet | No | n/a | If a manufacturer has more than one code set from which a code can come, this must uniquely identify the one for this tag. (e.g. 'Lotek-3')
manufacturer | No | n/a | Name of manufacturer (e.g. Lotek)
model | No | n/a | Tag model name
lifeSpan | No | n/a | Expected lifespan in days (integer)
nomFreq | No | 166.380 | Nominal transmitter frequency (MHz). This is typically printed on the tag by the manufacturer, as the kilohertz portion of the frequency (e.g. 380 in the case of a tag @ 166.380).
offsetFreq | No | n/a | Offset frequency (kHz)
period | No | n/a | Repeat period (seconds)
periodSD | No | n/a | standard deviation of repeat period (seconds)
pulseLen | No | n/a | Pulse length (seconds)
param1-8 | No | n/a | Up to 8 additional parameters (float numbers). Their definitions depend on the paramType field.
paramType | No | n/a | Numeric value referring to the metadata description of the (up to) 8 additional parameters param1-8. This field is not strictly required, but is expected to be provided whenever values are entered in param1-8. There is however no validation to ensure that the values in param1-8 make sense in relation to the paramType value. If you are unsure what to use here, and would like to make use of the param1-8 fields, please contact the database or API administrator.
ts | No | n/a | Timestamp at which registration occurred. This will only be useful if the registration was routed via the sensorgnome.org server because the user provided a .WAV file via email or upload, rather than using the upcoming self-registration web page.


### 5.2 Register a receiver (sensor) ###

    /api/receiver/register

Additional parameters:

 - **secretkey**: sensor hash key
 - **receiverType**: eg: LOTEKSRX800
 - **serno**: the receiver id (eg: `SG-1234BBBK678`, `Lotek-153`)
 - **userID**: the motus id of the user whose sensor is being processed
 - **projectID**: the project id to which this sensor will be deployed
 - ~~**hash**: Calculate the hex encoded sha1 hash of  (serno + underscore + date + underscore + sharedMasterSecret)~~
     no longer used as of 2017 Dec 15 (beta server)

Note that userID and projectID are not mandatory, but are highly encouraged!

### 5.3 Register a project ###

    /api/project/register

Additional parameters:

 - **projectName**: a name for the project
 - **projectCode**: a short code for the project (not available yet on live
server)

### 5.4 Deploy a tag (untested) ###

    /api/tag/deploy

**Name** | **Required** | **Default value** | **Description**
---|---|---|---
fmt | No | false | Supported output formats: json, jsonp
tagID | Yes | n/a | Unique numeric ID assigned to the tag, provided at the time of tag registration.
status | Yes | pending | Status from one of the possible following values: **pending, deploy, terminate**. “pending” indicates that the deployment record isn’t ready to be finalized for deployment yet. “deploy” indicates that the deployment can be activated. “terminate” will attempt to finalize a deployment by providing an end date. Note that deployments can only be activated or terminated if all required information has been provided. Attempts to deploy or terminate an incomplete deployment will return an error. At the moment, several of the parameters can no longer be modified once a deployment is activated, and the status cannot be changed back to an earlier status (permitted sequence is pending -&gt; deploy -&gt; terminate, but levels can be omitted).
tsStart | Yes | n/a | Timestamp for start of deployment (seconds since '1970-01-01T00:00:00'). If the tag has a deferred time lag (deferTime), this is the time at which the bird is released, in such a way that the time when the tag is expected to be active is given by tsStart + deferTime.
tsEnd | No\* | n/a | Timestamp for end of deployment (seconds since '1970-01-01T00:00:00'). Required for status=terminate.
deferTime | No | 0 | Defer time (in seconds from tsStart). Some tags are capable of deferred activation - they don't start transmitting until some number of seconds (can be very large) after activation.
speciesID | No | n/a | Numeric ID (integer) of the species on which the tag is being deployed. You should refer to the listspecies API entry point to query species ID based on codes or names.
speciesName | No | n/a | Name (String, preferably scientific name, but can also be common name or species code) of the species on which the tag is being deployed. There is no validation on this field, including whether it matches the speciesID value. This is strictly used as a guide for users looking at data records.
markerType | No | n/a | Type of marker (e.g. “metal band”, “color band”)
markerNumber | No | n/a | Marker number or descriptor (e.g. “1234-56789” or “L:Red/Blue,R:Metal 1234-56789”)
lat | No | n/a | Latitude (decimal degrees) of the deployment site. E.g. 45.123.
lon | No | n/a | Longitude (decimal degrees) of the deployment site. E.g. -60.325.
elev | No | n/a | Elevation above sea level (meters) of the deployment site. E.g. 23.
ts | Yes | n/a | Timestamp (seconds since '1970-01-01T00:00:00'). Time at which deployment information was generated.
comments | No | n/a | User comments related to the deployment, for their own use.
properties | No | n/a | JSON object containing pairs of name/value pairs for any custom properties for the tag. E.g. \[{"AgeID":"HY", "blood":"N", "Country":"Canada", "Include\_in\_analysis":"Y", "LocationID":"Niapiskau", "Province":"Quebec","SexID":"U"}\]. Values are currently limited to 128 chars for names and unlimited for values. Each property name must be unique.

### Output (HTTP response codes: 201) ###

  **Name** | **Description** | **Example values**
  ---|---|---
  importID  |     Numeric import ID assigned to the imported data record. Only deployments submitted with valid credentials immediately receive a deployment ID. Others will have a value of “pending” until they are approved within the submitted project by a principal investigator. |  1234
  deploymentID |  Deployment ID assigned to the new deployment. Only deployments submitted with valid credentials immediately receive a deployment ID. Others will have a value of “pending” until they are approved within the submitted project by a principal investigator.           |  2605, pending

### 5.5 Undeploy a tag (untested) ###

    /api/tag/undeploy

See api version 1.0 documentation for parameters (deletetagdeployment)

## Changelog ##

2018-09-14 add `tsLastModified` parameter to `api/tags/search`

2018-08-24 correct parameter name and possible values for `/api/tags/search`

2018-02-06 document `/api/receiver/sensordeployments`, which no longer returns projectID.
