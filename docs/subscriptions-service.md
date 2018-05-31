#Subscriptions Service
This service allows resources to be subscribed and unsubscribed, and to have data associated with each subscription.

##Request|URL (indicative only) and Content-Type|Method|Description|
|---|---|---||/resources/v2/subscriptions/{resourceId} application/vnd.citrix.subscriptionupdate+xml|POST|Action depends upon updateType attribute value:<br>**subscribe**: Try to subscribe to the resource. This may create a new subscription (if one does not already exist) and by default will set the subscription to the subscribed state (although later extensions may complicate this behaviour).<br>**unsubscribe** : Try to unsubscribe from the resource. This may delete the subscription and by default will set the subscription to the notSubscribed state (although later extensions may complicate this behaviour).<br>**update**: Update the data associated with the subscription. This should succeed independent of future extensions, although will return 404 (Not Found) if no subscription exists.|
|Parameter|Description|
|---|---|
|{resourceId}|The identifier of the associated resource. This is unique to the instance of the service.|
**Note**: The URLs given above are for illustrative purposes only. The actual URL used in a request should be obtained from the results of an enumeration request (eg /resources/v2/ or /resources/v2/{id}).

##Request data
The POSTed requests are Xml, described by the schema: /Schemas/Subscriptions.xsd, with top level element subscriptionUpdate.

##Response|Response Code (and friendly name)|Description|
|---|---||200 (OK)|Subscription added/removed successfully or conflict occurred, the response content-type indicates which. See Success Response Content and Conflict Response Content sections below. <br>The conflict response is returned in any of the following cases:<ol><li> A request is made to subscribe to a resource in an incompatible state (e.g. subscribed).</li><li> A request is made to unsubscribe from a resource in an incompatible state (e.g. unsubscribed).</li> <li> A request is made to update data for a resource with no subscription record.</li> <li> An unspecified data conflict occurs due to clash with another simultaneous request.</li> <li> An extension (e.g. workflow integration) indicates a conflict in state (see relevant extension specification for details).</li>||202 (Accepted)|The request to subscribe/unsubscribe has been accepted, however there is workflow associated with the subscription/unsubscription which must be completed before the action will be complete.||400 (Bad Request)|Missing {id} value or malformed data in the request body.||401 (Unauthorized)|Bad/Missing security token <br> (see CitrixAuth Authentication Scheme document [3])|
|403 (Forbidden)|The request token did not contain a user security identifier claim (this is required to give a unique identifier for the user for recording subscriptions)||404 (Not Found)|Subscriptions are not enabled or the supplied resourceId does not refer to a resource visible to the requesting user.<br>The reason is indicated by the X-Citrix-Error-Reason header value in the HTTP response:<br>**ResourceNotFound**: No resource was found for the requesting user for the specified id.<br>**SubscriptionsDisabled**: Subscriptions are not enabled for the Store service. (Not expected unless advanced customization is performed on the server.)<br>**SubscriptionsReadOnly**: Subscriptions are read only for the Store service|
|503 (Service Unavailable)|An extension is temporarily unable to contact a service it requires to contact as part of the subscription process (see relevant extension specification for details)| ##Success Response Content
In the case that a successful response (HTTP status code 200 (OK)) is returned, the response body contains an Xml document, described by the schema: /Schemas/Subscriptions.xsd with root node subscriptionResult. The response contains the subscription and resource ids, current status of the subscription, and also properties that contain data relating to the subscription action (this data is empty except in the case of extensions):

|Response Format|Request Accept /Response Content-Type Header|
|---|---||Xml|application/vnd.citrix.subscriptionresult+xml|


**Notes**:

* The data Xml element of the subscription result may contain properties that relate to the status change that has occurred as a result of the subscribe/unsubscribe action. This is intended as an extension mechanism.
* No account is made in the subscribe/unsubscribe actions what permissions, if any, the requesting user has on the resource identified by {resourceId}, other than that it is visible to that user.

##Subscription Status|Status|Description|
|---|---||notSubscribed|The resource is not subscribed.||subscribed|The resource is subscribed.||approved|Workflow has run and the user has been granted permission to use the resource. The subscription is awaiting user acknowledgement.||denied|Workflow has run and the user has been denied permission to use the resource. The subscription is awaiting user acknowledgement.||All other values|Workflow is running to determine whether the user may use the resource.|
###Example: Successful Unsubscription

