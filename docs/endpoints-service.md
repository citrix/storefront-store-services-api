#Endpoints Service

This service returns a list of all the endpoints available with their associated metadata. An endpoint is essentially the URL of a service that the client can use to start a communication with the server. The endpoint service itself is considered an endpoint and it is included in the response.

**Note**: The endpoints service is not limited to the Store service (it is shared with, for example, the authentication service), but is defined here.

##Request
|URL|Method|Description|
|---|---|---||/endpoints/v1/|GET|Returns the list of all the endpoints available.|

**Notes**:

* The request does not require an Authorisation token.

##Response
|Response Code|Description|
|---|---||200|Success|

##Success Response Content
In the case that a successful reponse is returned, the response body contains an Xml document giving the list of endpoints available. Each endpoint has the following data:

* id : Uniquely identifies the endpoint.
* url: The absolute URL of the endpoint.
* capabilities: A list of strings identifying the capabilities supported by the endpoint.

|Response Format|Request Accept /Response Content-Type Header|
|---|---||Xml|application/vnd.citrix.endpoints+xml|

The result content is Xml described by the schema: /Schemas/Endpoints.xsd.

###Example: Successful Endpoints Enumeration
###Request
```
GET http://www.example.com/endpoints/v1/ HTTP/1.1
Host: www.example.com
Accept: application/vnd.citrix.endpoints+xml
Authorization: CitrixAuth security-token
```
####Response
```
HTTP/1.1 200 OK
Content-Type: application/vnd.citrix.endpoints+xml
Content-Length: xxx
Cache-Control: xxx
```
```
<endpoints xmlns="http://citrix.com/dazzleservices/2-0/endpoints"> <endpoint id="Endpoints">
<url>http://www.example.com/Data/endpoints/v1</url> <capabilities>
      <capability>endpoints</capability>
    </capabilities>
  </endpoint>
  <endpoint id="ListResourcesWithAutoProvision">
<url>http://www.example.com/Data/resources/v2</url> <capabilities>
<capability>ResourcesEnumerationV2</capability> <capability>AutoProvision</capability> <capability>group:core</capability>
  ...
    </capabilities>
</endpoint>
  ...
</endpoints>
``` 