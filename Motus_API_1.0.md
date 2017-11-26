# Motus API extensions \
**Bird Studies Canada**\
1 April 2016 #

Contents {#contents .ContentsHeading}
========

Contents [1](#contents)

1 Authentication issues [3](#authentication-issues)

1.1 Hash Key (hash) [3](#hash-key-hash)

1.2 Sensor serial number (serno) [3](#sensor-serial-number-serno)

1.3 Date (date) [3](#date-date)

1.4 Motus login and password (login, pword)
[3](#motus-login-and-password-login-pword)

2 Parameters [4](#parameters)

3 Response and error codes [5](#response-and-error-codes)

4 API Entry Points [7](#api-entry-points)

4.1 List of entry points [7](#list-of-entry-points)

4.2 List of projects (status: completed)
[8](#list-of-projects-status-completed)

4.3 List of sensors (status: completed)
[10](#list-of-sensors-status-completed)

4.4 List of tags (status: completed)
[12](#list-of-tags-status-completed)

4.5 Searching for tags (status: completed)
[13](#searching-for-tags-status-completed)

4.6 User validation (status: completed)
[15](#user-validation-status-completed)

4.7 User registration (Status: completed)
[16](#user-registration-status-completed)

4.8 Create new project [17](#create-new-project-status-completed)

4.9 Sensor registration (status: completed)
[18](#sensor-registration-status-completed)

4.10 Sensor deployment (status: completed?)
[19](#sensor-deployment-status-completed)

4.11 Delete sensor deployment (status: completed)
[25](#delete-sensor-deployment-status-completed)

4.12 Tag registration (status: in progress)
[26](#tag-registration-status-in-progress)

4.13 Tag deployment (status: in progress)
[29](#tag-deployment-status-in-progress)

4.14 Delete tag deployment (status: completed)
[34](#delete-tag-deployment-status-completed)

4.15 Lookup values [34](#lookup-values)

4.15.1 Lookup values: speciesID (status: completed)
[35](#lookup-values-speciesid-status-completed)

4.15.2 Lookup values: Receiver Types (status: completed)
[36](#lookup-values-receiver-types-status-completed)

4.15.3 Lookup values: Receiver Status (status: completed)
[36](#lookup-values-receiver-status-status-completed)

4.16 Debug (status: in progress) [36](#debug-status-in-progress)

Authentication issues
======================

You should anticipate sooner or later that the API will be transferred
to an https framework, and that authentication parameters (hash key,
sensor serial number and date, as well as Motus login credentials where
needed) may be required to be passed along using standard HTTP Basic
authentication
(http://en.wikipedia.org/wiki/Basic\_access\_authentication) over HTTPS
(SSL). At the moment however, all parameters should be passed as
standard parameters within the json object (see below). There are no
sessions created or maintained, and each API call must have the
appropriate authentication information.

Hash Key (hash)
---------------

The Hash key is a secret key unique to each sensor, and *must be used
with every *external* API call that requires API authentication*. It is
important that the hash key remains hidden (e.g. not visible from the
client scripts). **Manual registration files (e.g. for web upload by
users) should NOT contain the hash key.** Internal API calls can bypass
the requirement by using the isInternal parameter, indicating that the

Sensor serial number (serno)
----------------------------

Each Sensor Gnome has a unique serial number (serno, e.g.
SG-1234BBBK678). Those parameters will be required only for calls that
are expected to originate exclusively from Sensor Gnome clients. Each
serial number must be registered with Motus *before* any other API call
are made (see **register.jsp** entry point below). Those calls should
originate from the server, not the application client, as they rely on
the master application key for encryption.

Date (date)
-----------

All authenticated API calls must also contain a **UTC date string**
parameter, in the format *YYYYMMDDHHMMSS*. Note that to be valid, all
API calls must be made within a period of plus or minus 30 minutes
around the time of the date field. I.E., a unique combination of serial
number, hash and date is only valid for a period of about 30 minutes
from the date/time used (plus or minus any differences in server times).

Motus login and password (login, pword)
---------------------------------------

Motus credentials are typically required for any data uploads (POST
requests). Each person should typically have their own Motus
credentials, which have been preferably created ahead of time, as well
as assigned the necessary permissions to manage sensors and tags within
respective projects. New logins can also be created by an API call, but
new data may need to be approved by a project PI or someone with project
authority before they are accepted in the database.

Parameters
==========

All parameters are passed within a JSON object, rather than using GET or
POST parameters. For all entry points, the only parameter required is
“json”. GET entry point requests will also accept POST requests, but
entry points marked as POST should be sent accordingly. All responses
are in JSON format by default and CSV is being implemented in some data
entry point.

Example of authenticated GET request:

[**Error! Hyperlink reference not valid.**]()

Example of public GET request:

[http://motus-wts.org/data/api/v1.0/listprojects.jsp?json={“fmt”:”csv”}](http://motus-wts.org/data/api/v1.0/listprojects.jsp?json={\“fmt\)

Common parameters, recognized by all API entry points:

  ------------ -------------- ------------------- -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  **Name **    **Required**   **Default value**   **Description**

  serno        Yes            n/a                 Serial number (all uppercase)

  date         Yes            n/a                 UTC Date in YYYYMMDDHHMMSS format.

  hash         Yes            n/a                 SHA-1 hash key (uppercase hexadecimal string, e.g. A0BC67D3E4F1), which must be calculated using the serial number + “\_” + the date string + “\_” + the sensor unique secret key. The hash will also be validated on the API server side and must match the one provided as a parameter.

                                                  **Exception**: for the register.jsp entry point, the master secret key must be used, rather than the sensor specific secret key.

  isInternal   No             false               Whether the API call is made from an internal source, signaling to the API to retrieve the secret key value from the http session object called “apiAuthKey”. Make sure that the secret key is never being exposed in the client code. (TO-DO)

  login        No\*           public              Motus login. Not required for public calls (when those supported for a specific API entry point).

  pword        No\*           n/a                 Motus password. Only required in association with a Motus login, but not required for public calls.

  fmt          No             json                Output format. Only json and jsonp are supported at this time by all entry points. csv is also supported in some entry points identified below.
  ------------ -------------- ------------------- -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

Response and error codes
========================

HTTP request status codes follow w3 standards:
[*http://www.w3.org/Protocols/rfc2616/rfc2616-sec10.html*](http://www.w3.org/Protocols/rfc2616/rfc2616-sec10.html).
Generally, values in range 200-2xx indicate a successful request, values
of 400 or higher indicate an error caused by the request, and values of
500 or higher indicate an error caused by a server process (which will
typically indicate a problem within the API itself, or an broken
database connection or unavailable resources) Error status codes will be
accompanied by an “errorCode” parameter in the response object and an
errorMsg containing a more readable description. In cases where multiple
errorCode could apply, only one will be provided, typically based on the
sequence provided in the table below. No errorCode are provided for
successful requests.

The following status codes are generally those used by the Motus API:

  ---------- ----------------------------- ----------------------------------------------------------------------------------------------
  **Code**   **Java**                      **Description**
  200        SC\_OK                        request completed successfully
  201        SC\_CREATED                   data upload or action was successful
  202        SC\_ACCEPTED                  data upload was successful, but request is still being processed by server
  400        SC\_BAD\_REQUEST              server could not interpret the request, including because of invalid or missing parameters)
  401        SC\_UNAUTHORIZED              failed authentication, including invalid API key or Motus credentials)
  404        SC\_NOT\_FOUND                the entry point doesn’t exist
  409        SC\_CONFLICT                  the request could not be completed due to a conflict with the current state of the resource)
  500        SC\_INTERNAL\_SERVER\_ERROR   an unexpected error occurred, such as a database connection problem)
  ---------- ----------------------------- ----------------------------------------------------------------------------------------------

Possible errorCode values (shared by all API entry points) associated
with error response codes (note: errorMsg parameter may contain
additional details). Individual entry points may have additional error
codes, listed under their respective section below.

  -------------------------- ------------------- --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  **Value**                  **Http response**   **Description**
  missing-json               400                 Missing json object (must be provided as a GET or POST parameter called “json”).
  invalid-json               400                 Invalid json object (e.g. parsing error).
  auth-error                 401                 Invalid or missing authentication parameters
  auth-expired               401                 Date token expired (only valid for 10 minutes). Remember to use UTC dates in 24 hour format.
  login-error                401                 Invalid or missing Motus login parameters
  invalid-format             400                 Only json and jsonp formats are supported at this time
  invalid-date               400                 Missing or invalid date string (must be 24h UTC date in YYYYMMDDHHMMSS format).
  invalid-parameter-format   400                 At least one of the parameter has a formatting issue (e.g. a string is provided where a number was expected). Specific entry points may provided more specific errors to help identify which parameters are problematic.
  database-error             500                 Server error related to the database. Maybe due to a temporary connection issue, or a problem related to the SQL code.
  unknown-error              500                 Unknown server error (uncaught).
  -------------------------- ------------------- --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

1.  API Entry Points
    ================

    1.  List of entry points
        --------------------

GET
[*http://motus-wts.org/data/api/entrypoints.jsp*](http://motus-wts.org/data/api/entrypoints.jsp)

Return a list of API entry points (URL). Applications using the API can
use this to update the list of URL’s used for different functions. Each
function has a unique code (e.g. “entrypoints” or “listprojects”)

API Authentication Required: no

Additional request parameters:

  ----------- -------------- ------------------- ----------------------------------------------------------------------------------------------------
  **Name **   **Required**   **Default value**   **Description**
  fmt         No             json                Supported output formats: json, jsonp, csv
  version     No             latest (e.g. 1.0)   API version. Will always default to latest stable version, unless version is explicitly specified.

  ----------- -------------- ------------------- ----------------------------------------------------------------------------------------------------

Output (HTTP response codes: 200)

  ----------------- ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- --------------------------------------------------------------------------------------------------------------
  **Name **         **Description**                                                                                                                                                                    **Example values**

  version           API version (useful if no version is provided in the request to know the latest stable release).                                                                                   1.0

  data              Data vector containing the list of entry points, each node containing the following elements:                                                                                      {"data": \[{"code": "entrypoints",

                                                                                                                                                                                                       "url": "http: //motus-wts.org/data/api/entrypoints.jsp",

                                                                                                                                                                                                       "description": "Return a list of API entry points (URL)." }\]}

  -   code          API entry point unique code                                                                                                                                                        entrypoints, listprojects



  -   url           API entry point url                                                                                                                                                                [*http://motus­­­wts.org/data/api/entrypoints.jsp*](http://motuswts.org/data/api/entrypoints.jsp)

                                                                                                                                                                                                       [*http://motus-wts.org/data/api/v1.0/listprojects.jsp*](http://motus-wts.org/data/api/v1.0/listprojects.jsp)

  -   description   API entry point short description



  -   deprecated    A note indicating that the entry point (or some of its functionalities) is deprecated and will be discontinued in future releases, including replacement options when available.   \*\* Deprecated \*\* Use the listproject entry point instead.


  ----------------- ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- --------------------------------------------------------------------------------------------------------------

Errors specific to this entry point: none

List of projects (status: completed)
------------------------------------

GET
[*http://motus-wts.org/data/api/v1.0/listprojects.jsp*](http://motus-wts.org/data/api/v1.0/listprojects.jsp)

Return a list of Motus projects. Public requests are allowed
(login=public), and return a complete list of public projects when used
in combination with the showAll = true parameter. Authenticated requests
return a restricted list of projects that the login is authorized to
modify, depending on the type parameter. The type “both” limits the list
to projects for which the user can modify both sensors and tags (to
check? may not be all that useful anyway). Private projects are always
hidden, except when the call is authenticated with a Motus administrator
account. When showAll = true and no authentication is used, the response
will include a list of all projects (except private ones). When showAll
= true and authentication is used, the list of projects will include all
those for which the user is allowed to register and tag and/or a sensor.
When showAll = false and no authentication is used, the list of projects
returned will be limited to those that allow unauthenticated
registrations. When showAll = false and authentication is used, the list
will only include projects for which the user has edit permissions.

API Authentication Required: no

Additional request parameters:

  ----------- -------------- ------------------- ---------------------------------------------------------------------------------------------------------------------------------------
  **Name **   **Required**   **Default value**   **Description**
  fmt         No             json                Supported output formats: json, jsonp, csv
  type        No\*           n/a                 One of the following values: tag, sensor or both. Only required in association with a Motus login, but not required for public calls.
  showAll     No             false               Whether the list should include all projects. See notes above related to authentication.
  ----------- -------------- ------------------- ---------------------------------------------------------------------------------------------------------------------------------------

Output (HTTP response codes: 200)

  ----------- -------------------------------------------------------------------------------------------------- ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  **Name **   **Description**                                                                                    **Example values**

  data        Data vector containing the list of projects, each node containing the following elements:          {"data":\[{"id":1,"name":"Motus Ontario Array", "tagPermissions": 0, "sensorPermissions": 2 },{"id":2,"name":"Motus Atlantic Array", "tagPermissions": 0, "sensorPermissions": 2 }\]}

              -   id: unique project ID

              -   name: project name

              -   tagPermissions: value 0-3 indicating the level of permission for new tag registrations

              -   sensorPermissions: value 0-3 indicating the level of permission for new sensor registrations


  ----------- -------------------------------------------------------------------------------------------------- ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

Errors specific to this entry point:

  -------------- ------------------- ------------------------------------------------------------------------------
  **Value**      **Http response**   **Description**
  invalid-type   400                 The type parameter must be one of the following values: sensor, tag or both.
  -------------- ------------------- ------------------------------------------------------------------------------

List of sensors (status: completed)
-----------------------------------

GET <http://motus-wts.org/data/api/v1.0/listsensors.jsp>

Return a list of Motus sensors already registered with a project. Public
requests are allowed (should they?).

API Authentication Required: no

Additional request parameters:

  ------------ -------------- ------------------- -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  **Name **    **Required**   **Default value**   **Description**
  fmt          No             json                Supported output formats: json, jsonp, csv
  projectID    Yes            n/a                 Numeric ID of the project for which sensors are requested.
  year         No             n/a                 Year of deployment. Leaving blank will return all sensors for the project, including those that are yet to be deployed. Using a year parameter will restrict the results to only those sensors deployed in a given year.
  serialNo     No             n/a                 String containing the serial number, used to filter the results (i.e. search if a specific combination of project and serno already exists). The serialNo parameter name here is used to distinguish from the serno field used for authentication purposes.
  macAddress   No             n/a                 String containing the MAC address, used to filter the results (i.e. search if a specific combination of project and macAddress already exists).
  ------------ -------------- ------------------- -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

Output (HTTP response codes: 200)

  ----------- ------------------------------------------------------------------------------------------- ------------------------------------------------------------------------------------------------------------------------------------------------
  **Name **   **Description**                                                                             **Example values**

  data        Data vector containing the list of projects, each node containing the following elements:   {"data":\[{"id":1,"serno":" SG-1234BBBA0123", "macAddress":" 0123456789ab", "receiverType":"SENSORGNOME", "year":2015, "status":"deployed"}\]}

              -   id: unique sensor numeric ID                                                            Note: should macAddress only be available for authenticated calls? TO-DO

              -   serno: serial number of the sensor

              -   macAddress: MAC address of the sensor

              -   receiverType: e.g. SENSORGNOME

              -   year: start year of the most recent deployment, if applicable

              -   status: deployment status (pending, deployed or terminated)


  ----------- ------------------------------------------------------------------------------------------- ------------------------------------------------------------------------------------------------------------------------------------------------

Errors specific to this entry point:

  -------------------- ------------------- --------------------
  **Value**            **Http response**   **Description**
  invalid-project-id   400                 Invalid project ID
  -------------------- ------------------- --------------------

List of tags (status: completed)
--------------------------------

GET <http://motus-wts.org/data/api/v1.0/listtags.jsp>

Return a list of Motus tags already registered with a project. Public
requests are allowed (should they?).

API Authentication Required: no

Additional request parameters:

  ----------- -------------- ------------------- --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  **Name **   **Required**   **Default value**   **Description**
  fmt         No             json                Supported output formats: json, jsonp, csv
  projectID   Yes            n/a                 Numeric ID of the project for which tags are requested.
  year        No             n/a                 Year of deployment. Leaving blank will return all tags for the project, including those that are yet to be deployed. Using a year parameter will restrict the results to only those tags deployed in a given year.
  mfgID       No             n/a                 ID printed on the tag, used to filter the results (i.e. search if a specific combination of project and mfgID already exists).
  ----------- -------------- ------------------- --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

Output (HTTP response codes: 200)

  ----------- -------------------------------------------------------------------------------------------------------------------------------------------------------- ------------------------------------------------------------------------------------------------------------------------
  **Name **   **Description**                                                                                                                                          **Example values**

  data        Data vector containing the list of projects, each node containing the following elements:                                                                {"data":\[{"id":1,"mfgID":" 156.2","status":"deployed", "year":2015}, {"id":2,"mfgID":" 141.1","status":" pending"}\]}

              -   id: unique tag numeric ID

              -   mfgID: ID printed on tag by manufacturer

              -   year: start year of the most recent deployment, if applicable (tags should only exceptionally be deployed twice, even more so in different years!)


  ----------- -------------------------------------------------------------------------------------------------------------------------------------------------------- ------------------------------------------------------------------------------------------------------------------------

Errors specific to this entry point:

  -------------------- ------------------- --------------------
  **Value**            **Http response**   **Description**
  invalid-project-id   400                 Invalid project ID
  -------------------- ------------------- --------------------

Searching for tags (status: completed)
--------------------------------------

GET <http://motus-wts.org/data/api/v1.0/searchtags.jsp>

Return a vector of Motus tags already registered, with complete list of
tag and deployment properties, but excluding additional custom
properties. Public requests are allowed, but only authenticated user
requests list to specific project permissions can obtain detailed
metadata. For a simpler version of this method, which provides less
information on each tag, you can also refer to listtags.jsp.

API Authentication Required: no

Additional request parameters:

  ----------------- -------------- ------------------- --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  **Name **         **Required**   **Default value**   **Description**

  fmt               No             json                Supported output formats: json, jsonp, csv

  projectID         No             n/a                 Numeric ID of the project for which tags are requested.

  tsStart           No             n/a                 Earliest deployment timestamp (seconds since '1970-01-01T00:00:00').

                                                       With searchMode = startsBetween: Leaving blank will return all tags since the beginning. Leaving both tsStart and tsEnd blank will return all tags, including those that are yet to be deployed. Using a tsStart parameter will restrict the results to only those tags deployed after that timestamp.

                                                       With searchMode = overlaps: Both tsStart and tsEnd are required. Will return all tags that were likely active, assuming a given lifespan (either assigned to the tag, or using defaultLifespan).

  tsEnd             No             n/a                 Latest deployment timestamp (seconds since '1970-01-01T00:00:00'). Leaving blank will return all tags until current time Leaving both tsStart and tsEnd blank will return all tags, including those that are yet to be deployed. Using a tsEnd parameter will restrict the results to only those tags deployed before that timestamp.

  searchMode        No             startsBetween       Determines what type of search is being conducted. Possible values include “overlaps” (returns any tag likely still active during the period, requires both the tsStart and tsEnd value) and “startsBetween” (limits the search to tags that were deployed during the period).

  defaultLifespan   No             90                  Default expected tag lifespan in days (integer). The default value is used when the manufacturer value is not available from the database.

  lifespanBuffer    No             1.5                 Ratio by which to multiple the expected lifespan, to account for tags that may outlast the expected values (e.g. 90 days \* 1.5 = 135 days)

  regStart          No             n/a                 Earliest registration timestamp (seconds since '1970-01-01T00:00:00'). regStart and regEnd can be used to search based on the timestamp of registration of the tag, rather than the deployment date/time. Leaving blank will return all tags registered since the beginning. Leaving both regStart and regEnd blank will return all tags.

  regEnd            No             n/a                 Latest registration timestamp (seconds since '1970-01-01T00:00:00'). regStart and regEnd can be used to search based on the timestamp of registration of the tag, rather than the deployment date/time. Leaving blank will return all tags registered until the current time. Leaving both regStart and regEnd blank will return all tags.

  mfgID             No             n/a                 ID printed on the tag, used to filter the results to a specific tag.

  details           No             false               Boolean indicating that the detailed metadata properties should be included. Only allowed for user-authenticated calls and only for projects with appropriate read permissions. (TO-DO)
  ----------------- -------------- ------------------- --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

Output (HTTP response codes: 200)

  ----------- -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  **Name **   **Description**                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                              **Example values**
  data        Data vector containing the list of tags, each node containing the following fields (only non-null values are included; refer to registertags and deploytags for an explanation of the fields): tagID, projectID, mfgID, type, codeSet, manufacturer, model, lifespan, freq, offsetFreq, period, periodSD, pulseLen, param1, param2, param3, param4, param5, param6, param7, param8, extra, tsSG, approved, repeatInterval, deployID, status, tsStart, tsEnd, deferSec, speciesID, markerNumber, markerType, latitude, longitude, elevation   {"data":\[{"tagID":6659, "projectID":26, "mfgID":"312", "type":"ID", "codeSet":"Lotek-4", "manufacturer":"Lotek", "freq":166.38, "offsetFreq":-0.43039024, "period":10.095333, "periodSD":7.0710937E-10, "pulseLen":2.5, "param1":85.416664, "param2":131.91667, "param3":78.0, "param4":0.14433698, "param5":0.14433728, "param6":5.7735133E-7, "tsSG":1.43231949E9, "approved":false, "status":"pending"}\]}
  ----------- -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

Errors specific to this entry point: none

User validation (status: completed)
-----------------------------------

GET
[*http://motus-wts.org/data/api/v1.0/validateuser.jsp*](http://motus-wts.org/data/api/v1.0/validateuser.jsp)

Validate a Motus login. Public requests are not available.

Additional parameters: none

Output (HTTP response codes: 200)

  ----------- ------------------------------------------------------------ --------------------
  **Name **   **Description**                                              **Example values**

  userType    Type of user (provided when authentication is successful):

              -   other

              -   collaborator

              -   principal

              -   administrator


  ----------- ------------------------------------------------------------ --------------------

Errors specific to this entry point: none

User registration (Status: completed)
-------------------------------------

POST
[*http://motus-wts.org/data/api/v1.0/registeruser.jsp*](http://motus-wts.org/data/api/v1.0/registeruser.jsp)

Creates a new Motus login profile. Attempts to register using an
existing login name, or providing incomplete parameters, will return an
error and will not create the new login. In this case, loginNew and
pwordNew parameters refer to the new credentials, and are therefore not
validated against an existing login. However, logins that already exist
or logins or passwords that fail to meet the required standards (to be
determined) will generate an error. Public calls are permitted, but an
authenticated call with admin credentials can be used in conjunction
with type=principal to immediately approve the new user PI status.

Additional request parameters:

  ----------- -------------- ------------------- -----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  **Name **   **Required**   **Default value**   **Description**
  fmt         No             json                Supported output formats: json, jsonp
  loginNew    Yes            n/a                 New login
  pwordNew    Yes            n/a                 New password
  firstName   Yes            n/a                 User first name
  lastName    Yes            n/a                 User last name
  email       Yes            n/a                 User email
  phone       No             n/a                 User phone number
  lang        No             EN                  Two-letter language code associated with the user preference
  type        No             other               Whether the user is a collaborator or requires principal investigator credentials. Possible values are: collaborator, principal, other. PI’s have additional permissions including creating new projects, but this level must be approved by a Motus administrator before becoming effective.
  comments    No             n/a                 Text field describing why the user is registering to Motus, which can be read by the administrators
  ----------- -------------- ------------------- -----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

Output (HTTP response codes: 201)

  ----------- ------------------------------------------------------- --------------------
  **Name **   **Description**                                         **Example values**
  success     Indicates that the login was successfully registered.   True
  ----------- ------------------------------------------------------- --------------------

Errors specific to this entry point:

  -------------------- ------------------- ----------------------------------------------------------------------------------------------------------
  **Value**            **Http response**   **Description**
  missing-login        400                 loginNew parameter is missing
  missing-pword        400                 pwordNew parameter is missing
  missing-email        400                 email parameter is missing
  missing-first-name   400                 firstName parameter is missing
  missing-last-name    400                 lastName parameter is missing
  invalid-login        400                 The login name must be between 4 and 128 characters and should not contain special characters or accents
  invalid-pword        400                 The password must be between 5 and 32 characters
  login-exists         400                 Motus login name already exists
  email-exists         400                 A Motus login has already been registered with this email
  -------------------- ------------------- ----------------------------------------------------------------------------------------------------------

Create new project (Status: completed)
--------------------------------------

POST
[*http://motus-wts.org/data/api/v1.0/registerproject.jsp*](http://motus-wts.org/data/api/v1.0/registerproject.jsp)

Create a new Motus project. For now, only users with Principal
investigator or Administrator level can create new projects.

Parameters:

  ------------- -------------- ------------------- ---------------------------------------
  **Name **     **Required**   **Default value**   **Description**
  fmt           No             json                Supported output formats: json, jsonp
  projectName   Yes            n/a                 Project name
  ------------- -------------- ------------------- ---------------------------------------

Output (HTTP response codes: 201)

  ----------- ------------------------------------------------- --------------------
  **Name **   **Description**                                   **Example values**
  projectID   Numeric identifier for the new project created.
  ----------- ------------------------------------------------- --------------------

Errors specific to this entry point:

  ------------------ ------------------- --------------------------------------------------------------
  **Value**          **Http response**   **Description**
  missing-name       401                 Missing project name
  login-permission   401                 Login not associated with permission to create a new project
  ------------------ ------------------- --------------------------------------------------------------

Sensor registration (status: completed)
---------------------------------------

GET
[*http://motus-wts.org/data/api/v1.0/registersensor.jsp*](http://motus-wts.org/data/api/v1.0/registersensor.jsp)

Register the secret key for a new sensor. This only needs to be done
once for each serial number. All subsequent API calls must use this
secret sensor key, specific to each serial number, to calculate a hash
key for its request. Motus authentication is not required for this call,
which should only originate from the server (not the clients). Serial
numbers are required to be unique, but may not always be in practice in
the field. MAC address can then be used to discriminate them. To
register a device with a non-unique serial number, you must add a suffix
to the serial number to differentiate it from the previous registered
device (e.g. SG-1234BBBK678.0). Note that you can use this method to add
a MAC address for a previously registered device.

Additional request parameters:

  ------------ -------------- ------------------- ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  **Name **    **Required**   **Default value**   **Description**
  fmt          No             json                Supported output formats: json, jsonp
  secretKey    Yes            n/a                 Unique encrypted representation of the serial number, used to validate all subsequent API calls originating from the serial number (e.g. E7A5DDF7ED54B61810204905018396660BFF0845)
  macAddress   No             n/a                 Unique 12 hex digit identifier representing the MAC address of the device, to allow discriminating among non-unique serial numbers (e.g. 0123456789ab)
  ------------ -------------- ------------------- ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

Output (HTTP response codes: 201)

  ----------- ---------------------------------------------------------------------------------------------------------------- ----------------------------------------------
  **Name **   **Description**                                                                                                  **Example values**
  result      Describes whether the serial number was successfully registered, or whether it already existed in the database   registered, serial-exists, mac-address-added
  ----------- ---------------------------------------------------------------------------------------------------------------- ----------------------------------------------

Errors specific to this entry point:

  -------------------- ------------------- ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  **Value**            **Http response**   **Description**
  missing-key          400                 The secret key is missing.
  invalid-key          400                 The serial number has already been registered, but using a different secret key, or the key doesn’t conform to an hexadecimal value in uppercase.
  permission-denied    400                 This project does not accept unauthenticated registrations. Only authorized users (administrators, PI’s or users with appropriate permissions) can submit new registrations.
  mac-address-exists   400                 The MAC address has already been registered with a different serial number
  -------------------- ------------------- ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

Sensor deployment (status: completed?)
--------------------------------------

POST
[*http://motus-wts.org/data/api/v1.0/deploysensor.jsp*](http://motus-wts.org/data/api/v1.0/deploysensor.jsp)

This entry point uploads information pertaining to new sensor
deployments (e.g. location, date and antenna configuration). This entry
point can be used with public calls, but only if the project allows
unauthenticated calls. Unauthenticated deployments will most likely
require manual approval by the project PI using the Motus web interface,
unless the status is set to automatically approval all calls, which is
typically not recommended.

A deployment corresponds to a particular configuration of a receiver,
including its geographic position (except for mobile deployments) and
its sensor configuration. Each change to a position or sensor
configuration should be submitted as a new deployment. There can only be
1 active deployment for a given receiver at any time, and deployments
cannot overlap in time. If a new deployment is submitted that conflicts
with an existing one (same serial number at the same time among all
projects), an error message will be returned with details and the
deployment will not be created, unless the forceTermination field is set
to true. A recommended approach is to issue a first call without that
parameter and offer users whether they actually wish to terminate the
previous deployment, before issuing a second call using the
forceTermination parameter. Note that conflicts may occur in a few
situations. For instance, if you are trying to upload a new deployment
that is earlier than one already registered, you *must* set the status
terminate and provided an tsEnd value that does not overlap with the
tsStart of the existing one. forceTermination will only help resolve
cases where the new deployment is *after* the last existing one, and
only when its status is set to deploy (thus, with no end date,
suggesting that it is still active).

You can also use this method simply to “attach” a sensor to a project
and thus generate a sensorID (which is project specific). You only need
to register each physical unit only once using registersensor.jsp, but
you also need to link the sensor to a project. You can do so by issuing
a deployment using the current method, but you can also use the method
uniquely for the purpose of generating a sensorID, by using the option
status = attach, and providing a projectID (and preferably also the
receiverType).

Additional request parameters:

  -------------------------------------------------------------------------------------------------------------------------- -------------- ------------------- -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  **Name **                                                                                                                  **Required**   **Default value**   **Description**

  fmt                                                                                                                        No             json                Supported output formats: json, jsonp

  projectID                                                                                                                  Yes            0                   Motus numeric project ID. 0 means not yet assigned to a project.

  status                                                                                                                     No             n/a                 Status from one of the possible following values: **attach, pending, deploy, terminate**. “attach” is only used for the purpose of attaching the sensor to a specific project and obtain a sensorID, without creating any new deployment. All other status values generate deployments: “pending” indicates that the deployment record isn’t ready to be finalized for deployment yet, “deploy” indicates that the deployment can be activated, and “terminate” will attempt to finalize a deployment by providing an end date. Note that deployments can only be activated or terminated if all required information has been provided. Attempts to deploy or terminate an incomplete deployment will return an error.

  receiverType                                                                                                               No             SENSORGNOME         Type of receiver. One of SENSORGNOME, LOTEKSRX600, LOTEKSRXDL for now. Refer to the listreceivertype lookup API entry point.

  The following fields are only used for status = pending, deploy and terminate, and will be ignored when status = attach.

  siteCode                                                                                                                   No             n/a                 Short code assigned to the deployment site (e.g. CapeMay1).

  siteDesc                                                                                                                   No             n/a                 Human readable deployment site description

  siteNotes                                                                                                                  No             n/a                 Any additional information about this deployment site location (e.g. up on the roof, use ladder at the back).

  landOwnerID                                                                                                                No             n/a                 Numeric ID of the landowner. Refer to the listlandowner lookup API entry point.

  fixtureType                                                                                                                No             n/a                 Type of fixture (e.g. tower, existing, etc.)

  lat                                                                                                                        Yes\*          n/a                 Latitude (decimal degrees). E.g. 45.123. Not required for mobile or pending deployments.

  lon                                                                                                                        Yes \*         n/a                 Longitude (decimal degrees). E.g. -60.325. Not required for mobile or pending deployments.

  elev                                                                                                                       No             n/a                 Elevation above sea level (meters). E.g. 23. Not required.

  isMobile                                                                                                                   No             False               True or False. Whether the sensor is deployed on a mobile platform (e.g. a ship).

  ts                                                                                                                         Yes            n/a                 Timestamp (seconds since '1970-01-01T00:00:00'). Time at which registration information was generated.

  tsStart                                                                                                                    No\*           n/a                 Timestamp for start of deployment (seconds since '1970-01-01T00:00:00'). Required for status=deploy and status=terminate.

  tsEnd                                                                                                                      No\*           n/a                 Timestamp for end of deployment (seconds since '1970-01-01T00:00:00'). Required for status=terminate.

  forceTermination                                                                                                           No             false               Option to attempt overriding a conflict in deployment of the sensor serial number (including those in other projects). This may occur when the previously sensor deployment was not ended by mistake. When a conflict occurs and this option is set to true, the API will attempt to close the currently opened deployment by using the tsStart of the new deployment as the tsEnd of the existing one. A conflict error may still occur if the deployment in conflict already has a tsEnd, or if the conflict occurs with the tsStart of the existing deployment (i.e. if you are trying to upload a new deployment with no end date that occurs before an existing deployment.

  sensors                                                                                                                    No             n/a                 Data vector containing details of the antenna configuration each node containing the following elements:

  -   port                                                                                                                   Yes            n/a                 USB Hub port the sensor is attached to; 0 means directly attached; -1 means not USB



  -   type                                                                                                                   Yes            n/a                 Type of sensor. One of: "radio", "microphone", "GPS" so far.



  -   params                                                                                                                 No             n/a                 Data object, which content will depend on "type", and containing the following parameters (none of which are required):

                                                                                                                                                                Type=radio

                                                                                                                                                                -   **radio**: one of funcubePro, funcubeProPlus, RTLSDR \[so far\]

                                                                                                                                                                -   **antenna**: one of: "omni", "whip", "none", "yagi-5", "yagi-9", "jpole"

                                                                                                                                                                -   **bearing**: real; magnetic compass bearing of antenna axis, in degrees clockwise from North; not corrected for magnetic declination

                                                                                                                                                                -   **tilt**: real; tilt of antenna axis, in degrees above (positive) or below (negative) horizontal

                                                                                                                                                                -   **polBearing**: real; magnetic compass bearing of antenna polarization, in degrees clockwise from North; not corrected for magnetic declination

                                                                                                                                                                -   **polTilt**: real; tilt of antenna polarization, in degrees above (positive) or below (negative) horizontal

                                                                                                                                                                -   **height**: height of antenna above ground (metres)

                                                                                                                                                                -   **filter**: string; type of filter attached. "VHF-BPF" or "other" for now. A filter is not required.

                                                                                                                                                                -   **cableLength**: real; length of analog cable connecting antenna to radio receiver, in metres

                                                                                                                                                                -   **cableType** (cable\_type): arbitrary string

                                                                                                                                                                -   **mountDistance**: real; distance between antenna and receiver

                                                                                                                                                                -   **mountBearing**: real; magnetic compass bearing from receiver to antenna, not corrected for declination.

                                                                                                                                                                -   **details**: string; user notes about qualitative information on antenna set-up (e.g. "large tree obscuring view to NW")

                                                                                                                                                                Type=gps

                                                                                                                                                                -   **model**: one of USB, cape

                                                                                                                                                                -   **pps**: high precision pulse per second signal (True or False)

                                                                                                                                                                Type=microphone

                                                                                                                                                                -   **model**: too many to categorize? Need a free-format string?

                                                                                                                                                                -   ???


  -------------------------------------------------------------------------------------------------------------------------- -------------- ------------------- -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

Output (HTTP response codes: 201)

  -------------- ------------------------------------------------------------------------------------------------------------------------------------------------------------- --------------------
  **Name **      **Description**                                                                                                                                               **Example values**
  importID       Numeric ID assigned to the imported data record. Not currently used for anything, but may eventually be used to allow updating previously sent deployments.   1234
  sensorID       Numeric ID assigned to the sensor within the project specified. This is the value that must be included in the tag detection database.                        1015
  deploymentID   Numeric ID assigned to the new deployment.                                                                                                                    2605
  -------------- ------------------------------------------------------------------------------------------------------------------------------------------------------------- --------------------

Output (HTTP response codes: 400)

  ----------- -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- -----------------------------------------------------------------------------------------------------------------------
  **Name **   **Description**                                                                                                                                                                                                                                                                                                                                                                                                                                                                  **Example values**
  conflicts   Object containing details (importID, deploymentID, status, tsStart, tsEnd) pertaining to other existing deployments (status = deploy or terminate) that conflict with the submitted data. This is only provided when errorCode = deployment-conflict, or when a previous deployment was forcibly terminated (errorCode = success-deploy-terminated). A complete list of deployments, including pending ones, can be obtained with the *listsensordeployments* API entry point.   {“sensorID”:1234, “deploymentID”: 1019, ”status”:”deploy”, “name”: “Plover Bay”, ”tsStart”: 1427846400, ”tsEnd”:null}
  ----------- -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- -----------------------------------------------------------------------------------------------------------------------

Errors specific to this entry point:

  ---------------------------- ------------------- --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  **Value **                   **Http response**   **Description**
  import-id-not-found          400                 The import ID provided does not exist.
  invalid-coordinates          400                 Invalid coordinates or elevation parameters.
  invalid-coordinates-format   400                 One or more invalid location or elevation parameters
  invalid-coordinates-value    400                 coordinates or elevation values out of bounds
  invalid-import-id            400                 The import ID provided was not found or was not associated with the same serial number (serno).
  invalid-receiver-type        400                 Invalid receiver type
  invalid-sensor-parameters    400                 Invalid sensor params
  invalid-status               400                 Invalid deployment status (possible values: pending, deploy, terminate)
  invalid-ts-format            400                 One or more invalid timestamp parameters
  invalid-ts-value             400                 ts must be between Jan 1^st^, 2000 and cannot be in the future.
  missing-coordinates          400                 Coordinates are required for status=deploy or status=terminate (except for mobile deployments: isMobile=true).
  missing-sensor-port          400                 At least one of the sensor is missing a valid port number
  missing-sensor-type          400                 At least one of the sensor is missing a valid sensor type
  missing-ts                   400                 ts is a required field
  missing-ts-end               400                 tsStart and tsEnd are both required to terminate a deployment
  missing-ts-start             400                 tsStart is required to activate a deployment
  unsupported-type             400                 The provided sensor type is not yet supported
  invalid-ts-start-end         400                 The tsStart cannot be larger than the tsEnd.
  deployment-conflict          400                 The deployment conflicts with an existing active or terminated deployment for the same receiver serial number. This error code will be accompanied by another parameter called “conflict”.
  permission-denied            400                 This project does not accept unauthenticated deployments. Only authorized users (administrators, PI’s or users with appropriate permissions) can submit deployments.


  ---------------------------- ------------------- --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

Delete sensor deployment (status: completed)
--------------------------------------------

POST
[*http://motus-wts.org/data/api/v1.0/deletesensor.jsp*](http://motus-wts.org/data/api/v1.0/deletesensor.jsp)

Delete information pertaining to previously uploaded sensor deployments.
This entry point does not require Motus authentication. Only pending
deployments can be deleted. Once activated, it is currently not possible
to delete existing deployments. You can only delete deployments that
have been originally submitted using the API.

Additional request parameters:

  ----------- -------------- ------------------- ------------------------------------------------------
  **Name **   **Required**   **Default value**   **Description**
  fmt         No             json                Supported output formats: json, jsonp
  deployID    Yes            n/a                 Deployment ID provided after a previous data import.
  ----------- -------------- ------------------- ------------------------------------------------------

Output (HTTP response codes: 200)

  ----------- ---------------------------------------------------------------------------------------------------------- --------------------
  **Name **   **Description**                                                                                            **Example values**
  deployID    Numeric deployID assigned to the deleted data record. Should match the deployed provided in the request.   1234
  ----------- ---------------------------------------------------------------------------------------------------------- --------------------

Errors specific to this entry point:

  --------------------- ------------------- -------------------------------------------------------------------------------------------------
  **Value **            **Http response**   **Description**
  missing-deploy-id     400                 The deployID was not provided.
  deploy-id-not-found   400                 The deployID provided does not exist.
  invalid-deploy-id     400                 The deploy ID provided was not found or was not associated with the same serial number (serno).
  already-deployed      400                 The sensor has already been deployed and cannot be deleted.
  --------------------- ------------------- -------------------------------------------------------------------------------------------------

1.

2.  Tag registration (status: in progress)
    --------------------------------------

POST
[*http://motus-wts.org/data/api/v1.0/registertag.jsp*](http://motus-wts.org/data/api/v1.0/registertag.jsp)

Upload information pertaining to new tags. Register generates the tag
unique database ID and registers the tag signal properties, but presumes
that the deployment details will be provided later on, either through
the Sensor Gnome interface, or manually from within the Motus web site.
Each tag must be registered only once, except in rare cases where you
may want to re-deploy the same physical tag in more than 1 project in
various deployments. In those cases, the tag must be registered
individually to each project where it needs to be deployed. Note however
that you can change the projectID of a physical tag (and all associated
deployments) after registration through the Motus web site, without
having to register it again.

Note that the combination of projectID, mfgID and dateBin must be unique
for all tags across the entire Motus database, and that this is strictly
enforced.

Parameters:

  -------------- -------------- ------------------- -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  **Name**       **Required**   **Default value**   **Description**

  fmt            No             false               Supported output formats: json, jsonp

  projectID      Yes            n/a                 Motus numeric project ID.

  mfgID          Yes            n/a                 ID printed on tag by manufacturer. If a single project is registering more than one tag with the same mfgID, (e.g. three tags with mfgID=123), the 2nd tag is called 123.1, the third tag is called 123.2, and so on. (i.e. the first tag is 123.0 = 123). For most tags, the only additional labeling we can reasonably expect users to attach is one or more indelible dots. So we tell people to add one dot to the first tag duplicating a given ID, two dots to the second tag duplicating a given ID, and so on.\
                                                    This should be a character string, as some later tags will have manufacturer IDs consisting of 8 hexadecimal digits.

  dateBin        Yes            n/a                 Field representing a definite time period (e.g. a quarter) of when the tag registration occurred, and that is mean to discriminate tags with the same mfgID across years or seasons within the same project. For now, this is done by combining the year (YYYY) and quarter (1-4) of the time of registration (e.g. 2015-3), but this may change to some other rule later (e.g. YYYY-MM). This field should be generated by a script based on registration time, and not be user-provided.

  type           No             n/a                 Type of tag (BEEPER or ID)

  codeSet        No             n/a                 If a manufacturer has more than one code set from which a code can come, this must uniquely identify the one for this tag. (e.g. 'Lotek-3')

  manufacturer   No             n/a                 Name of manufacturer (e.g. Lotek)

  model          No             n/a                 Tag model name

  lifeSpan       No             n/a                 Expected lifespan in days (integer)

  nomFreq        No             166.380             Nominal transmitter frequency (MHz). This is typically printed on the tag by the manufacturer, as the kilohertz portion of the frequency (e.g. 380 in the case of a tag @ 166.380).

  offsetFreq     No             n/a                 Offset frequency (kHz)

  period         No             n/a                 Repeat period (seconds)

  periodSD       No             n/a                 standard deviation of repeat period (seconds)

  pulseLen       No             n/a                 Pulse length (seconds)

  param1-8       No             n/a                 Up to 8 additional parameters (float numbers). Their definitions depend on the paramType field.

  paramType      No             n/a                 Numeric value referring to the metadata description of the (up to) 8 additional parameters param1-8. This field is not strictly required, but is expected to be provided whenever values are entered in param1-8. There is however no validation to ensure that the values in param1-8 make sense in relation to the paramType value. If you are unsure what to use here, and would like to make use of the param1-8 fields, please contact the database or API administrator.

  ts             No             n/a                 Timestamp at which registration occurred. This will only be useful if the registration was routed via the sensorgnome.org server because the user provided a .WAV file via email or upload, rather than using the upcoming self-registration web page.


  -------------- -------------- ------------------- -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

Output (HTTP response codes: 201)

  ----------- ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- --------------------
  **Name **   **Description**                                                                                                                                                                             **Example values**
  tagID       Unique tag ID generated by the Motus Database, which is globally unique and must be used in other tables to refer to this physical tag, or other subsequent API calls (e.g. deployments).
  ----------- ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- --------------------

Errors specific to this entry point:

  -------------------- ------------------- --------------------------------------------------------------------------------------------------------
  **Value**            **Http response**   **Description**
  tag-exists           400                 A tag with the same mfgID has already been registered under this project and the same period (dateBin)
  missing-project-id   400                 projectID is a required field
  missing-mfg-id       400                 mfgID is a required field
  missing-date-bin     400                 dateBin is a required field



  -------------------- ------------------- --------------------------------------------------------------------------------------------------------

Tag deployment (status: in progress)
------------------------------------

POST
[*http://motus-wts.org/data/api/v1.0/deploytag.jsp*](http://motus-wts.org/data/api/v1.0/deploytag.jsp)

Uploads or updates information pertaining to tag deployments (e.g.
location, date, species, etc.).

Parameters:

  -------------- -------------- ------------------- ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  **Name **      **Required**   **Default value**   **Description**
  fmt            No             false               Supported output formats: json, jsonp
  tagID          Yes            n/a                 Unique numeric ID assigned to the tag, provided at the time of tag registration.
  status         Yes            pending             Status from one of the possible following values: **pending, deploy, terminate**. “pending” indicates that the deployment record isn’t ready to be finalized for deployment yet. “deploy” indicates that the deployment can be activated. “terminate” will attempt to finalize a deployment by providing an end date. Note that deployments can only be activated or terminated if all required information has been provided. Attempts to deploy or terminate an incomplete deployment will return an error. At the moment, several of the parameters can no longer be modified once a deployment is activated, and the status cannot be changed back to an earlier status (permitted sequence is pending -&gt; deploy -&gt; terminate, but levels can be omitted).
  tsStart        Yes            n/a                 Timestamp for start of deployment (seconds since '1970-01-01T00:00:00'). If the tag has a deferred time lag (deferTime), this is the time at which the bird is released, in such a way that the time when the tag is expected to be active is given by tsStart + deferTime.
  tsEnd          No\*           n/a                 Timestamp for end of deployment (seconds since '1970-01-01T00:00:00'). Required for status=terminate.
  deferTime      No             0                   Defer time (in seconds from tsStart). Some tags are capable of deferred activation - they don't start transmitting until some number of seconds (can be very large) after activation.
  speciesID      No             n/a                 Numeric ID (integer) of the species on which the tag is being deployed. You should refer to the listspecies API entry point to query species ID based on codes or names.
  speciesName    No             n/a                 Name (String, preferably scientific name, but can also be common name or species code) of the species on which the tag is being deployed. There is no validation on this field, including whether it matches the speciesID value. This is strictly used as a guide for users looking at data records.
  markerType     No             n/a                 Type of marker (e.g. “metal band”, “color band”)
  markerNumber   No             n/a                 Marker number or descriptor (e.g. “1234-56789” or “L:Red/Blue,R:Metal 1234-56789”)
  lat            No             n/a                 Latitude (decimal degrees) of the deployment site. E.g. 45.123.
  lon            No             n/a                 Longitude (decimal degrees) of the deployment site. E.g. -60.325.
  elev           No             n/a                 Elevation above sea level (meters) of the deployment site. E.g. 23.
  ts             Yes            n/a                 Timestamp (seconds since '1970-01-01T00:00:00'). Time at which deployment information was generated.
  comments       No             n/a                 User comments related to the deployment, for their own use.
  properties     No             n/a                 JSON object containing pairs of name/value pairs for any custom properties for the tag. E.g. \[{"AgeID":"HY", "blood":"N", "Country":"Canada", "Include\_in\_analysis":"Y", "LocationID":"Niapiskau", "Province":"Quebec","SexID":"U"}\]. Values are currently limited to 128 chars for names and unlimited for values. Each property name must be unique.
  -------------- -------------- ------------------- ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

Output (HTTP response codes: 201)

  -------------- ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ --------------------
  **Name **      **Description**                                                                                                                                                                                                                                                          **Example values**
  importID       Numeric import ID assigned to the imported data record. Only deployments submitted with valid credentials immediately receive a deployment ID. Others will have a value of “pending” until they are approved within the submitted project by a principal investigator.   1234
  deploymentID   Deployment ID assigned to the new deployment. Only deployments submitted with valid credentials immediately receive a deployment ID. Others will have a value of “pending” until they are approved within the submitted project by a principal investigator.             2605, pending
  -------------- ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ --------------------

Output (HTTP response codes: 400)

  ----------- ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ -------------------------------------------------------------------------------------------------
  **Name **   **Description**                                                                                                                                                                                                                                                                                                                                                                **Example values**
  conflict    Object containing details (importID, deploymentID, status, tsStart, tsEnd) pertaining to other existing deployments (status = deploy or terminate) that conflict with the submitted data. This is only provided when errorCode = deployment-conflict. A complete list of deployments, including pending ones, can be obtained with the *listtagdeployments* API entry point.   {“importID”:1234, “deploymentID”: 1019, ”status”:”deploy”, ”tsStart”: 1427846400, ”tsEnd”:null}
  ----------- ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ -------------------------------------------------------------------------------------------------

Errors specific to this entry point:

  ---------------------------- ------------------- --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  **Value **                   **Http response**   **Description**
  invalid-coordinates          400                 Invalid coordinates or elevation parameters.
  invalid-coordinates-format   400                 One or more invalid location or elevation parameters
  invalid-coordinates-value    400                 coordinates or elevation values out of bounds
  invalid-tag-id               400                 Invalid tagID. The tagID must have been registered through the API previously using the *registertag* entry point.
  invalid-status               400                 Invalid deployment status (possible values: pending, deploy, terminate)
  invalid-ts-format            400                 One or more invalid timestamp parameters
  invalid-ts-value             400                 ts must be between Jan 1^st^, 2000 and cannot be in the future.
  missing-tag-id               400                 tagID is a required field
  missing-ts                   400                 ts is a required field
  missing-ts-end               400                 tsStart and tsEnd are both required to terminate a deployment
  missing-ts-start             400                 tsStart is required to activate a deployment
  invalid-ts-start-end         400                 The tsStart cannot be larger than the tsEnd.
  deployment-conflict          400                 The deployment conflicts with an existing active or terminated deployment for the same receiver serial number. This error code will be accompanied by another parameter called “conflict”.
  permission-denied            400                 This project does not accept unauthenticated deployments. Only authorized users (administrators, PI’s or users with appropriate permissions) can submit deployments.


  ---------------------------- ------------------- --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

Output (HTTP response codes: 401)

  ----------- ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ -------------------------------------------------------------------------------------------------
  **Name **   **Description**                                                                                                                                                                                                                                                                                                                                                                **Example values**
  conflict    Object containing details (importID, deploymentID, status, startTs, endTs) pertaining to other existing deployments (status = deploy or terminate) that conflict with the submitted data. This is only provided when errorCode = deployment-conflict. A complete list of deployments, including pending ones, can be obtained with the *listtagdeployments* API entry point.   {“importID”:1234, “deploymentID”: 1019, ”status”:”deploy”, ”startTs”: 1427846400, ”endTs”:null}
  ----------- ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ -------------------------------------------------------------------------------------------------

Errors specific to this entry point:

  ---------------------------- ------------------- -----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  **Value **                   **Http response**   **Description**
  import-id-not-found          401                 The import ID provided does not exist.
  invalid-coordinates          401                 Invalid coordinates or elevation parameters.
  invalid-coordinates-format   401                 One or more invalid location or elevation parameters
  invalid-coordinates-value    401                 coordinates or elevation values out of bounds
  invalid-import-id            401                 The import ID provided was not found or was not associated with the same tagID.
  invalid-status               401                 Invalid deployment status (possible values: pending, deploy, terminate)
  invalid-ts-format            401                 One or more invalid timestamp parameters
  invalid-ts-value             401                 ts must be between Jan 1^st^, 2000 and cannot be in the future.
  missing-coordinates          401                 Coordinates are required for status=deploy or status=terminate (except for mobile deployments: isMobile=true).
  missing-ts                   401                 ts is a required field
  missing-ts-end               401                 tsStart and tsEnd are both required to terminate a deployment
  missing-ts-start             401                 tsStart is required to activate a deployment
  invalid-ts-start-end         401                 The tsStart cannot be larger than the tsEnd.
  deployment-conflict          401                 The deployment conflicts with an existing active or terminated deployment for the same mfgID and period, within geographical proximity and time This error code will be accompanied by another parameter called “conflict”.



  ---------------------------- ------------------- -----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

1.

2.  Delete tag deployment (status: completed)
    -----------------------------------------

POST <http://motus-wts.org/data/api/v1.0/deletetagdeployment.jsp>

Delete information pertaining to previously uploaded tag deployments.
This entry point does not require Motus authentication. Only pending
deployments can be deleted. Once activated, it is currently not possible
to delete existing deployments. You can only delete deployments that
have been originally submitted using the API. \[TODO: \*Currently
requests can originate from any sensor, including those not used for
registration\]

Additional request parameters:

  ----------- -------------- ------------------- ------------------------------------------------------
  **Name **   **Required**   **Default value**   **Description**
  fmt         No             false               Supported output formats: json, jsonp
  deployID    Yes            n/a                 Deployment ID provided after a previous data import.
  tagID       Yes            n/a                 tagID of the original record to be deleted.
  ----------- -------------- ------------------- ------------------------------------------------------

Output (HTTP response codes: 200)

  ----------- ---------------------------------------------------------------------------------------------------------- --------------------
  **Name **   **Description**                                                                                            **Example values**
  deployID    Numeric deployID assigned to the deleted data record. Should match the deployID provided in the request.   1234
  ----------- ---------------------------------------------------------------------------------------------------------- --------------------

Errors specific to this entry point:

  --------------------- ------------------- --------------------------------------------------------------------------------
  **Value **            **Http response**   **Description**
  missing-deploy-id     400                 The deployID was not provided.
  missing-tag-id        400                 The tagID was not provided.
  deploy-id-not-found   400                 The deployID provided does not exist.
  invalid- deploy-id    400                 The deployID provided was not found or was not associated with the same tagID.
  already-deployed      400                 The tag has already been deployed and cannot be deleted.

  --------------------- ------------------- --------------------------------------------------------------------------------

1.

2.  Lookup values
    -------------

    1.  ### Lookup values: speciesID (status: completed)

GET
[*http://motus-wts.org/data/api/v1.0/listspecies.jsp*](http://motus-wts.org/data/api/v1.0/listspecies.jsp)

Provides an array of JSON objects containing species ID’s and names.
“id” is the value is to be provided when sending deployment data to the
deploytag entry point. For birds, species names generally follow eBird
taxonomy. Other taxonomic groups are mostly available for Ontario and/or
Canada and are by no mean exhaustive. Please contact Motus staff if you
cannot find your species. A reasonable implementation would consist in
letting users type in a species name or code and return a limited number
of matches (e.g. 10) from which they can pick.

Additional request parameters:

  ----------- -------------- ------------------- --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  **Name **   **Required**   **Default value**   **Description**
  fmt         No             false               Supported output formats: json, jsonp, csv
  qstr        No             n/a                 Query string to filter the list of species (e.g. Warbler or YRWA). Unless the language parameter is specified, this will search within the following fields and return all possible hits: English name, French name, Scientific name and Species code.
  nrec        No             20000               The maximum number of records returned by the request (max: 20,000).
  group       No             n/a                 Taxonomic group used to search for species names( e.g. BIRDS). Refer to the listspeciesgroups entry points for values. When not provided, search among entire species taxonomy.
  qlang       No             n/a                 The language used to search for species. One of the following possible values: EN (English), FR (French), SC (Scientific), CD (Species code). Leaving blank searches among all languages.
  ----------- -------------- ------------------- --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

Output (HTTP response codes: 200)

  ----------- ------------------------------------------------------------------------------------------------------------------------------- -----------------------------------------------------------------------------------------------------------------------------------------------------------------
  **Name **   **Description**                                                                                                                 **Example values**
  data        Data vector containing the species, including for each the following fields: id, english, french, scientific, group and sort.   {"data":\[{"id":16610,"english":"Yellow-rumped Warbler","french":"Paruline à croupion jaune","scientific":"Setophaga coronata","group":"BIRDS","sort":16610}\]}
  ----------- ------------------------------------------------------------------------------------------------------------------------------- -----------------------------------------------------------------------------------------------------------------------------------------------------------------

Errors specific to this entry point:

  --------------- ------------------- ----------------------------------------------------------------------------------------------------------
  **Value **      **Http response**   **Description**
  invalid-group   400                 Invalid group name (e.g. BIRDS). Refer to the listspeciesgroups entry point for complete list of values.
  invalid-lang    400                 lang must be one of the following: EN, FR, SC or CD.
  --------------- ------------------- ----------------------------------------------------------------------------------------------------------

### Lookup values: Receiver Types (status: completed)

GET
[*http://motus-wts.org/data/api/v1.0/listreceivertypes.jsp*](http://motus-wts.org/data/api/v1.0/listreceivertypes.jsp)

Provides an array of receiver types (e.g. SENSORGNOME). “id” is the
value is to be provided when sending deployment data to the deploysensor
entry point. “name “ is the value to be displayed for the end users
(e.g. drop down list).

Additional request parameters:

  ----------- -------------- ------------------- --------------------------------------------
  **Name **   **Required**   **Default value**   **Description**
  fmt         No             false               Supported output formats: json, jsonp, csv
  ----------- -------------- ------------------- --------------------------------------------

Output (HTTP response codes: 200)

  ----------- ---------------------------------------------------- ----------------------------------------------------------------------------------------------------------------------------------------------
  **Name **   **Description**                                      **Example values**
  data        Data vector containing the id and name value pairs   {"data":\[{"id":"LOTEKSRX600","name":"Lotek SRX 600"},{"id":"LOTEKSRXDL","name":"Lotek SRX DL"},{"id":"SENSORGNOME","name":"SensorGnome"}\]}
  ----------- ---------------------------------------------------- ----------------------------------------------------------------------------------------------------------------------------------------------

Errors specific to this entry point:None

### Lookup values: Receiver Status (status: completed)

GET
[*http://motus-wts.org/data/api/v1.0/listreceiverstatus.jsp*](http://motus-wts.org/data/api/v1.0/listreceiverstatus.jsp)

  ----------- -------------- ------------------- --------------------------------------------
  **Name **   **Required**   **Default value**   **Description**
  fmt         No             false               Supported output formats: json, jsonp, csv
  ----------- -------------- ------------------- --------------------------------------------

Debug (status: in progress)
---------------------------

GET <http://motus-wts.org/data/api/v1.0/debug.jsp>
