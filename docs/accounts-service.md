#Accounts Service
This service is not specific to a particular store, but spans all stores in an installation. Whereas each store has a base url of http(s)://<host>/Citrix/<StoreName>/ (where <host> is the single server or cluster hostname for the installation, and <StoreName> the name of the store), the accounts service has the base url http(s)://<host>/Citrix/Roaming/

##Request
|URL|Method|Description|
|---|---|---||/Citrix/Roaming/accounts|GET/POST|Returns the list of available accounts with full details.||/Citrix/Roaming/accounts?summary|GET/POST|Returns the list of available accounts, as summaries, i.e. no details.|

**Note**: These requests may require an Authorisation token depending on the Stores configuration. Authorisation is only required if all Stores require authentication and share the same authentication service and authentication protocols. Future releases may remove authentication entirely from this service.

##Response

|Response Code|Description|
|---|---||200|Success|

##Success Response Content
In the case that a successful reponse is returned, the response body contains an Xml document giving the list of accounts available, described by the schema: /Schemas/Accounts.xsd

**Notes**:

* For the StoreFront Services 1.2 release, a few simplifications apply:
	* No trust settings or properties are defined.
	* Plug-ins are only defined at the account level.

###Content Hash
The <contentHash> element contains a hash of the account content that can be used by the client to determine whether the content has changed. This can be used in conjunction with the summary parameter to optimize network bandwidth usage. The algorithm used to generate the hash may be changed in later revisions, so the client should not treat the contentHash as anything other than an opaque string or make assumptions about the length or format.

###Published
Indicates whether the account is published. Published accounts are presented to the user by preference over unpublished accounts.

**Note**: The xs:boolean type allows ‘true’/’false’ or, equivalently, ‘1’/’0’ values to be used so clients should parse both correctly.

###MetaData
From the Accounts Xml format, metadata (plug-ins, trust settings and properties) can be specified at the account or individual service level. Metadata specified at the account level should be assumed to apply to all services in the account, individual services may choose to add extra metadata or override the account-level values.

###Plug-ins
The plug-in values which may be returned by the account service are as follows:
|Plug-in value|Description|
|---|---|
|A9852000-047D-11DD-95FF-0800200C9A66|Receiver (Online)|
|2C882560-4469-4E65-9EBE-4A842840F273|Offline Plug-in||b9852000-041d-11ff-25ff-0800400c9a66|VPN||73cf72ab-2528-4836-8b69-062a92da02bb|ShareFile Outlook Plug-in||18952f2a-6e32-4606-9cc2-48dab425ba36|ShareFile Desktop Widget||DF0F00A4-BB7E-4af5-915F-F0A674B2264A|HDX RealTime Media Engine (aka ‘Lync Accelerator’) from Acosta.||4122F203-D87B-4e9c-BE7A-222E4E8A9C6F|Acceleration Plug-in (not returned by StoreFront Services 1.2)|
**Note**: These are equivalent values to those used by Merchandising Server

###Trust Settings
These specify the trust settings for various aspects of the client. Values can be Yes, No or Ask, indicating whether settings should be enabled, disabled or whether the user should be prompted to choose.

###Properties
These allow arbitrary data to be associated with a service and are intended as an extension mechanism.

|Response Format|Request Accept /Response Content-Type Header|
|---|---||Xml|application/vnd.citrix.roamingaccounts+xml|

##Account Service Example
This is an example of a client discovering the latest account information for a user. It assumes that the client has already authenticated and has a suitable authorization token, so the authentication exchange is not shown. This example illustrates the use of contentHash and the summary request. This does not imply that using the summary request is necessarily the best way to query account information as the cost of two network round-trips may outweigh the saved bandwidth for occasions where the second request is saved because the account data has not changed.

###Example: Request Summary Account Data
In this example, the client requests summary account data to find out if any of the accounts have changed, been added or removed since the client last retrieved account data, which it has cached, including the contentHash values:

