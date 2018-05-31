#Launch
##Request|URL (indicative only)|Method|Description||---|---|---|
|/resources/v2/{id}/launch/ica|POST|Attempt to launch the specified resource using the ICA protocol||/resources/v2/{id}/launch/rade|POST|Attempt to launch the specified resource using the Rade protocol||/resources/v2/{id}/launch/{protocol}/{retrykey}|POST|Retry the attempt to launch the specified resource using the specified protocol|
|Parameter|Description|
|---|---|
|{id}|The identifier of the associated resource. This is unique to the instance of the service.||{protocol}|The protocol to be used to launch the resource ('ica' or 'rade')||{retrykey}|The retry key used to identify a launch request that is not ready (see below)|
**Notes**:

* These requests require an Authorisation token.
* The URLs given above are for illustrative purposes, the actual URLs used by a client should be taken from the result of a previous request (e.g. /resources/v2/ or /resources/v2/{id} for all launch urls apart from the retry url which should be returned in a launch retry response).
* The launch requests require the specification of launch parameters, included as Xml in the POST body, described below.
* If subscriptions are enabled, then unsubscribed resources are prevented from launching.

##Launch Parameters
The following information is required by the launch process, which is sent as POSTed Xml, described by the schema: /Schemas/LaunchParams.xsd|Parameter|Required|Launch Type|Description|
|---|---|---|---||clientName|Optional|All protocols|A string identifying the client (any characters except null (0) or newline characters. It is the client's responsibility to use a value that will behave appropriately (see notes below)||clientAddress|Optional|All protocols|The IPv4 or IPv6 address of the client, as claimed by the client||deviceId|Optional|All protocols|A string identifying the client device||audio|Optional|ica and rade|The audio settings for the session, one of the following:<br><pre>\[high\|medium\|low\|off\]</pre>||colourDepth|Optional|ica and rade|The colour depth for the session, one of the following: <pre>\[16 \| 256 \| high \| truecolor \]</pre>||display|Optional|ica and rade|The display type for the session, one of the following: <pre>\[seamless \| percent \| absolute \| fullscreen\]</pre>|
|displayPercent|Optional|ica and rade|Only if display=percent<br>The percentage of the screen to be used for the session <br><pre>\[ 0 < percent ≤ 100\]</pre>||displayAbsolute|Optional|ica and rade|Only if display=absolute<br>The x and y extents of the window to be used for the session<br><pre>\[ 0 < x,y\]</pre>||transparentKeyPassthrough|Optional|ica and rade|Set the behavior of the windows keys etc., one of the following: <pre>\[local \| remote \| fullscreenonly \]</pre>|
|specialFolderRedirection|Optional|ica and rade|Are the special folders directed, one of the following: <pre>\[true \| false \]</pre>||clearTypeRemoting|Optional|ica and rade|Are ClearType fonts remoted, one of the following: <pre>\[true \| false \]</pre>||showDesktopViewer|Optional|ica|Should the desktop viewer be used as the ICA client. <pre>\[true \| false \]</pre>||launcher|Optional|ica|Identifier for the launcher program.<br>Any string is allowed, not including null (0) or newline characters.||virtualComPort|Optional|ica|If specified, determines the value of the VirtualCOMPortEmulation setting in the WFClient section.<pre>\[true \| false \]</pre>||comPortMapping|Optional|ica|If specified, determines the value of the COMAllowed setting in the WFClient section.<pre>\[true \| false \]</pre>||clientPrinter|Optional| ica|If specified, determines the values of the CPMAllowed and VSLAllowed settings in the WFClient section.<pre>\[true \| false \]</pre>|

**Notes**:

* The presence of the launch parameters shall be signaled by setting the Content-Type of the POST to be: application/vnd.citrix.launchparams+xml
* The use of clientAddress is required to take advantage of the Zone Preference feature of XenApp.
* For the Workspace control feature, either clientName or deviceId is required.
* The clientName either identifies the client machine or the end-user, depending on the semantics required by the client and is usually one of the following:
	* The client hostname
	* A random string
	* A digest of username+domain (assumed by older versions of the XenApp Xml Service and XenApp for Unix versions)
	
**Note**: If the resources are being accessed through Access Gateway, then the clientName is specified by the gateway.

##Response|Response Code|Description|
|---|---||200|Success/Retry required/failure||401|Bad/Missing security token <br>(see CitrixAuth Authentication Scheme document [3])||404|Resource identified by {id} was not available for launch. The reason is indicated by the X-Citrix-Error-Reason header value in the HTTP response: <br>ResourceNotFound: No resource was found for the requesting user for the specified id.<br>**ResourceDisabled**: The resource was found but was not enabled.<br>**UnsupportedOperation**: ICA launch was not supported for this resource.<br>**SubscriptionStatusInvalid**: Workflow was enabled for the resource but was not in the subscribed state.<br>**SubscriptionStatusUnknown**: Workflow was enabled for the resource, but the system was unable to determine the subscription state.|

|Response Format|Request Accept /Response Content-Type Header|
|---|---||Xml|application/vnd.citrix.launchdata+xml|

The response returns data regarding the launch request that can be categorised as: Success, Retry required, or Failure. A retry is required, for example, when a desktop has to be started before a connection can be made to it. The data returned in the response is Xml described by the schema: /Schemas/LaunchData.xsd.

##ICA Launch Result
Successful ICA launch results contain the following data: 

ica : The ICA file content.

logonTicket : Specifies any logon ticket that might have been generated for the current launch. This value could be parsed from the ICA file content, but is supplied as a convenience for clients. Clients are expected to use this as a security measure to allow any outstanding logon tckets to be cleaned up if the client is closed while launches are outstanding with logon tickets that might not have been redeemed.

##Failed Launch Error Codes
If the response represents a failed launch (launch type ‘error’), then the contained error id will be one of the following values:
|Launch error Id|Description|
|---|---||AppRemoved|The resource to launch no longer exists, is no longer enabled or is no longer visible to the current user.||NoMoreActiveSessions|The user is not allowed any more active sessions. Currently applies to XenDesktop only.||NotLicensed|The server does not have the required license to perform the requested activity.||UnavailableDesktop|No workstations are available to service this request. Currently applies to XenDesktop only.||CouldNotConnectToWorkstation|The server refused a connection. Currently applies to XenDesktop only.||WorkstationInMaintenance|The requested workstation is in maintenance mode and cannot be used. Currently applies to XenDesktop only.||ResourceError <br> GeneralAppLaunchError|General error that cannot be further specified.|
##Retry Reason Codes
If the response represents a delayed launch (launch type ‘’retry”), then the contained retryReason will be one of the following values:

|Launch retry reason|Description|
|---|---||rebooting|The requested workstation is rebooting.|
|resuming|The requested workstation is resuming.|
|unknown|Retry reason not covered by the other values.|

###Remedy Hint Codes
If the response represents a delayed or failed launch, then the response may contain a remedyHint value with one of the following values:

|remdyHint values|Description|
|---|---||restart-desktop|The problem may be resolved by restarting the desktop.|

##Launch Examples
###Example: Successful Ica launch
####Request
```
POST https://www.example.com/Citrix/Store/resources/v2/Q2l0cml4Lk1QUy5BcHAu/launch/ica HTTP/1.1 Accept: application/vnd.citrix.launchdata+xml
Authorization: CitrixAuth H4sIAAAAAAAEAO29B2AcS/T+x8z9Ajw4AAA==
Content-Type: application/vnd.citrix.launchparams+xml
Content-Length: xxx
```
```
<?xml version="1.0" encoding="utf-16"?>
<launchparams xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
              xmlns:xsd=http://www.w3.org/2001/XMLSchema
xmlns="http://citrix.com/delivery-services/1-0/launchparams"> <deviceId>clientname</deviceId>
  <clientName>clientname</clientName>
</launchparams>
```

####Response
```
HTTP/1.1 200 OK
Cache-Control: public, no-store, max-age=0 Content-Type: application/vnd.citrix.launchdata+xml Content-Length: xxxx
```
```
<?xml version="1.0" encoding="utf-8"?>
<launch status="success" xmlns="http://citrix.com/delivery-services/1-0/launch"> <result type="ica">
  <ica>...[ICA File Content]...</ica>
<logonTicket>+dPeXidrRyAgdyqJzX...</logonTicket> </result>
</launch>
```
###Example: Failed Rade launch
####Request
```
POST https://www.example.com/Citrix/Store/resources/v2/Q2l0cml4Lk1QUy5BcHAu/launch/rade HTTP/1.1 Host: www.example.com
Accept: application/vnd.citrix.launchdata+xml
Accept-Language: en-gb
Content-Type: application/vnd.citrix.launchparams+xml Content-Length: xxx
Authorization: CitrixAuth H4sIA....
```
```
<?xml version="1.0"?>
<launchparams xmlns="http://citrix.com/delivery-services/1-0/launchparams">
<deviceId>clientname</deviceId> <clientName>clientname</clientName> <clientAddress>127.0.0.1</clientAddress>
</launchparams>
```

####Response
```
HTTP/1.1 200 OK
Content-Type: application/vnd.citrix.launchdata+xml Content-Length: xxx
Cache-Control: no-cache
```
```
<?xml version="1.0"?>
<launch xmlns="http://citrix.com/delivery-services/1-0/launch"
        status="failure">
  <result type="error">
<error>
<id>ResourceUnavailable</id>
<text>The resource is unavailable</text>
    </error>
  </result>
</launch>
```

###Example: ICA launch with retry
####Request
```
POST http://www.example.com/Citrix/Store/resources/v2/Q2l0cm...Q-/launch/ica HTTP/1.1 Host: www.example.com
Accept: application/vnd.citrix.launchdata+xml
Accept-Language: en-gb
Content-Type: application/vnd.citrix.launchparams+xml
Content-Length: xxx
Authorization: CitrixAuth H4sIA....
```
```
<?xml version="1.0"?>
<launchparams xmlns="http://citrix.com/delivery-services/1-0/launchparams">
<deviceId>clientname</deviceId> <clientName>clientname</clientName> <clientAddress>127.0.0.1</clientAddress>
</launchparams>
```
####Response
```
HTTP/1.1 200 OK
Content-Type: application/vnd.citrix.launchdata+xml Content-Length: xxx
Content-Language: en-gb
Cache-Control: no-cache
```
```
<?xml version="1.0"?>
<launch status="retry" xmlns="http://citrix.com/delivery-services/1-0/launch">
  <result type="retry">
    <retry>
<url>http://www.example.com/Citrix/Store/resources/v2/Q2l0cm...Q-/launch/ica/bbb56</url> <after>30</after>
<reason>unknown</reason>
    </retry>
  </result>
</launch>
```

**Note**: The URL used in this next request is the one returned in the previous response, which includes the retry key: ‘bbb567’:

####Request
```
POST http://www.example.com/Citrix/Store/resources/v2/Q2l0cm...Q-/launch/ica/bbb567 HTTP/1.1 Host: www.example.com
Accept: application/vnd.citrix.launchdata+xml
Accept-Language: en-gb
Content-Type: application/vnd.citrix.launchparams+xml Content-Length: xxx
Authorization: CitrixAuth H4sIA....
```
```
<?xml version="1.0"?>
<launchparams xmlns="http://citrix.com/delivery-services/1-0/launchparams">
<deviceId>clientname</deviceId> <clientName>clientname</clientName> <clientAddress>127.0.0.1</clientAddress>
</launchparams>
```
####Response
```
HTTP/1.1 200 OK
Cache-Control: public, no-store, max-age=0 Content-Type: application/vnd.citrix.launchdata+xml Content-Length: xxxx
```
```
<?xml version="1.0" encoding="utf-8"?>
<launch status="success" xmlns="http://citrix.com/delivery-services/1-0/launch">
<result type="ica">
<ica>...[ICA File Content]...</ica> <logonTicket>85EB92973D2C3804F93A99AAEC232D</logonTicket>
 </result>
</launch>
```
