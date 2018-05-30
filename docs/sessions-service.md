#Sessions Service
There are methods available through the NFuse protocol for tracking ICA sessions not connected to by the current client. These may be disconnected or connected to by other clients. Methods are provided to allow the client to:

* Disconnect the currently connected session(s) with support for logon tickets.
* LogOff the currently connected session(s) with support for logon tickets.
* List any disconnected sessions.
* List any sessions connected to other clients (although this disconnects those sessions as an unavoidable side-effect!).
* Connect to a disconnected session.

As the current behaviour is neither safe nor idempotent, all requests require the use of the POST verb.

##RequestURL (indicative only)|Method|Request Content-Type|Response Content-Type|Description|
---|---|---|---|---|
/sessions/v1/available|POST|Session Parameters|Session State|Returns data on sessions which are available for connection from the client.<br>**Note that with the exception of XenDesktop 7 (and later) that this has side-effect of disconnecting sessions if 'active' sessions are queried.**|/sessions/v1/disconnect/|POST|Session Parameters|Session Result|Disconnect the requested sessions|/sessions/v1/logoff/|POST|Session Parameters|Session Result|Logoff the requested sessions|/sessions/v1/{session-id}/launch/ica|POST|Launch Parameters|Launch Data|Attempt to launch the specified session using the ica protocol.| Parameter|Description
---|---|{session-id}|The identifier of the session.
**Notes**:

* These requests require an Authorisation token
* Requesting ‘active’ session types will actively disconnect those sessions from other clients as a side-effect while returning the session details. This is a limitation of the underlying XenApp API.
* The Launch Parameters are described by the schema: /Schemas/LaunchParams.xsd and the response Launch Data is described by the schema: /Schemas/LaunchData.xsd
* The launch URL value is obtained by the client from the results of the available session requests.
* The other URLs are obtained via the endpoints service (see Endpoints section below).

##Request Parameters
The following information is required by the requests to the session service. The data is sent as Xml in the body of the POST described by the schema: /Schemas/SessionParameters.xsd
Parameter|Required|Description|
---|---|---|clientName|Yes|A string identifying the client. It is the client's responsibility to use a value that will behave appropriately (see notes in Launch section)|deviceId|No|Identifies the device, if specified, it overrides the clientName in identifying the end-point.includeActiveSessions|No|A boolean specifying whether to include active sessions. <pre>\[true\|false\]
appSessionsOnly|No|A Boolean specifying whether to only return only application sessions.<br>A value of true will list app sessions from any XD7 farm and pre- XD7 XenApp app hosting farm. Pre XD7 XenDesktop hosting farms will not be queried so as not to disconnect active desktops (a side effect of querying sessions).<br>Where a XenApp hosts both apps and desktops the session queries will continue to disconnect a users active desktop.<br>Farm exclusion based on version can be overridden to list all farm application sessions by setting the LegacyWorkspaceControl value to on (off is the default) in the Store <farm/> element.<br> <pre>\[ true \| false \] </pre> with the default being false.|
tickets|No|This parameter is **only used in LogOff and Disconnect session services** and represents a list of tickets that will be invalidated by the server.
**Note**: The presence of the session parameters shall be signaled by setting the Content-Type of the POST to be: 
<pre>application/vnd.citrix.sessionparams+xml</pre>

##Response

Response Code|Description
---|---|200|Success/Failure|401|Bad/Missing security token <br>(see CitrixAuth Authentication Scheme document [3])|404|Session identified by {session-id} was not found|400|POSTed data missing or invalid|406|Unrecognised Accepts Header|
Response Format|Request Accept /Response Content-Type Header
---|---|Session State|application/vnd.citrix.sessionstate+xml
**Notes**:

* The Response to the Request for session information returns Session Data, the schema for which can be found at: /Schemas/SessionState.xsd.
* The Response to the Request to disconnect or logoff returns Session Result Data, that describes the success or failure of the entire operation. This data is Xml, described by the schema: /Schemas/SessionResult.xsd.

##Endpoints
URL (indicative only)|Id|Capabilities
---|---|---|/sessions/v1/available|ListSessions|listSessions/sessions/v1/disconnect|DisconnectSessions|disconnectSessions/sessions/v1/logoff|LogOffSessions|logOffSessions

