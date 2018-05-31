#General Behavior
## Content Negotiation
The service shall respect the response format specified through the HTTP Accept header. If multiple such headers are supplied,
the first understood value is used.

If no supplied Accept header is recognised a HTTP 406 (‘Not Acceptable’) response is returned.

If no Accept header is sent, this is interpreted as \*/* in line with the HTTP/1.1 specification [1].

##Authentication
Some end-points are protected and require authentication information to allow access. If the required authentication information is not present, then the client will be challenged for authentication using an approach similar to that described by RFC2617 [2]. Full details of this mechanism can be found in the CitrixAuth Scheme document [3], including the reasons why authentication information may be required.

The Client should be prepared to respond to a challenge at any point, and in particular both Enumeration and Launch requests may require the client to obtain fresh authentication information.

##Localisation
In the case where a localised string is to be returned, the value specified in the Accept-Language header, shall be used where possible. If no value is specified in the header, then the default language shall be used.

##Hypermedia Control and Parameterised URLs
This API uses the concept of hypermedia controls, where links to resources and actions are obtained from the server, either from the end-points service or directly as the result of calling this API. This means that at no time should clients construct URLs.

In this document various parameterised URLs are discussed, such as /resources/{id}. It is important to note that these are indicative only, and again, the client code should not construct URLs directly. Instead, all parameterised URLs should be obtained from the service itself, for example the URL for a particular resource should be extracted from the Xml returned from the request to /resources/v2/.

In some cases (indicated in the relevant sections) additional parameters can be supplied to a service that is invoked by a GET HTTP request. These parameters are supplied to the service by adding query string parameters to the base service URL. Clients should not make assumptions on the form of the service URL, which may vary with different releases or within a single release depending on the particular circumstances. In particular clients should not make assumptions about the presence or non-presence of query string parameters in the base service URL and should determine in each case whether new parameters should be appended with ‘?’ or ‘&’.

##Backwards and Forwards Compatibility
The Store services are intended to be backwards and forwards compatible in the following ways:

* A client consuming this version of the Store service should be able to consume future versisons of the store without requiring changes.
* A client written to consume future versions of the Store should be able to consume this version, with possible degradation of service, unless it requires some future capabilities added in a later version.

	In order to achieve this there are some restrictions that the client must follow:
	
	* Most of the URLs supplied in this document are marked as for illustration only and should not be hardcoded within clients. These URLs can be obtained from other starting URLs which are defined relative to the hosting server or Store. For this release, these are:

		* Accounts service: defined relative to host.
		* Discovery Service: defined relative to the hosting Store. 
		* Endpoints Service: defined relative to the hosting Store.	
	* Use the endpoints service to identify URLs (the endpoint id and capabilities of each service are listed in this document).
	* Use the HTTP Accepts header to indicate which version of a response is required, avoid wildcards and ask explicitly for the content-types the client understands; ideally including older content-types for greater backwards compatibility. In the future, if the responses to certain services are changed significantly, then new schema and associated content-type will be defined. The Accepts header of the HTTP request will be used to determine which version(s) of the response the client is able to handle and typically the most recent of those will be returned. For example, consider a service with 2 versions with associated content types v1 and v2:
		* An older client written when only v1 existed would request the content-type v1 and so receive the older v1 format which it understands.
		* A newer client with backwards 7ompatibility would request both v2 and v1 (in that order). Speaking to an older server it would get back the older v1 format, to a newer server the newer v2 format.
		* A newer client written without concerns for compatibility back to the older format would just request v2. It would work with newer servers, but not older servers where it would receive an HTTP 406 error response.
		* A ‘badly written’ older client which understood only the v1 format, but used \*/* as its accepts header, would work with older servers, but with newer servers it would receive a v2 format response which it would fail to parse.
	* Ignore unexpected extra elements or values in returned Xml. The schemas supplied in this specification should not be taken to preclude future versions from including extra elements. Where changes are significant, and not merely the addition of elements or attributes, new schemas and content types will be defined.

##Client Name, Client Address and Device Id

The client name and device id parameters are passed to a variety of individual Store services. These values are required for workspace control to function correctly. It is important that the client pass consistent data for these parameters to all services that take these parameters. For StoreFront versions up to and including 2.3 it is strongly encouraged that clients supply a value for both parameters. The client name should be the hostname of the client machine. The device id should be a value unique to the client device, eg the hostname, the same as the client name value. For clients unable to determine the local hostname, eg clients running within browsers with limited access to the client device OS, a random string, ideally persisted between sessions, is a possible approach.

If multiple clients are run on a single device, but use different client name or device id values, then they will be treated by the Store services as running on separate devices which may lead to apps disappearing and reappearing when performing session operations, e.g. querying and reconnecting to sessions.

The client address value identifies the client ip address and may be used on some XenApp/XenDesktop systems to determine whether a client should be presented with a particular resource. It may also be used to determine the best worker to be used to service a launch request. StoreFront will also use the HTTP request headers to identify the client ip address automatically if no explicit value is supplied. As for client name and device id, it is important that clients supply consistent client address parameters for all services.

**Note**: For StoreFront 2.0 a number of services have had client name and address parameters added, either as query strings or as XML elements in the POSTed data.


