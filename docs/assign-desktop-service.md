#Assign Desktop Service

This service allows a desktop to be assigned to a particular user (the user is identified by the security token in the request.

##Request
This service allows a desktop to be assigned to a particular user (the user is identified by the security token in the request.

URL (indicative only) and Content-Type|Method| Description|
---|---|---|/resources/v2/{resourceId}/assigndesktop Application/vnd.citrix.assigndesktop+xml|POST|Assigns the desktop indicated by the resourceId.

**Notes**:

* The URLs given above are for illustrative purposes only. The actual URL used in a request should be obtained from the
results of an enumeration request (eg /resources/v2/ or /resources/v2/{id}).
<br>**Request data**
* The POSTed requests are Xml, described by the schema: /Schemas/AssignDesktopParams.xsd, with top level element assigndesktopparams.

##ResponseResponse Code (and friendly name)|Description|
---|---|200 (OK)|The desktop assignment operation has been attempted and the response xml document (Content-Type: application/vnd.citrix.assigndekstop+xml) indicates whether the operation succeeded.<br>The response xml document is described by the schema /Schemas/AssignDesktop.xsd.|404 (Not Found)|The specified resource does not support the assign operation.

##Response Error Codes
The following error codes can be returned for errors in assigning a desktop:

Error Code|Description---|---|no-available-workstation|No VDAs are available to process this request.|connection-refused|The server refused a connection.resource-unavailable|The desktop is no longer available to the user.unspecified|Unspecified error.

###Example: Assign Desktop Success
####Request
```
POST http://www.example.com/Citrix/Store/resources/v2/Q2l0cm...Q-/assigndesktop HTTP/1.1 Authorization: CitrixAuth H4sIA....
Host: www.example.com
Accepts: application/vnd.citrix.assigndesktop+xml,
Content-Type: application/vnd.citrix.assigndesktopparams+xml Content-Length: xxxx
```
```
<?xml version="1.0" encoding="utf-8"?>
<assigndesktopparams xmlns="http://citrix.com/delivery-services/1-0/assigndesktopparams">
  <clientName>clientname</clientName>
<clientAddress>10.70.0.123</clientAddress> </subscriptionUpdate>
```
####Response
```
HTTP/1.1 200 OK
Cache-Control: public, no-store, max-age=0
Content-Type: application/vnd.citrix.assigndesktop+xml
Date: Wed, 16 Feb 2011 17:31:43 GMT
Content-Length: xxxx
```
```
<?xml version="1.0" encoding="utf-8"?>
<assigndesktop xmlns=http://citrix.com/delivery-services/1-0/assigndesktop status=”success”>
 <netbiosname>desktop1</netbiosname>
</assigndesktop>
```

###Example: Assign Desktop Failure
####Request
```
POST http://www.example.com/Citrix/Store/resources/v2/Q2l0cm...Q-/assigndesktop HTTP/1.1 Authorization: CitrixAuth H4sIA....
Host: www.example.com
Accepts: application/vnd.citrix.assigndesktop+xml,
Content-Type: application/vnd.citrix.assigndesktopparams+xml Content-Length: xxxx
```
```
<?xml version="1.0" encoding="utf-8"?>
<assigndesktopparams xmlns="http://citrix.com/delivery-services/1-0/assigndesktopparams">
  <clientName>clientname</clientName>
<clientAddress>10.70.0.123</clientAddress> </subscriptionUpdate>
```

####Response
```
HTTP/1.1 200 OK
Cache-Control: public, no-store, max-age=0
Content-Type: application/vnd.citrix.assigndesktop+xml
Date: Wed, 16 Feb 2011 17:31:43 GMT
Content-Length: xxxx
```
```
<?xml version="1.0" encoding="utf-8"?>
<assigndesktop xmlns=http://citrix.com/delivery-services/1-0/assigndesktop status=”failure”>
 <errorid>no-available-workstation</errorid>```