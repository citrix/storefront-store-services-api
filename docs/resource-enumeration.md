#Resource Enumeration
##Request

|URL <span style="font-weight:normal">(indicative only)</span>|Method|Description|
|---|---|---|
|/resources/v2/ /resources/v2/withoutautoprovision (see Endpoints section)| GET | Returns a list of all resources available for the user in resources Xml format, with or without auto-provision.|
|/resources/v2/{id} | GET | Returns the details of specified resource in resource Xml format. This URL is given for illustrative purposes. The actual URL should be obtained from the enumeration result returned by the request to /resources/v2/.|

|URL parameter | Description
|---|---|
|{id} | The identifier of the resource. This is unique to the instance of the service.|

|Query string parameter | Description|
|---|---|
|group | Allows the requester to specify which resource elements should be returned by the enumeration. Rather than specify the elements individually, ‘groups’ are defined which give a group name associated with a collection of elements names (the mechanism by which groups are defined is not described here). <br> The default value (if no group is specified) is ‘**all**’. The ‘**all**’ group corresponds to requesting all available elements.  <br> Group names are matched regardless of case. Unrecognized group names are ignored. If multiple groups are specified then the union of the elements for each group is used. Multiple groups can be requested using multiple group parameters e.g. to request all elements in either the **core** or **sub** groups: .../resources/v2/?group=core&group=sub. <br>  The id element is always returned. If one or more groups are specified, but none of the group names are recognized, then resources containing just the id element will be returned. <br> See below for the default groups defined. This set of groups may be increased by Citrix or 3rd- party extensions in the future (the mechanisms for this extension are not discussed here). Note, however, that groups extensions will not be allowed to change the elements associated with a group.|
|subscriptionStatus | Allows the requester to specify that only resources with a specified set of status values should be returned. Multiple status values can be requested using multiple subscriptionStatus parameters. If no subscriptionStatus is supplied, resources are returned regardless of subscription status. <br> The valid values are those defined in the subscription schema for the subscriptionStatus element (see /Schemas/Subscriptions.xsd).|
|scope | Allows the scope to be specified (a concept in the Nfuse protocol). <br> Only a limited set of allowed values will be permitted. Other values will result in a HTTP 400 (Bad Request) response. The value check will be case-insensitive, although the value passed on to the underlying system will be of the defined case. <br> Allowed values: <pre>$PRELAUNCH$ </pre> and <pre> $ANONYMOUS_PRELAUNCH$ </pre> - As defined by the pre-launch optimizations for PNAgent. <br>The scope value will be passed in the Nfuse RequestAppData made to XenApp/XenDesktop.|
|clientName, clientAddress | See Client Name, Client Address and Device Id section above.|

**Notes**:

* These requests require an Authorisation token
* The URL for a request for a specific resource (i.e. /resources/v2/{id}) is obtained from the Response to resource enumeration (i.e. /resources/v2/).
* In all cases, for each request, the service will perform a fresh enumeration for the user and so will pick up any changes.
* Group and subscriptionStatus query string parameters can be applied in any combination (both, either, neither) and order.

##Response
|Response Code|Description|
|---|---|
|200|Success|
|401|Bad/Missing security token (see CitrixAuth Authentication Scheme document [3] )|
|404|No resource available for the specified {id} parameter (the resource may not exist or access to it may not be authorized for the requesting user).|

|Response Format|Request Accept /Response Content-Type Header|
|---|---|
|Resources Xml format|application/vnd.citrix.resources+xml|
|Resource Xml format|application/vnd.citrix.resource+xml|

