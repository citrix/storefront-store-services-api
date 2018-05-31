#Appendix A: Keywords and Properties Parsing

The resource enumeration Xml return by the resource enumeration service contains keywords and properties elements. For resources derived from XenApp, XenDesktop or VDI-in-a-Box, these values are generated from mark-up in the description field of the corresponding resource. The description field is then 'tidied up' to remove this mark-up.

The keyword/property parser looks for the string KEYWORDS: (case-insensitive). If found, all text after that string is treated as keyword/property mark-up. The mark-up is made up of the following tokens separated by one or more spaces:

* <property>=<value>
* <property>="<quoted-value>"
*  <keyword>

Where:

* <property>, <value> and <keyword> are strings made up of any characters other than space, = or "
* <quoted-value> is a string made up of any character other than "

The keywords are collected into a single list.

All property entries with the same property name are collected togther to give a multiple-valued property. 

Some concrete examples:|Original Description in XenApp/XenDesktop (plus comment)|Description as returned by StoreFront Services.|Keywords|Properties|
|---|---|---|---||"My description" <br> (Basic desciption without mark-up)|"My description"|\<keywords/>|\<properties/>|
|"My description KEYWORDS: kw1 kw2 (Description with keyword mark-up)|"My description"|\<keywords> <br>\<keyword>kw1\</keyword> <br>\<keyword>kw2\</keyword><br> \</keywords>|\<properties/>||"My description keywords: name1=a name2=b name1="the rain in Spain" (Description with keyword and property mark-up)|"My description"|\<keywords/>|\<properties><br>\<property propertyId="name1"><br>\<value>a\</value>\<value>The rain in Spain\</value> \</property>\<property propertyId="name2"><br>\<value>b\<value><br>\</property><br>\</properties>|
|"My description keywords: name1=a name2=b" (Description with misformatted mark-up - a misspelled KEYWORDS: token)|"My description keywrds: name1=a name2=b "|\<keywords>|\<properties/>||"My description KEYWORDS: kw1 name1=a kw2 name2=b kw3" (Description with interspersed keywords and properties.)|"My description"|\<keywords> <br>\<keyword>kw1\</keyword> <br>\<keyword>kw2\</keyword><br>\<keyword>kw3\</keyword><br>\<keywords>|\<properties><br>\<property propertyId="name1"><br>\<value>a\</value> <br>\</property><br>\<property propertyId="name2"><br>\<value>b\</value><br>\</property><br>\</properties>| 
|"My description KEYWORDS: name1=a Name1=b "(Property names are case- sensitive.)|"My description"|\<keywords/>|\<property propertyId="name1"><br>\<value>a\</value><br>\</property><br>\<property propertyId="Name1"><br>\<value>b\</value><br>\</property><br>\</properties>|