####Request
```
GET https://storefront01/Citrix/Roaming/accounts/?summary HTTP/1.1 Accept: application/vnd.citrix.roamingaccounts+xml
Authorization: CitrixAuth H4sIAA........
Host: storefront01
```
```
Response
HTTP/1.1 200 OK
Cache-Control: public, no-store, max-age=0
Content-Type: application/vnd.citrix.roamingaccounts+xml Content-Length: xxxxx
<?xml version=”1.0” encoding=”utf-8”>
<accounts xmlns="http://citrix.com/delivery-services/2-1/accounts"
xmlns:sr="http://www.citrix.com/ServiceRecord"> <account>
<id> 775552B2-3E81-4FDA-93D1-CE0374E902AD</id> <name>Store</name>
<description></description> <published>true</published> <contentHash>AAA34596786767...345DEF</contentHash>
  </account>
  <account>
<id>1234DEF-1111-4FDA-5555-1234567890</id> <name>Store2</name>
<description></description> <published>false</published> <contentHash>BBB3373737377...123CCC</contentHash>
</account>
</accounts>
```

The data has changed so the client requests the full data:

####Request
```
GET https://storefront01/Citrix/Roaming/accounts/ HTTP/1.1
Accept: */*
Authorization: CitrixAuth H4sIAA........
Host: storefront01
Response
HTTP/1.1 200 OK
Cache-Control: public, no-store, max-age=0
Content-Type: application/vnd.citrix.roamingaccounts+xml Content-Length: xxxxx
```
```
<?xml version=”1.0” encoding=”utf-8”>
<accounts xmlns="http://citrix.com/delivery-services/2-1/accounts"
xmlns:sr="http://www.citrix.com/ServiceRecord"> <account>
<id> 775552B2-3E81-4FDA-93D1-CE0374E902AD</id> <name>Store</name>
<description></description> <published>true</published> <contentHash>AAA34596786767...345DEF</contentHash> <details>
      <updaterType>citrix</updaterType>
      <annotatedServices>
       <!-- Store service -->
       <annotatedService>
         <sr:Service type=”Store”>
<sr:SRID>1231231232312312</sr:SRID>
<sr:Name>Store</sr:Name> <sr:Address>https://storefront01/Citrix/Store/discovery</sr:Address> <sr:Gateways>
             <sr:Gateway ...>...</sr:Gateway>
           </sr:Gateways>
           <sr:Beacons>
<sr:Internal> <sr:Beacon>https://storefront01/Citrix/Store/discovery</sr:Beacon>
             </sr:Internal>
             <sr:External>
<sr:Beacon>http://www.example.com</sr:Beacon>
<sr:Beacon>http://www.microsoft.com</sr:Beacon> </sr:External>
           </sr:Beacons>
         </sr:Service>
         <metadata>
           <plugins/>
           <trustSettings/>
           <properties/>
         </metadata>
       </annotatedService>
       <!-- VPN service has gateways but no beacons or address -->
       <annotatedService>
<sr:Service type=”VPN”> <sr:SRID>454545454545454545454</sr:SRID> <sr:Name>VPN</sr:Name> <sr:Address></sr:Address>
           <sr:Gateways>
             <sr:Gateway ...>...</sr:Gateway>
           </sr:Gateways>
           <sr:Beacons/>
         </sr:Service>
         <metadata>
           <plugins/>
           <trustSettings/>
           <properties/>
         </metadata>
       </annotatedService>
      </annotatedServices>
      <metadata>
<plugins>
          <!-- offline & vpn plugins -->
          <plugin>A9852000-047D-11DD-95FF-0800200C9A66</plugin>
<plugin>b9852000-041d-11ff-25ff-0800400c9a66</plugin> </plugins>
<trustSettings/>
<properties/>
      </metadata>
    </details>
  </account>
  <account>
<id>1234DEF-1111-4FDA-5555-1234567890</id> <name>Store2</name>
<description></description> <published>false</published> <contentHash>BBB3373737377...123CCC</contentHash> <details>
       ...
    </details>
  </account>
</accounts>
```