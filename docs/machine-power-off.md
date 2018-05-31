#Machine Power Off
This service triggers the physical machine for the specified resource to be powered down. Only specific resources support this action, and typically only for a limited set of users. Due to limitations on the XenDesktop server, this service may return with a timed-out error response, even if the power off is successful, if it does not happen within a required time-limit.

##Request
|URL (indicative only)|Method|Description|
|---|---|---||/resources/v2/{id}/poweroff|POST|Returns the result of triggering a Power Off operation on the underlying machine. See below for details on the format of the response.|

**Notes**:

* The URLs given above are for illustrative purposes only. The actual URL used in a request should be obtained from the server in the response to a resource enumeration request (e.g. /resources/v2/).
* The data POSTed is Xml described by the schema: /Schemas/ MachinePowerOffParams.xsd
* The request content-type must be: application/vnd.citrix.machinepoweroffparams+xml

##Response|Response Code|Description|
|---|---||200|PowerOff request sent to machine leading to success/failure of power off operation.||401|Bad/Missing security token <br>(see CitrixAuth Authentication Scheme document [3])||404|Resource identified by {id} was not found or the power-off action did not exist or was not enabled for that resource for the current user.<br> The reason is indicated by the X-Citrix-Error-Reason header value in the HTTP response: <br>**ResourceNotFound**: No resource was found for the requesting user for the specified id. <br> **ResourceDisabled**: The resource was found but was not enabled.<br> **UnsupportedOperation**: ICA launch was not supported for this resource.|

|Response Format| Request Accept /Response Content-Type Header|
|---|---||Xml|application/vnd.citrix.machinepoweroff+xml|
The response returns data regarding the power off request indicating whether the power off operation succeeded. If unsuccessful, the error reason is indicated with an error id. The response is Xml, the schema for which can be found at: /Schemas/MachinePowerOff.xsd.
|Error Id|Description|
|---|---||"in-maintenance-mode"|The requested operation couldn't be performed because the machine is in maintenance mode.||"no-machine"|No assigned machines were found for the user in the specified machine group.||"not-supported"|The requested operation is not supported on the requested machine group type.||"operation-in-progress"|The server is busy with a control operation on the specified machine and cannot handle another request at this time.||"timed-out"|The server timed out waiting for the request to complete.<br> **Note**: The power off operation may have succeeded, but not within the timeout period defined on the XenDesktop server.|
|"not-trusted"|The client is not trusted to perform the requested operation. This does not imply whether the credentials supplied are/are not valid.||"unspecified"|Error condition not covered by the other values.|
###Example: Successful PowerOff

####Request
```
POST http://www.example.com/resources/v2/1234...789/poweroff HTTP/1.1 Host: www.example.com
Accept: application/vnd.citrix.machinepoweroff+xml
Accept-Language: en-gb
Content-Type: application/vnd.citrix.machinepoweroffparams+xml Content-Length: 0
Authorization: CitrixAuth security-token
```
```
<?xml version="1.0"?>
<machinepoweroffparams xmlns="http://citrix.com/delivery-services/1-0/machinepoweroffparams">
  <clientName>clientname</clientName>
</machinepoweroffparams>
```
####Response
```
HTTP/1.1 200 OK
Content-Type: application/vnd.citrix.machinepoweroff+xml Content-Length: xxx
Content-Language: en-gb
Cache-Control: no-cache
```
```
<?xml version="1.0"?>
<machinepoweroff xmlns="http://citrix.com/delivery-services/1-0/machinepoweroff"
        status="success">
</machinepoweroff>
```