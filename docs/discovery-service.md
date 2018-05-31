##Discovery Service
This service is part of a particular Store. It returns a document describing the location of the endpoint services of the Store and associated authentication service plus information on the beacons and gateways that the client can use to access these services from internal or external networks (i.e. using a gateway or not as appropriate).

##Request

|URL|Method|Description|
|---|---|---||/discovery|GET|Returns the discovery document for the store.|

**Notes**:

* This request does not require an Authorisation token.
* The URL for this service is fixed relative to the Store location (e.g. if the Store is at location https://server/Citrix/Store then the discovery document can be requested from URL https://server/Citrix/Store/discovery.

##Response
|Response Code|Description|
|---|---||200|Success|

##Success Response Content
The result content is Xml, described by the schema: /Schemas/Discovery.xsd with top-level element Discovery.

|Response Format|Request Accept /Response Content-Type Header|
|---|---|
|Xml|application/vnd.citrix.discovery+xml <br>application/vnd.citrix.receiver.configure (deprecated, see notes)|

**Note**: For backwards compatibility with earlier versions of this service, the response format will be reported as application/vnd.citrix.receiver.configure unless application/vnd.citrix.discovery+xml is explicitly requested by the client via the Accepts header (provided application/citrix.receiver.configure is not also explicitly requested). Also no 406 response will be returned even if the Accepts header is not compatible with the returned content type. **Use of the application/vnd.citrix.receiver.configure content-type is deprecated and support for it may be dropped in future releases.**

##Endpoints|URL|Id|Capabilities|
|---|---|---||/discovery|DiscoveryDocument|DiscoveryV1|