####Request
```
POST http://www.example.com/Citrix/Store/resources/v2/subscriptions/Q2l0cm...Q- HTTP/1.1 Authorization: CitrixAuth H4sIA....
Host: www.example.com
Accepts: application/vnd.citrix.subscriptionresult+xml,
application/vnd.citrix.subscriptionconflict+xml Content-Type: application/vnd.citrix.subscriptionupdate+xml Content-Length: 269
```
```
<?xml version="1.0" encoding="utf-8"?>
<subscriptionUpdate xmlns="http://citrix.com/delivery-services/2-0/subscriptions"
       updateType="unsubscribe"
       updateDataMode="merge">
  <clientName>clientname</clientName>
</subscriptionUpdate>
```
####Response
```
HTTP/1.1 200 OK
Cache-Control: public, no-store, max-age=0
Content-Type: application/vnd.citrix.subscriptionresult+xml Date: Wed, 16 Feb 2011 17:31:43 GMT
Content-Length: xxxx
```
```
<?xml version="1.0" encoding="utf-8"?>
<subscriptionResult xmlns="http://citrix.com/delivery-services/2-0/subscriptions">
<subscriptionId>4C34BB98E543F084BAA33696C5659F63</subscriptionId> <resourceId>Citrix.MPS.App.Farm 1.Notepad</resourceId> <status>notSubscribed</status>
<data />
</subscriptionResult>
```

###Example: Successful Update of Subscription Data
This is an example of a successful update of the data associated with a subscription. The property ids and values are arbitrary from the perspective of the Resources service.

####Request
```
POST http://www.example.com/Citrix/Store/resources/v2/subscriptions/Q2l0cm...Q- HTTP/1.1 Authorization: CitrixAuth H4sIA....
Host: www.example.com
Accepts: application/vnd.citrix.subscriptionresult+xml,
application/vnd.citrix.subscriptionconflict+xml Content-Type: application/vnd.citrix.subscriptionupdate+xml Content-Length: xxxx
Expect: 100-continue
Connection: Keep-Alive
```
```
<?xml version="1.0" encoding="utf-8"?>
<subscriptionUpdate xmlns="http://citrix.com/delivery-services/2-0/subscriptions"
       updateType="update"
       updateDataMode="merge" >
  <data>
    <property key="Data1Key">
      <value>Data1 Value1</value>
      <value>Data1 Value2</value>
    </property>
  </data>
  <clientName>clientname</clientName>
</subscriptionUpdate>
```

####Response
```
HTTP/1.1 200 OK
Cache-Control: public, no-store, max-age=0
Content-Type: application/vnd.citrix.subscriptionresult+xml Content-Length: xxxx
```
```
<?xml version="1.0" encoding="utf-8"?>
<subscriptionResult xmlns="http://citrix.com/delivery-services/2-0/subscriptions">
<subscriptionId>4C34BB98E543F084BAA33696C5659F63</subscriptionId> <resourceId>Citrix.MPS.App.Farm 1.Notepad</resourceId> <status>subscribed</status>
<data />
</subscriptionResult>
```
####Conflict Response Content
In the case that a conflict is returned (HTTP status code 200 (OK)), a response potentially containing data pertaining to the conflict is returned in the response body:

|Response Format|Request Accept /Response Content-Type Header|
|---|---||Xml|application/vnd.citrix.subscriptionconflict+xml|

The subscription result content is Xml, described by the schema: /Schemas/Subscriptions.xsd with root node **subscriptionConflict**.

**Note**: The subscriptionconflict content type is intended to allow for extension, and will only contain properties in the case of extensions (e.g. workflow integration).

####Conflict Reason
The conflict response contains a reason attribute. The possible reason values are an open set allowing for workflow-integration- specific values in the future. The values defined in the base system are:|Reason value|Meaning|
|---|---||GeneralDataStoreConflict|An undetermined conflict within the subscription data storage (typically due to transaction conflicts).||MissingSubscription|A conflict due to a missing subscription.<br>(e.g. an attempt to unsubscribe or update a subscription to a resource that has never been subscribed.)||ConflictingSubscriptionState|A conflict due to the state of an existing subscription.<br>(e.g. an attempt to subscribe to a resource with a subscription with status subscribed.)||GeneralWorkflowConflict|A conflict caused by an undetermined conflict within the wokflow system.|
 ###Example: Unsuccessful Subscription
An example of an attempted subscription that is unsuccessful because the resource is already subscribed.

The subscription, if successful, would have associated a client-specific property named “dazzle:path” with the subscription. The property id and value are arbitrary from the perspective of the Resources service.

####Request
```
POST http://www.example.com/Citrix/Store/resources/v2/subscriptions/Q2l0cm...Q- HTTP/1.1 Authorization: CitrixAuth H4sIA....
Host: www.example.com
Accepts: application/vnd.citrix.subscriptionresult+xml,
application/vnd.citrix.subscriptionconflict+xml Content-Type: application/vnd.citrix.subscriptionupdate+xml Content-Length: xxxx
```
```
<?xml version="1.0" encoding="utf-8"?>
<subscriptionUpdate xmlns="http://citrix.com/delivery-services/2-0/subscriptions"
       updateType="subscribe"
       updateDataMode="merge" />
```
####Response
```
HTTP/1.1 200 OK
Cache-Control: public, no-store, max-age=0
Content-Type: application/vnd.citrix.subscriptionconflict+xml Content-Length: xxxx
```
```
<?xml version="1.0" encoding="utf-8"?>
<subscriptionConflict xmlns="http://citrix.com/delivery-services/2-0/subscriptions"
       reason="ConflictingSubscriptionState" >
 <data />
</subscriptionConflict>
```