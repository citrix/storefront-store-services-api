#Rade Service
The Rade client requires a proxying service to pass various messages to the XenApp server. The StoreFront Services will supply a service with the same behavior as the Rade service in Citrix Web Interface. This will therefore not follow the standard patterns of the other StoreFront services.

##Request
URL (indicative only)|Method|Description|
---|---|---|/resources/v2/streaming|POST|Pass the supplied POST body, which should be a well-formed rade 1.1 request Xml message, to the appropriate XenApp server. Returns the rade 1.1 response Xml message.|

**Note**: The URLs given above are for illustrative purposes only. The actual URL used in a request should be obtained from the .rad file supplied to the Rade client.

##ResponseResponse Code|Description|
---|---|200|For backwards compatibility, this response code is used in all circumstances.|