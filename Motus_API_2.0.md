# Motus Version 2.0 API #

Each request requires the single json={} parameter, with contents as
described.

When returning JSON ("fmt":"json" or "fmt":"jsonp"), returns a JSON object with:
 - **version**: string, the API version number, e.g. "2.0"

When returning multiple records, also returns:
 - **data**: an array of objects

When returning CSV ("fmt":"csv"), returns a CSV with the same column names as the properties of the JSON objects in the **data** array, except *all columns with non-atomic values are dropped*. Note that errors are always returned as JSON, no matter what format is specified.

| Table of Contents  |
| ------------------ |
| [1. Querying Projects](#1-querying-projects) |
| [1.1 List projects with basic information](#11-list-projects-with-basic-information) |
| [1.2 List projects with descriptions](#12-list-projects-with-descriptions) |
| [2. Querying Receivers](#2-querying-receivers) |
| [2.1 List receivers](#21-list-receivers) |
| [2.2 List Receiver Deployments (nested version)](#22-list-receiver-deployments-nested-version) |
| [2.3 List Receiver Deployments (flat version)](#23-list-receiver-deployments-flat-version) |
| [2.4 List Receiver Antennas](#24-list-receiver-antennas) |
| [3. Querying Tags](#3-querying-tags) |
| [3.1 List Tags](#31-list-tags) |
| [3.2 Search Tag Deployments](#32-search-tag-deployments) |
| [4. Miscellaneous Querying](#4-miscellaneous-querying) |
| [4.1 Validating a user](#41-validating-a-user) |
| [4.2 List Species](#42-list-species) |
| [4.3 List receiver status](#43-list-receiver-status) |
| [5. Registering Tags, Receivers and Projects](#5-registering-tags-receivers-and-projects) |
| [5.1 Register a project](#51-register-a-project) |
| [5.2 Register a receiver](#52-register-a-receiver) |
| [5.3 Deploy a receiver](#53-deploy-a-receiver) |
| [5.4 Register a tag](#54-register-a-tag) |
| [5.5 Deploy a tag](#55-deploy-a-tag) |
| [5.6 Undeploy a tag](#56-undeploy-a-tag) |
| [Changelog](#changelog) |

## 1. Querying Projects ##

### 1.1 List projects with basic information ###

    /api/projects
(aliases: /api/project, /api/projects/list, /api/project/list)

**Parameters:**

| Name | Parameter Type | Value Type | Description |
| ---- | -------------- | ---------- | ----------- |
| **fmt** | Default | String | Default is "json". Accepts "jsonp" or "csv". |
| **showAll** | Default | Boolean | Default is 'false'. When false *and* the user is authenticated, shows full information on projects the user is authorized for, otherwise shows limited information on all projects. |
| **login** | Optional | String |  |
| **pword** | Optional | String |  |

 **Returns:**

  - **id**: integer; motus project ID
  - **name**: string; descriptive name of project
  - **code**: string; short project code, e.g. for plots
  - **tagPermissions**: integer, 0..3; permission level for tag registration (low # means restricted, high # means permissive)
  - **sensorPermissions**: integer, 0..3; permission level for receiver registration (low # means restricted, high # means permissive)

Example:

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

| Name | Parameter Type | Value Type | Description |
| ---- | -------------- | ---------- | ----------- |
| **fmt** | Default | String | Default is "json". Accepts "jsonp" or "csv". |
| **showAll** | Default | Boolean | Default is 'false'. When false *and* the user is authenticated, shows full information on projects the user is authorized for, otherwise shows limited information on all projects. |
| **login** | Optional | String |  |
| **pword** | Optional | String |  |

**Returns:**

 - **projectID**: integer, motus project ID
 - **projectName**: string, descriptive name of project
 - **projectCode**: string, short project code, e.g. for plots
 - **descriptionShort**: string, short summary of the project (a few sentences)
 - **descriptionLong**: string, long summary of the project (a few paragraphs)

Example:

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

| Name | Parameter Type | Value Type | Description |
| ---- | -------------- | ---------- | ----------- |
| **date** | Required | String | "YYYYMMDDhhmmss" UTC |
| **fmt** | Default | String | Default is "json". Accepts "jsonp" or "csv". |
| **login** | Optional | String | Some receivers are only visible to authenticated users. |
| **pword** | Optional | String |  |
| **projectID** | Optional | Integer | Only return receivers from the given project (and users authorized with the project may be able to see more than is visible to third parties). |
| **status** | Optional | Integer | Only return receivers whose most recent deployments have the given status code (0, 1, 2). |

**Returns:**

 - **receiverID**: string, receiver serial number
 - **motusRecvID**: integer
 - **deviceID**: integer
 - **macAddress**: string (currently always "0")
 - **receiverType**: string
 - **dtStart**: string, most recent deployment start date/time
 - **deploymentStatus**: string
 - **deploymentName**: string

Example:

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

This version includes JSON nested structures for individual antennas.

    /api/receivers/sensordeployments
(aliases: /api/recv/sensordeployments, /api/receiver/sensordeployments)

**Parameters:**

| Name | Parameter Type | Value Type | Description |
| ---- | -------------- | ---------- | ----------- |
| **date** | Required | String | "YYYYMMDDhhmmss" UTC |
| **fmt** | Default | String | Default is "json". Accepts "jsonp" or "csv". |
| **login** | Optional | String | Some receiver information is only visible to authenticated users. |
| **pword** | Optional | String |  |
| **projectID** | Optional | Integer | Only return receiver information from the given project (and users authorized with the project may be able to see more than is visible to third parties). |
| **year** | Optional | Integer | Only return deployments which ran during at least part of the given year. |
| **sensorID** | Optional | Integer | Only return information about the receiver with the given Motus receiver ID. |
| **serialNo** | Optional | String | Only return information about the receiver with the given serial number. |
| **macAddress** | Optional | String | Only return information about the receiver with the given MAC address. |

**Returns:**

 - **id**: integer, Motus receiver ID
 - **serno**: string, receiver serial number, e.g. Lotek-123, SG-1234BBBK4321
 - **receiverType**: string, e.g. SENSORGNOME, LOTEKSRX600, LOTEKSRX800, etc.
 - **deviceID**: integer, Motus ID # for this device
 - **macAddress**: string, not used
 - **status**: string, "active", "terminated", or "pending"
 - **deployID**: integer, Motus ID for this deployment
 - **name**: string, (short) name for this deployment, e.g. for labelling graphs
 - **fixtureType**: string
 - **latitude**: double, decimal degrees N
 - **longitude**: double, decimal degrees E
 - **elevation**: double, metres ASL
 - **isMobile**: boolean
 - **tsStart**: double, deployment start timestamp, in seconds since 1st Jan. 1970 UTC
 - **tsEnd**: double, deployment finish timestamp, in seconds since 1st Jan. 1970 UTC
 - **antennas**: array of objects with these fields:
   - **port**: integer
   - **antennaType**: string
   - **bearing**: double, degrees clockwise from local magnetic north
   - **heightMeters**: double, height of antenna mast above elevation quoted above
   - **cableLengthMeters** double; length of coax cable between antenna and radio

Example:

``` json
{
    "version":"2.0",
    "data":[
        {
            "id":427,
            "serno":"SG-5113BBBK3139",
            "receiverType":"SENSORGNOME",
            "deviceID":337,
            "macAddress":"0",
            "status":"terminated",
            "deployID":629,
            "name":"Werden",
            "fixtureType":"Tower",
            "latitude":42.75463,
            "longitude":-80.27077,
            "isMobile":false,
            "tsStart":1.396656E9,
            "tsEnd":1.407456E9,
            "antennas":[
                {
                    "port":1,
                    "antennaType":"yagi-9-laird",
                    "bearing":0.0,
                    "heightMeters":6.0,
                    "cableLengthMeters":7.62
                },
                {
                    "port":2,
                    "antennaType":"yagi-9-laird",
                    "bearing":240.0,
                    "heightMeters":5.6,
                    "cableLengthMeters":7.62
                },
                {
                    "port":3,
                    "antennaType":"yagi-9-laird",
                    "bearing":120.0,
                    "heightMeters":5.2,
                    "cableLengthMeters":7.62
                }
            ]
        }
    ]
}
```

### 2.3 List Receiver Deployments (flat version) ###

    /api/receivers/deployments
(aliases: /api/receivers/template, /api/receiver/deployments, /api/receiver/template, /api/recv/deployments, /api/recv/template)

Additional parameters:

 - **projectID**: filter by a single project ID
 - **year**: filter by year deployment started
 - **serialNo**: filter by receiverID
 - **status**: filter by deployment status (0|1|2)

**Parameters:**

| Name | Parameter Type | Value Type | Description |
| ---- | -------------- | ---------- | ----------- |
| **date** | Required | String | "YYYYMMDDhhmmss" UTC |
| **fmt** | Default | String | Default is "json". Accepts "jsonp" or "csv". |
| **login** | Optional | String | Some receiver information is only visible to authenticated users. |
| **pword** | Optional | String |  |
| **projectID** | Optional | Integer | Only return receiver deployments from the given project (and users authorized with the project may be able to see more than is visible to third parties) |
| **year** | Optional | Integer | Only return deployments which started during the given year. |
| **serialNo** | Optional | String | Only return deployments of the receiver with the given serial number. |
| **mac** | Optional | String | Only return deployments of the receiver with the given MAC address. |
| **status** | Optional | Integer | Only return deployments with the same status code (0, 1, 2). |

**Returns:**

 - **motusRecvID**: integer
 - **recvProjectID**: integer
 - **receiverID**: string
 - **receiverType**: string
 - **recvDeployID**: integer
 - **deploymentStatus**: string
 - **deploymentName**: string
 - **siteName**: string
 - **dtStart**: string
 - **dtEnd**: string
 - **tsStart**: double
 - **tsEnd**: double
 - **latitude**: double
 - **longitude**: double
 - **elevation**: double
 - **isMobile**: boolean
 - **macAdress**: string
 - **locationPrecision**: string

Example:

``` json
{
    "version":"2.0",
    "data":[
        {
            "motusRecvID":383,
            "recvProjectID":1,
            "receiverID":"SG-5113BBBK0173",
            "receiverType":"SENSORGNOME",
            "recvDeployID":681,
            "deploymentStatus":"terminated",
            "deploymentName":"Crysler Park Marina",
            "siteName":"Crysler Park Marina",
            "dtStart":"2014-07-31 00:00:00 +00:00",
            "dtEnd":"2016-11-19 00:00:00 +00:00",
            "tsStart":1.4067648E9,
            "tsEnd":1.4795136E9,
            "latitude":44.93670000,
            "longitude":-75.08670000,
            "elevation":null,
            "isMobile":false,
            "macAddress":"0",
            "locationPrecision":"Exact"
        }
    ]
}
```

### 2.4 List Receiver Antennas ###

    /api/receivers/antennas
(aliases: /api/receiver/antennas, /api/recv/antennas)

**Parameters:**

| Name | Parameter Type | Value Type | Description |
| ---- | -------------- | ---------- | ----------- |
| **date** | Required | String | "YYYYMMDDhhmmss" UTC |
| **fmt** | Default | String | Default is "json". Accepts "jsonp" or "csv". |
| **login** | Optional | String | Some antenna information is only visible to authenticated users. |
| **pword** | Optional | String |  |
| **projectID** | Optional | Integer | Only return antenna information from the given project (and users authorized with the project may be able to see more than is visible to third parties). |
| **serialNo** | Optional | String | Only return antenna information from receivers with the given serial number (receiver ID). |
| **status** | Optional | Integer | Only return antenna information from receiver deployments with the given status code (0, 1, 2). |

**Returns:**

 - **motusRecvID**: integer
 - **recvProjectID**: integer
 - **receiverID**: integer
 - **recvDeployID**: integer
 - **port**: integer
 - **antennaType**: string
 - **bearing**: double
 - **magneticBearing**: double
 - **trueBearing**: double
 - **tilt**: double
 - **polBearing**: double
 - **polTilt**: double
 - **heightMeters**: double
 - **filter**: double
 - **cableLengthMeters**: double
 - **cableType**: string
 - **mountDistanceMeters**: double
 - **mountBearing**: double
 - **details**: string
 - **elevationAngle**: double
 - **polarization1**: double
 - **polarization2**: double
 - **deploymentStatus**: string

Example:

```json
{
    "version":"2.0",
    "data":[
        {
            "motusRecvID":383,
            "recvProjectID":1,
            "receiverID":"SG-5113BBBK0173",
            "recvDeployID":681,
            "port":1,
            "antennaType":"yagi-9-laird",
            "bearing":65.0,
            "magneticBearing":65.0,
            "trueBearing":51.313793,
            "tilt":null,
            "polBearing":null,
            "polTilt":null,
            "heightMeters":6.0,
            "filter":null,
            "cableLengthMeters":6.0,
            "cableType":"RG58",
            "mountDistanceMeters":null,
            "mountBearing":null,
            "details":null,
            "elevationAngle":null,
            "polarization1":null,
            "polarization2":null,
            "deploymentStatus":"terminated"
        }
    ]
}
```

## 3. Querying Tags ##

### 3.1 List Tags ###

    /api/tags
(aliaes: /api/tags/list, /api/tag, /api/tag/list)

**Parameters:**

| Name | Parameter Type | Value Type | Description |
| ---- | -------------- | ---------- | ----------- |
| **date** | Required | String | "YYYYMMDDhhmmss" UTC |
| **fmt** | Default | String | Default is "json". Accepts "jsonp" or "csv". |
| **login** | Optional | String | Some tag information is only visible to authenticated users. |
| **pword** | Optional | String |  |
| **projectID** | Optional | Integer | Only show tags from this project (and authorized users will get additional fields hidden from third-parties). |
| **year** | Optional | Integer | Only show tags deployed during the given year. |
| **mfgID** | Optional | String | Only show tags with the given manufacturer's ID (serial number). |

**Returns:**

 - **tagID**: integer
 - **tagProjectID**: integer
 - **mfgID**: string
 - **dateBin**: string
 - **dtRegistered**: string
 - **type**: string
 - **codeset**: string
 - **manufacturer**: string
 - **model**: string
 - **lifespan**: integer
 - **nomFreq**: double
 - **offsetFreq**: double
 - **period**: double
 - **periodSD**: double
 - **pulseLen**: double

Authorized users also get:

 - **param1**: double
 - **param2**: double
 - **param3**: double
 - **param4**: double
 - **param5**: double
 - **param6**: double
 - **param7**: double
 - **param8**: double
 - **paramType**: integer
 - **dtStart**: string
 - **deploymentStatus**: string
 - **motusScientificName**: string
 - **RecentTagDeployID**: integer

Example:

```json
{
    "version":"2.0",
    "data":[
        {
            "tagID":18611,
            "tagProjectID":36,
            "mfgID":"10",
            "dateBin":"2016-2",
            "dtRegistered":"2016-04-05 23:56:39.017",
            "type":"ID",
            "codeset":"Lotek4",
            "manufacturer":"Lotek",
            "model":"NTQB-1",
            "lifespan":null,
            "nomFreq":166.3,
            "offsetFreq":0.6796,
            "period":39.6896,
            "periodSD":1.0049E-14,
            "pulseLen":2.5
        }
    ]
}
```

### 3.2 Search Tag Deployments ###

    /api/tags/search
(alias: /api/tag/search)

**Parameters:**

| Name | Parameter Type | Value Type | Description |
| ---- | -------------- | ---------- | ----------- |
| **date** | Required | String | "YYYYMMDDhhmmss" UTC |
| **login** | Required | String | User must be authorized to register tags and receivers through the API. |
| **pword** | Required | String |  |
| **fmt** | Default | String | Default is "json". Accepts "jsonp" or "csv". |
| **searchMode** | Default | String | Default is "startsBetween". Also accepts "overlap" or "overlaps". |
| **defaultLifespan** | Default | Long | Default is 90 (days). Only used with "searchMode":"overlap". |
| **lifespanBuffer** | Default | Double | Default is 1.5. Factor by which to multiply estimated lifespan when "searchMode" is "overlap". |
| **projectID** | Optional | Integer | Only return tag deployments from the given project. |
| **status** | Optional | Integer | Only return tag deployments with the given status code (0, 1, 2). |
| **mfgID** | Optional | String | Only return tag deployments with the given manufacturer's ID (serial number). |
| **tsStart** | Optional | Long | When "searchMode" is "startsBetween", only returns deployments which start/end at or after the given time(s) (in seconds since 1970). |
| **tsEnd** | Optional | Long | When "searchMode" is "overlaps" and tsStart and tsEnd are both given, only returns deployments which overlap the specified time range. |

**Returns:**

 - **id**: integer, tag ID
 - **projectID**: integer
 - **mfgID**: string
 - **dateBin**: string
 - **type**: string
 - **codeSet**: string
 - **manufacturer**: string
 - **model**: string
 - **lifespan**: integer
 - **nomFreq**: double
 - **offsetFreq**: double
 - **period**: double
 - **periodSD**: double
 - **pulseLen**: double
 - **param1**: double
 - **param2**: double
 - **param3**: double
 - **param4**: double
 - **param5**: double
 - **param6**: double
 - **param7**: double
 - **param8**: double
 - **paramType**: integer
 - **deployID**: integer
 - **sdID**: integer
 - **status**: integer
 - **dtStart**: string
 - **dtEnd**: string
 - **tsStart**: double
 - **tsEnd**: double
 - **deferSec**: double
 - **speciesID**: integer
 - **speciesName**: string
 - **motusScientificName**: string
 - **motusEnglishName**: string
 - **motusFrenchName**: string
 - **bandNumber**: string
 - **markerNumber**: string
 - **markerType**: string
 - **latitude**: double
 - **longitude**: double
 - **elevation**: double
 - **comments**: string

Example:

```json
{
    "version":"2.0",
    "data":[
        {
            "id":18611,
            "projectID":36,
            "mfgID":"10",
            "dateBin":"2016-2",
            "type":"ID",
            "codeSet":"Lotek4",
            "manufacturer":"Lotek",
            "model":"NTQB-1",
            "lifeSpan":null,
            "nomFreq":166.3,
            "offsetFreq":0.6796,
            "period":39.6896,
            "periodSD":1.0049E-14,
            "pulseLen":2.5,
            "param1":null,
            "param2":null,
            "param3":null,
            "param4":null,
            "param5":null,
            "param6":null,
            "param7":null,
            "param8":null,
            "paramType":1,
            "deployID":9263,
            "sdID":null,
            "status":2,
            "dtStart":"2016-09-08 12:00:00 +00:00",
            "dtEnd":null,
            "tsStart":1.473336E9,
            "tsEnd":null,
            "deferSec":null,
            "speciesID":14280,
            "speciesName":"Poecile atricapillus",
            "motusScientificName":"Poecile atricapillus",
            "motusEnglishName":"Black-capped Chickadee",
            "motusFrenchName":"Mésange à tête noire",
            "bandNumber":null,
            "markerType":"metal band",
            "latitude":45.30621000,
            "longitude":-75.81655300,
            "elevation":null,
            "comments":null,
            "dateLastModified":"2017-08-14 17:28:24.133"
        }
    ]
}
```

## 4. Miscellaneous Querying ##

### 4.1 Validating a user ###

Check that credentials are valid. If so, return basic user information
and a list of projects that the user can access.

    /api/user/validate
(alias: /api/users/validate)

**Parameters:**

| Name | Parameter Type | Value Type | Description |
| ---- | -------------- | ---------- | ----------- |
| **date** | Required | String | "YYYYMMDDhhmmss" UTC |
| **login** | Required | String |  |
| **pword** | Required | String |  |
| **fmt** | Default | String | Default is "json". Accepts "jsonp". |

**Returns:**
 - **userID**: integer
 - **emailAddress**: string
 - **userType**: string
 - **projects**: object, a map from project ID to project name containing all the projects the user is authorized to read

Example:

```json
{
    "version":"2.0",
    "userID":161,
    "emailAddress":"example@example.com",
    "projects":{
        "1":"Motus Ontario Array",
        "2":"Motus Atlantic Array"
    },
    "userType":"collaborator"
}
```

### 4.2 List Species ###

    /api/species
(alias: /api/lookup/species)

**Parameters:**

| Name | Parameter Type | Value Type | Description |
| ---- | -------------- | ---------- | ----------- |
| **date** | Required | String | "YYYYMMDDhhmmss" UTC |
| **fmt** | Default | String | Default is "json". Accepts "jsonp" and "csv". |
| **nrec** | Default | Integer | Default is 40000. Accepts int between 0 and 40000 inclusive. Maximum number of records to return. |
| **qlang** | Default | String | Default is "EN", accepts "FR", "SC", "CD". Field to query: ENglish, FRench, SCientific, 4-letter species CoDe (latter not returned). |
| **qstr** | Optional | String | Query string. |
| **group** | Optional | String | Return only records from the given group. |

**Returns:**
 - **id**: long, species ID
 - **scientific**: string
 - **english**: string
 - **french**: string
 - **group**: string
 - **sort**: integer

Example:
```json
{
    "version":"2.0",
    "data":[
        {
            "scientific":"Calidris canutus",
            "english":"Red Knot",
            "id":4670,
            "sort":3172,
            "french":"Bécasseau maubèche",
            "group":"BIRDS"
        },
        {
            "scientific":"Myotis lucifugus",
            "english":"Little Brown Bat",
            "id":100430,
            "sort":100430,
            "french":"petite chauve-souris brune",
            "group":"BATS"
        }
    ]
}
```

### 4.3 List receiver status ###

    /api/receiverstatus
(alias: /api/lookup/receiverstatus)

**Parameters:**

| Name | Parameter Type | Value Type | Description |
| ---- | -------------- | ---------- | ----------- |
| **date** | Required | String | "YYYYMMDDhhmmss" UTC |
| **fmt** | Default | String | Default is "json". Accepts "jsonp" and "csv". |

**Returns:**
 - **id**: integer, status code
 - **name**: string, status

Example:
```json
{
    "version":"2.0",
    "data":[
        {
            "id":"0",
            "name":"pending"
        },
        {
            "id":"1",
            "name":"terminated"
        },
        {
            "id":"2",
            "name":"active"
        }
    ]
}
```

## 5. Registering Tags, Receivers and Projects ##

### 5.1 Register a project ###

    /api/project/register
(alias: /api/projects/register)

**Paramters:**

| Name | Parameter Type | Value Type | Description |
| ---- | -------------- | ---------- | ----------- |
| **date** | Required | String | "YYYYMMDDhhmmss" UTC |
| **login** | Required | String | User must be PI or administrator. |
| **pword** | Required | String | |
| **projectName** | Required | String | Full name of the new project. |
| **projectCode** | Required | String | Short code for the project (e.g. for labelling tags and graphs). |
| **fmt** | Default | String | Default is "json". |

**Returns:**
 - **projectID**: integer, ID of the newly created project

Example:
```json
{
    "version":"2.0",
    "projectID":218
}
```

### 5.2 Register a receiver ###

    /api/receiver/register
(aliases: /api/receivers/register, /api/recv/register)

**Parameters:**

| Name | Parameter Type | Value Type | Description |
| ---- | -------------- | ---------- | ----------- |
| **date** | Required | String | "YYYYMMDDhhmmss" UTC |
| **login** | Required | String | User must be an administrator. |
| **pword** | Required | String | |
| **secretKey** | Required | String | Receiver hash key. |
| **serno** | Required | String | Receiver ID, receiver serial number (e.g. `SG-1234BBBK5678`, `Lotek-123`). |
| **hash** | Required | String | Calculate the hex encoded sha1 hash of  (serno + underscore + date + underscore + sharedMasterSecret) |
| **fmt** | Default | String | Default is "json". Accepts "jsonp". |
| **projectID** | Optional | Integer | The project to which this receiver will be assigned. |
| **macAddress** | Optional | String | |
| **receiverType** | Optional | String | E.g. `LOTEKSRX800`. |
| **userID** | Optional | Integer | The Motus ID of the user whose receiver is being registered. |

Note that although userID and projectID are not mandatory, they are highly encouraged!

**Returns:**
 - **deviceID**: integer
 - **result**: string, one of "registered", "updated", "serial-exists"

Example:
```json
{
    "version":"2.0",
    "deviceID":1000,
    "result":"serial-exists"
}
```

### 5.3 Deploy a receiver ###

    /api/receiver/deploy
(aliases: /api/receivers/deploy, /api/recv/deploy)

**Parameters:**

| Name | Parameter Type | Value Type | Description |
| ---- | -------------- | ---------- | ----------- |
| **date** | Required | String | "YYYYMMDDhhmmss" UTC |
| **projectID** | Required | Integer | |
| **serno** | Required | String | receiver serial number |
| **hash** | Required | String | |
| **status** | Default | String | Default is "pending". Accepts "deploy", "test", "terminate".
| **fmt** | Default | String | Default is "json". Accepts "jsonp". |
| **sensors** | Optional | Array of Objects | See below for details. |
| **siteCode** | Optional | String | |
| **siteDesc** | Optional | String | |
| **siteNotes** | Optional | String | |
| **landOwnerID** | Optional | Integer | |
| **fixtureType** | Optional | String | |
| **forceTerminate** | Optional | Boolean | |
| **lat** | Optional | Double | |
| **lon** | Optional | Double | |
| **elev** | Optional | Double | |
| **isMobile** | Optional | Boolean | |
| **ts** | Optional | Long | |
| **tsStart** | Optional | Long | |
| **tsEnd** | Optional | Long | |

Properties of the objects in the parameter **sensors**:
 - **port**: integer
 - **type**: string, accepts "radio" or "gps"
 - **params**: object

Properties of **params** when **type** == "radio":
 - **radio**: string
 - **antenna**: string
 - **bearing**: double
 - **tilt**: double
 - **polBearing**: double
 - **polTilt**: double
 - **height**: double
 - **filter**: string
 - **cableLength**: double
 - **cableType**: string
 - **mountDistance**: double
 - **mountBearing**: double
 - **details**: string

Properties of **params** when **type** == "gps":
 - **model**: string
 - **pps**: boolean

**Returns:**
 - **importID**: integer
 - **responseCode**: string
 - **sensorID**: integer
 - **deployID**: integer
 - **conflicts**: array of objects (dropped if empty)
   - **sensorID**: integer
   - **deploymentID**: integer
   - **status**: integer
   - **name**: string
   - **tsStart**: long
   - **tsEnd**: long

### 5.4 Register a tag ###

    /api/tag/register
(alias: /api/tags/register)

**Parameters**:

| Name | Parameter Type | Value Type | Description |
| ---- | -------------- | ---------- | ----------- |
| **date** | Required | String | "YYYYMMDDhhmmss" UTC |
| **login** | Required | String | User must be authorized to register tags and receivers through the API. |
| **pword** | Required | String | |
| **projectID** | Required | Integer | Motus numeric project ID. |
| **mfgID** | Required | String | ID printed on tag by manufacturer. If a single project is registering more than one tag with the same mfgID, (e.g. three tags with mfgID=123), the 2nd tag is called 123.1, the third tag is called 123.2, and so on. (i.e. the first tag is 123.0 = 123). For most tags, the only additional labeling we can reasonably expect users to attach is one or more indelible dots. So we tell people to add one dot to the first tag duplicating a given ID, two dots to the second tag duplicating a given ID, and so on. |
| **dateBin** | Required | String | Field representing a definite time period (e.g. a quarter) of when the tag registration occurred, and that is mean to discriminate tags with the same mfgID across years or seasons within the same project. For now, this is done by combining the year (YYYY) and quarter (1-4) of the time of registration (e.g. 2015-3), but this may change to some other rule later (e.g. YYYY-MM). This field should be generated by a script based on registration time, and not be user-provided. |
| **fmt** | Default | String | Default is "json". Accepts "jsonp". |
| **nomFreq** | Default | Double | Default is 166.380. Nominal transmitter frequency (MHz). This is typically printed on the tag by the manufacturer, as the kilohertz portion of the frequency (e.g. 380 in the case of a tag @ 166.380). |
| **type** | Optional | String | Type of tag (BEEPER or ID). |
| **codeSet** | Optional | String | If a manufacturer has more than one code set from which a code can come, this must uniquely identify the one for this tag. (E.g. 'Lotek-3'.) |
| **manufacturer** | Optional | String | Name of manufacturer (e.g. Lotek). |
| **model** | Optional | String | Tag model name. |
| **offsetFreq** | Optional | Double | Offset frequency (kHz). |
| **period** | Optional | Double | Repeat period (seconds). |
| **periodSD** | Optional | Double | Standard deviation of repeat period (seconds). |
| **pulseLen** | Optional | Double | Pulse length (seconds). |
| **param1** - **param8** | Optional | Double | Up to 8 additional parameters. Their definitions depend on the paramType field. |
| **paramType** | Optional | Integer | Numeric value referring to the metadata description of the (up to) 8 additional parameters param1-8. This field is not strictly required, but is expected to be provided whenever values are entered in param1-8. There is however no validation to ensure that the values in param1-8 make sense in relation to the paramType value. If you are unsure what to use here, and would like to make use of the param1-8 fields, please contact the database or API administrator. |
| **extra** | Optional | String | |
| **ts** | Optional | Double | Timestamp at which registration occurred. This will only be useful if the registration was routed via the sensorgnome.org server because the user provided a .WAV file via email or upload, rather than using the upcoming self-registration web page. |

**Returns:**
 - **importID**: integer
 - **responseCode**: string
 - **tagID**: integer
 - **code**: string

Example:
```json
{
    "version":"2.0",
    "importID":45644,
    "responseCode":"success-import",
    "tagID":31910,
    "code":"success-terminate"
}
```

### 5.5 Deploy a tag ###

    /api/tag/deploy
(alias: /api/tags/deploy)

**Parameters:**

| Name | Parameter Type | Value Type | Description |
| ---- | -------------- | ---------- | ----------- |
| **date** | Required | String | "YYYYMMDDhhmmss" UTC |
| **projectID** | Required | Integer | Motus project ID of the tag. |
| **login** | Required | String | The user must be authorized to edit tags of the given project, and to register tags and receivers through the API. |
| **pword** | Required | String | |
| **tagID** | Required | Integer | Unique numeric ID assigned to the tag, provided at the time of tag registration. |
| **tsStart** | Required | Double | Timestamp for start of deployment (seconds since '1970-01-01T00:00:00'). If the tag has a deferred time lag (deferTime), this is the time at which the bird is released, in such a way that the time when the tag is expected to be active is given by tsStart + deferTime. |
| **ts** | Required | Double | Timestamp (seconds since '1970-01-01T00:00:00'). Time at which deployment information was generated. |
| **fmt** | Default | String | Default is "json". Accepts "jsonp". |
| **speciesID** | Optional | Integer | ID of the species on which the tag is being deployed. You should refer to the listspecies API entry point to query species ID based on codes or names. |
| **speciesName** | Optional | String | Name (preferably scientific name, but can also be common name or species code) of the species on which the tag is being deployed. There is no validation on this field, including whether it matches the speciesID value. This is strictly used as a guide for users looking at data records. |
| **markerType** | Optional | String | Type of marker (e.g. “metal band”, “color band”). |
| **markerNumber** | Optional | String | Marker number or descriptor (e.g. “1234-56789” or “L:Red/Blue,R:Metal 1234-56789”). |
| **deferTime** | Optional | Double | Defer time (in seconds from tsStart). Some tags are capable of deferred activation - they don't start transmitting until some number of seconds (can be very large) after activation. |
| **comments** | Optional | String | User comments related to the deployment, for their own use. |
| **tagProps** | Optional | String | JSON object containing pairs of name/value pairs for any custom properties for the tag. E.g. \[{"AgeID":"HY", "blood":"N", "Country":"Canada", "Include\_in\_analysis":"Y", "LocationID":"Niapiskau", "Province":"Quebec","SexID":"U"}\]. Values are currently limited to 128 chars for names and unlimited for values. Each property name must be unique. |
| **lat** | Optional | Double | Latitude (decimal degrees) of the deployment site. E.g. 45.123. |
| **lon** | Optional | Double | Longitude (decimal degrees) of the deployment site. E.g. -60.325. |
| **elev** | Optional | Double | Elevation above sea level (meters) of the deployment site. E.g. 23. |
| **tsEnd** | Optional | Double | Timestamp for end of deployment (seconds since '1970-01-01T00:00:00'). Required for status=terminate. |

**Returns:**
 - **importID**: integer
 - **responseCode**: string

Example:
```json
{
    "version":"2.0",
    "importID":1,
    "responseCode":"success-deploy"
}
```

### 5.6 Undeploy a tag ###

    /api/tag/undeploy
(alias: /api/tags/undeploy)

**Parameters:**

| Name | Parameter Type | Value Type | Description |
| ---- | -------------- | ---------- | ----------- |
| **date** | Required | String | "YYYYMMDDhhmmss" UTC |
| **projectID** | Required | Integer | Motus project ID of the tag. |
| **login** | Required | String | The user must be authorized to edit tags of the given project, and to register tags and receivers through the API. |
| **pword** | Required | String | |
| **deployID** | Required | Integer | Motus ID of the deployment to delete. |
| **tagID** | Required | Integer | Unique numeric ID assigned to the tag, provided at the time of tag registration. |
| **fmt** | Default | String | Default is "json". |

**Returns:**
 - **deployID**: integer

Example:
```json
{
    "version":"2.0",
    "deployID":31910,
}
```

## Changelog ##

This is the changelog for the API (i.e. when entrypoints change), not the changelog for this documentation.

2018-10-12 remove `lifeSpan` parameter from `/api/tag/register` and `/api/tag/deploy`

2018-09-14 add `tsLastModified` parameter to `/api/tags/search`

2018-08-24 correct parameter name and possible values for `/api/tags/search`

2018-02-06 `/api/receiver/sensordeployments` no longer returns projectID.
