#Machine Power Off
This service triggers the physical machine for the specified resource to be powered down. Only specific resources support this action, and typically only for a limited set of users. Due to limitations on the XenDesktop server, this service may return with a timed-out error response, even if the power off is successful, if it does not happen within a required time-limit.

##Request
|URL (indicative only)|Method|Description|
|---|---|---|

**Notes**:

* The URLs given above are for illustrative purposes only. The actual URL used in a request should be obtained from the server in the response to a resource enumeration request (e.g. /resources/v2/).
* The data POSTed is Xml described by the schema: /Schemas/ MachinePowerOffParams.xsd
* The request content-type must be: application/vnd.citrix.machinepoweroffparams+xml

##Response
|---|---|

|Response Format| Request Accept /Response Content-Type Header|
|---|---|


|---|---|
|"not-trusted"|The client is not trusted to perform the requested operation. This does not imply whether the credentials supplied are/are not valid.|


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