The resources Xml format allows for extensions. The default Store service with no 3rd-party extensions will have the following elements which shall exist in the [http://citrix.com/delivery-services/2-0/resources] (‘res:’) and [http://citrix.com/delivery-services/2-0/subscriptions] (‘sub:’) namespaces.

|Extension element|Group(s)|Description|
|---|---|---|
|res:id|n/a|An identifier that is unique across this instance of the resources service. This value is always returned in resources regardless of specified groups.||Res:title|core|The user friendly name of the resource.||Res:link|core, status|The url to request the resource’s details.||Res:summary|core|A longer text description of the resource.||Res:path|core|The path defined in the XenApp/XenDesktop Farm for the resource||res:resourcetype|core|A string identifying the type of the resource. If no extensions are installed, this will be one of the values **‘Citrix.MPS.Application’**, **‘Citrix.MPS.Desktop’** or **‘Citrix.MPS.Document’**.||res:playsfiletypes|fta|A list of file extensions supported by the resource [Application only]||res:contentlocation|launch|The location (UNC or URL) of the content of a document resource. [Document only]||res:clienttypes|core|A value identifying the clients supported for launches. Known values include:<br> **‘ica30’**, **‘rade’**, **‘rdp’** (XenApp/XenDesktop), <br>**‘application/ios’**, **‘application/android’**(AppController), **‘g2m’**, **‘g2t’**, etc (Citrix Online Integration using Generic applications). <br><br>Future releases of the Store Service or using future versions of XenApp, XenDesktop or AppController may return additional values.||Res:rade|launch|Holds details on the Rade license type and offline mode.||Res:keywords|keywords|A list of keywords associated with the resource (see Appendix A: Keywords and Properties Parsing)||res:properties|keywords|A list of name value pairs associated with the resource (see Appendix A: Keywords and Properties Parsing)|
|res:images|images|A list of image sizes and color depths held by the StoreFront Services server (may be used to determine which image sizes are likely to give good results when requested by the client).||Res:launchica|launch|The url to which to make an ICA launch request (see Launch).||Res:launchrade|launch|The url to which to make a Rade (streaming) launch request (see Launch).||Res:image|core|The url to which to make a request for the best image for the resource (see Images and Icons).||Res:icon|icon|The url to which to request the ico file for the resource (see Images and Icons).||Res:enabled|core| <pre>true\|false </pre>value indicating whether the resource is enabled.||Res:machinepoweroff|vm|The url to which to send a request to trigger a shutdown of the physical/virtual machine supplying a desktop (see Machine Power Off). [XenDesktop desktops only]||res:mandatory|sub|<pre>true\|false</pre> value indicating whether the resource is mandatory. <br>Resources marked as mandatory are always subscribed and cannot be un-subscribed by the clients. <br>Clients should treat the resource as non mandatory if this element is missing.|res:showondesktop|core|<pre>true\|false</pre> value indicating whether the resource is should be shown as a desktop shortcut.<br>Clients should treat the resource as not to be shown on the desktop if this element is missing||res:showonstartmenu|core|<pre>true\|false</pre> value indicating whether the resource is should be shown as an entry on the start menu.<br>Clients should treat the resource as not to be shown on the start menu if this element is missing||res:startmenuroot|core|If the associated resource is to be shown on the start menu, this string value indicates the root location within the start menu structure.<br>Clients should treat the value as an empty string if this element is missing||res:startmenupath|core|If the associated resource is to be shown on the start menu, this string value indicates the sub-location beneath any specified start menu root location within the menu structure.<br>Clients should treat the value as an empty string if this element is missing||sub:subscriptionactions|sub|The url to which to send subscribe, unsubscribe or subscription- update requests for the current user and resource (see Subscriptions Service).<br>If this element is missing, clients are not able to change the resource subscription state, although the resource may still have subscriptionstatus and other subscription elements.||sub:subscriptionstatus|sub, status|The subscription status.||sub:subscriptionid|sub|The subscription id.||sub:subscriptiondata|keywords|The subscription properties.||res:featured|core|<pre>true\|false </pre> value. true if **featured** or **recommended** keywords are found (case-insensitive) otherwise false.||res:workflowenabled|core|<pre>true\|false</pre> value. true if workflow is enabled for the resource.||res:workflowwithoutclientinteraction|core|<pre>true\|false</pre> value. Set to true if workflow is enabled but does not require client interaction for the resource (typically due to the presence of the **WFI** keyword, but this may change in future). This value is only true if the server can determine with certainty that there will be no interaction. Due to the distributed nature of workflow integration, there may be situations where the server is unable to determine this value with certainty and so will return false.|
|res:imagehash|core|A string giving a hash of the resources full image data. If two resources have the same image hash value, then (barring the insignificant chance of a hash collision) they have the same image data.<br><br>For most purposes clients can use the res:image URL value to determine whether resources share image data.||sub:subscriptionquestionurl|workflow|The URL that the Receiver client should launch to allow the user to specify a reason for the subscription (the value of the SubscriptionQuestionURL subscription property).||sub:subscriptionreasontext|workflow|The user's provided reason for subscribing (the value of the SubscriptionReasonText subscription property).||sub:subscriptionresponsereason|workflow, status|The reason supplied by the approval system for allowing or rejecting a subscription, typically a value entered by a manager (the value of the SubscriptionResponseReason subscription property).||res:accessonetimeticket|launch|The URL to which a client can make a one-time request for a URL that can be used for a browser launch (also known as ‘pre-launch’ or ‘sso’ service).<br>The details of the access one-time service found at this URL are outside the scope of this document.||res:followmedataaccess|fmd|The URL to hold the location of the follow-me data (FMD) service for a resource.<br>The details of the service found at this URL are outside the scope of this document.||res:aggregatedresource|core|<pre>true\|false</pre>value. true if the resource is aggregated.||res:publisherresourceid|core|The publisher resource id. This id allows the publisher to uniquely identity a resource.||res:publishername|core|The name of the publisher (the farm name for XA).|

**Note**: The image and icon urls represent the image data at the point that the enumeration was performed. If, subsequently, the image data changes for the resource, these urls will not point to the updated images. On the other hand the urls are persistent in the sense that they will always point to the same image (or icon file), although they may return 404 after the image data has been updated on the XenApp/XenDesktop server.

The root <pre>resources </pre> element shall also have the following attributes:
|Attribute (namespace)|Value|Description|
|---|---|---|
|enumeration (http://citrix.com/delivery- services/2-0 resources)|full|All the aggregated Xml Services responded|
 |partial|Some of the aggregated Xml Services responded| |failed|None of the aggregated Xml Services responded||subscriptionsstatus (http://citrix.com/delivery- services/2-0/subscriptions)|enabled|Subscriptions enabled|  |disabled|Subscriptions disabled|  |unavailable|Subscriptions enabled, but database currently unavailable.|

##Endpoints
|URL (indicative only)|Id|Capabilities|
|---|---|---||/resources/v2/withoutautoprovision|ListResources|ResourcesEnumerationV2 + 'group:<groupname>' for each supported group name.||/resources/v2|ListResourcesWithAutoProvision|ResourcesEnumerationV2 AutoProvision + 'group:<groupname>' for each supported group name.|
##Auto Provisioning
This service offers optional auto-provisioning behaviour. With auto-provisioning, the enumeration service automatically subscribes an application if all of the following conditions are true:

* The application is an Auto-provisioned application.
* The application has never been subscribed.
* The application has no associated workflow for subscription.

##Mandatory Apps
All the apps which are marked as mandatory have the following subscriptions behaviour.

* They are always subscribed for all users.
* They cannot be unsubscribed by the Clients. 

##Example: Request all resources 

###Request
```
GET http://www.example.com/Citrix/Store/resources/v2?group=core&group=sub HTTP/1.1 Host: www.example.com
Accept: application/vnd.citrix.resources+xml
Accept-Encoding: gzip, deflate,gzip, deflate
Authorization: CitrixAuth H4sIAA........
Host: www.example.com
```

###Response
```
HTTP/1.1 200 OK
Cache-Control: public, no-store, max-age=0
Content-Type: application/vnd.citrix.resources+xml
Content-Length: xxxxxxx
```
```
<?xml version="1.0" encoding="utf-8"?>
<resources xmlns="http://citrix.com/delivery-services/2-0/resources"
xmlns:a="http://citrix.com/delivery-services/2-0/subscriptions"
           enumeration="full" a:subscriptionsstatus="enabled">
  <resource>
    <id>Controller.Notepad</id>
    <link>
<url>https://www.example.com/Citrix/Store/resources/v2/Q2l0cml4Lk1QUy5BcHAu</url> </link>
<title>Notepad</title>
<summary>Notepad</summary>
<path>\</path> <resourcetype>Citrix.MPS.Application</resourcetype> <clienttypes>
      <clienttype>ica30</clienttype>
      <clienttype>rdp</clienttype>
    </clienttypes>
    <rade>
      <licenseType>online</licenseType>
      <offlineMode>none</offlineMode>
    </rade>
    <images>
      <image size="48" depth="4"  />
      <image size="32" depth="4"  />
      <image size="24" depth="4"  />
      <image size="16" depth="4"  />
      <image size="48" depth="8"  />
      <image size="32" depth="8"  />
      <image size="24" depth="8"  />
      <image size="16" depth="8"  />
      <image size="256" depth="32"  />
      <image size="48" depth="32"  />
      <image size="32" depth="32"  />
      <image size="24" depth="32"  />
      <image size="16" depth="32"  />
    </images>
    <enabled>true</enabled>
    <launchica>
<url>https://www.example.com/Citrix/Store/resources/v2/Q2l0cml4Lk1QUy5BcHAu/launch/ica</url> </launchica>
<image>
<url>https://www.example.com/Citrix/Store/resources/v2/E4wNXorb0I5OHEvZlhsOWFZPQ--/image</url> </image>
<icon> <url>https://www.example.com/Citrix/Store/resources/v2/E4wNXorb0I5OHEvZlhsOWFZPQ--/icon</url>
</icon> <imagehash>E4wNXorb0I5OHEvZlhsOWFZPQ--</imagehash> <aggregatedresource>false</aggregatedresource> <publisherresourceid>Notepad<publisherresourceid> <publishername>Farm1</publishername> <mandatory>true</mandatory> <a:subscriptionactions>
<url>https://www.example.com/Citrix/Store/resources/v2/subscriptions/Q2l0cml4Lk1QUy5BcHAu</url> </a:subscriptionactions>
</resource>
  ...
</resources>
```
##Example: Request a specific resource

**Notes**: When a request is made for a specific resource, then the same Xml is returned as for the corresponding resource element in the resources enumeration using the same schema, as illustrated by the following example:

###Request
```
GET https://www.example.com/Citrix/Store/resources/v2/Q2l0cml4Lk1QUy5BcHAu HTTP/1.1 Accept: application/vnd.citrix.resource+xml
Authorization: CitrixAuth H4sIAA........
Host: www.example.com
Connection: Keep-Alive
```
###Response
```
HTTP/1.1 200 OK
Cache-Control: public, no-store, max-age=0
Content-Type: application/vnd.citrix.resource+xml
Content-Length: xxxxx
```
```
<?xml version="1.0" encoding="utf-8"?>
<resource>
  <id>Controller.Notepad</id>
  <link>
<url>https://www.example.com/Citrix/Store/resources/v2/Q2l0cml4Lk1QUy5BcHAu</url> </link>
<title>Notepad</title>
<summary>Notepad</summary>
<path>\</path> <resourcetype>Citrix.MPS.Application</resourcetype> <clienttypes>
    <clienttype>ica30</clienttype>
    <clienttype>rdp</clienttype>
  </clienttypes>
  <rade>
    <licenseType>online</licenseType>
    <offlineMode>none</offlineMode>
  </rade>
  <images>
    <image size="48" depth="4"  />
     ...
    <image size="16" depth="32"  />
</images>
  <enabled>true</enabled>
  <launchica>
<url>https://www.example.com/Citrix/Store/resources/v2/Q2l0cml4Lk1QUy5BcHAu/launch/ica</url> </launchica>
<image>
<url>https://www.example.com/Citrix/Store/resources/v2/E4wNXorb0I5OHEvZlhsOWFZPQ--/image</url> </image>
<icon>
<url>https://www.example.com/Citrix/Store/resources/v2/E4wNXorb0I5OHEvZlhsOWFZPQ--/icon</url> </icon>
<imagehash>E4wNXorb0I5OHEvZlhsOWFZPQ--</imagehash> <mandatory>true</mandatory> <a:subscriptionactions>
<url>https://www.example.com/Citrix/Store/resources/v2/subscriptions/Q2l0cml4Lk1QUy5BcHAu</url> </a:subscriptionactions>
</resource>
```
