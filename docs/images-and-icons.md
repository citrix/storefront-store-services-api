#Images and Icons
These services return the appropriate image/icon for a specified thumbprint. The two services are related, in that each is used to retrieve image data relating to one or more resource using the concept of a 'thumbprint' to allow sharing of image data for resources with equivalent images. The 'thumbprint' is designed to be the same for images with the same content. The services differ in that the image service will allow different sizes and format of image to be returned as desired by the caller, possibly resizing the raw images, while the icon service will always return all of the raw images combined into a single file in the Microsoft icon (.ico) format.

##RequestURL (indicative only)|Method|Description|
---|---|---|/resources/v2/{thumbprint}/image[/{size}]|GET|Returns the specified image. See below for details on the format and other details of the response.|/resources/v2/{thumbprint}/icon|GET|Returns the specified icon data in the image/vnd.microsoft.icon format.|
Parameter|Description|---|---|{thumbprint}|The thumbprint for the image/icon. This value is typically generated in the data returned to a GET request to /resources[/{id}] and is generated for the image data at that moment and so may be different in a subsequent call if the image data has changed. This value is also unique to the instance of the service.|
{size}|The size of the image to return (defaults to 32 if no value is supplied). Only the values 16, 24, 32, 48, 64, 96, 128, 256 and 512 are supported.|clientName, clientAddress|See Client Name, Client Address and Device Id section above.|
**Notes**:

* These requests require an Authorisation token
* The URLs for requests for image/icon data are illustrative only. The actual URLs should be obtained from the Response to resource enumeration (i.e. /resources/v2/ or /resources/v2/{id}) which will return the URL.
* However, the optional suffix of the image request (i.e. [/{size}] ) must be constructed by the client.
* The response image format is determined by the requested content type (i.e. the value(s) for the Accept header). The recognised content types are: image/png, and image/gif. Additionally the wildcard values image/* and */* are also recognised as referring to all supported content types in no defined order. If multiple content types are specified, then specific contents types (e.g. image/gif) are taken as higher precedence than wildcard types (e.g. image/*) and the order is taken to indicate preference amongst specific content types. Accept headers with ‘q’ values are ignored. See http://www.ietf.org/rfc/rfc2616.txt section 14.1 for details on Accept headers.
* The images service will always return an image of the requested format and size (or its best choice if not specified) even if this requires resizing the raw image data.

##Response
Response Code|Description|
---|---|200 (OK)|Success|401 (Unauthorized)|Bad/Missing security token (see CitrixAuth Authentication Scheme document [3])|400 (Bad Request)|Invalid size parameter.|404|Unknown/missing thumbprint.|
Response Format|Request Accept /Response Content-Type Header|
---|---
GIF|image/gif|
PNG|image/png|

###Example: Successful request for a 32x32 png image

####Request
```
GET http://www.example.com/Citrix/Store/resources/v2/5C6D3CD...7A/image/32 HTTP/1.1 Accept: image/png
Authorization: CitrixAuth H4sIA....
Host: www.example.com
```
####Response
```
HTTP/1.1 200 OK
Cache-Control: public, no-store, max-age=0
Content-Type: image/png
Content-Length: 2627

... 32 x 32 png image data ...
```

###Example: Successful request for a default-sized gif image
The request uses Accept headers to give image/gif as its required format, it also specifies no size, so the server returns the default size, i.e. 32.

####Request
```
GET http://www.example.com/Citrix/Store/resources/v2/5C6D3CD...7A/image HTTP/1.1 Accept: image/gif
Authorization: CitrixAuth H4sIA....
Host: www.example.com
Connection: Keep-Alive
```
####Response
```
HTTP/1.1 200 OK
Cache-Control: public, no-store, max-age=0
Content-Type: image/gif
Content-Length: xxxx

... 32 x 32 gif image data ...
```