###Example: Request Session Information
In this example all sessions are requested by not qualifying the server and client types.####Request
```
POST http://www.example.com/Store/sessions/v1/available HTTP/1.1 Host: www.example.com
Accept: application/vnd.citrix.sessionstate+xml
Content-Type: application/vnd.citrix.sessionparams+xml Content-Length: xxx
Authorization: CitrixAuth ...
```
```
<?xml version="1.0"?>
<sessionparams xmlns="http://citrix.com/delivery-services/1-0/sessionparams">
<clientName>xxxxxxxxx</clientName> <deviceId>xxxxxxxxx</deviceId> <includeActiveSessions>true</includeActiveSessions>
</sessionparams>
```
####Response
```
HTTP/1.1 200 OK
Content-Type: application/vnd.citrix.sessionstate+xml Content-Length: xxx
Cache-Control: no-cache
```
```
<?xml version="1.0"?>
<sessionState xmlns="http://citrix.com/delivery-services/1-0/sessionstate"
             enumeration="full">
  <sessions>
    <session id="zyxw9876">
      <serverType>win32</serverType>
      <launchIca>
<url>http://www.example.com/Store/session/v1/zyxw9876/launch/ica</url> </launchIca>
<initialapp>MSPaint</initialapp> <initialappresourceaggregated>false</initialappresourceaggregated> <initialappresourceid>XA1.MSPaint</initialappresourceid> <publishername>Farm1</publishername>
    </session>
    <session id="3e4d5f6g">
      <serverType>win32</serverType>
      <launchIca>
<url>http://www.example.com/Store/session/v1/3e4d5f6g/launch/ica</url> </launchIca>
<initialapp>Notepad</initialapp> <initialappresourceaggregated>false</initialappresourceaggregated> <initialappresourceid>XA1.Notepad</initialappresourceid> <publishername>Farm1</publishername>
    </session>
  </sessions>
</sessiondata>
```
###Example: Disconnect

####Request
```
POST http://www.example.com/Store/sessions/v1/disconnect HTTP/1.1 Host: www.example.com
Accept: application/vnd.citrix.sessionresults+xml
Content-Type: application/vnd.citrix.sessionparams+xml Content-Length: xxx
Authorization: CitrixAuth ...
```
```
<?xml version="1.0"?>
<sessionparams xmlns="http://citrix.com/delivery-services/1-0/sessionparams">
  <clientName>xxxxxxxxx</clientName>
  <deviceId>xxxxxxxxx</deviceId>
  <tickets>
<ticket>85EB92973D2C3804F93A99AAEC232D</ticket> </tickets>
</sessionparams>
```
####Response
```
HTTP/1.1 200 OK
Content-Type: application/vnd.citrix.sessionresults+xml Content-Length: xxx
Cache-Control: no-cache
```
```
<?xml version="1.0"?>
<sessionResults xmlns="http://citrix.com/delivery-services/1-0/sessionresult" status="success" />
```

###Example: LogOff

####Request

```
POST http://www.example.com/Store/sessions/v1/logoff HTTP/1.1 Host: www.example.com
Accept: application/vnd.citrix.sessionresults+xml Content-Type: application/vnd.citrix.sessionparams+xml Content-Length: xxx
Authorization: CitrixAuth ...
```
```
<?xml version="1.0"?>
<sessionparams xmlns="http://citrix.com/delivery-services/1-0/sessionparams">
  <clientName>xxxxxxxxx</clientName>
  <deviceId>xxxxxxxxx</deviceId>
  <tickets>
    <ticket>85EB92973D2C3804F93A99AAEC232D</ticket>
    <ticket>85EB92973D2C3804F93A99AAEC232A</ticket>
  </tickets>
</sessionparams>
```

####Response
```
HTTP/1.1 200 OK
Content-Type: application/vnd.citrix.sessionresults+xml Content-Length: xxx
Cache-Control: no-cache
```
```
<?xml version="1.0"?>
<sessionResults xmlns="http://citrix.com/delivery-services/1-0/sessionresult" status="success" />
```