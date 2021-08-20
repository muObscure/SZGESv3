# Senzing

# Generic Entity Specification

```
Version: 3.0
```
```
Date: July 3, 2021
```

## Table of Contents

 - Overview
-  Dictionary of registered attributes
   - Attributes for the record key
   - Attributes for names of individuals or organizations
   - Attributes for addresses
   - Attributes for phone numbers
   - Attributes for physical and other attributes
   - Attributes for government-issued identifiers
   - Attributes for identifiers issued by organizations
   - Attributes for websites, emails, and other social handles
   - Attributes for group associations
- Mapping source data
	- Creating JSON files
   - Creating CSV files
   - Sample structure for a person
   - Sample structure for an organization
- Disclosed Relationship Mapping
   - Attributes for disclosed relationships
   - Attributes for values that are not used for entity resolution
- Special attribute types and labels
- Additional configuration
   - How to add a data source
   - How to add a new identifier
  
- Loading Data 
	- 	Baremetal with G2loader
	-  Docker-Compose
		- RabbitMQ
		- Kafka
	- AWS Deployments with SQS
		- Marketplace Evaluation
		-  AWS ECS (Non-AWS marketplace deployment)
	

## Overview

The Senzing Software (TM) performs entity resolution – determining when entities are the same or related within
and across data sources.

This specification focuses on entities that are persons or organizations, such as customers, prospects,
vendors, employees, and watch lists. It contains a dictionary of pre-configured attributes that are used to
resolve and relate persons or companies and outlines the process of creating data sets with them so that
they are readily consumable by the Senzing engine. The dictionary also serves to identify what information
is desirable to perform Entity Resolution.

Data must be presented to the engine in either a JSON or CSV file format using the dictionary registered
attributes contained in this specification. The advantage of JSON is that its hierarchical structure allows for
multiple names, addresses, phones, etc to be presented in a single structure as one record may have only
one address and another may have five. CSV files are flat and multiple values must be presented as
additional columns, so if the maximum number of addresses is five, then all rows in the csv file will include
space for five addresses, whether needed for that record, or not.

There are several ways to load data into Senzing

- You can use the G2Loader.py python script in the install package that loads JSON or CSV files. There
    is a demo of it here ... Exploratory-Data-Analysis- 1 - Loading-the-truth-set-demo
- You can instantiate “stream” loader processes that read from queues as demonstrated here ...
    github.com/Senzing/docker-compose-demo
- Or you can call Senzing directly from your current processes to add, update, delete or even search
    records on demand by calling the API directly as documented [on the Senzing docs](https://docs.senzing.com/python/senzing-G2Engine-reference.html#Insert)

In any of these cases, your source data must be mapped according to this specification.

## Dictionary of registered attributes

### Attributes for the record key

Senzing is an entity repository that helps locate records for the same entity across data sources. Think of it
as a pointer system to where an entity’s records can be found. These are the fields required to tie the
records in Senzing back to the contributing sources.


| Attribute Name| Type | Required | Example | Notes |
| ----------- | ----------- | ----------- | ----------- | ----------- |
| DATA_SOURCE| String | Optional | CUSTOMER | **IMPORTANT**: Data Source IDs must not exceed 25 charaters and be in UPPER CASE. This is an important designation for reporting. Naming your data sources allows you to distiqush between records that orgniate from different systems or special loads like watchlists.  |
| RECORD_ID | String | Desired | 1001 | If the records in your data source have a unique record ID use that value here. |



#### Use of Record IDs

When loading records from specific sources, it is useful to prefix the record ID with the source.  

Examples:

| Data Source| Record ID | Mapped Record ID | Notes |
| ----------- | ----------- | ----------- |----------- |
| EMPLOYEES |	012934 | employees_012934|Include the data source name with the record id from that source|
| CUSTOMERS |	0012746 | customers_0012746|(same as above)|
| TRANSACTIONS |	gh342301 | transactions_gh342301| **Non-keyed Lists**: An event or transaction with identifying information about a non-keyed or external entity such as a money transfer to an external party. The record_id in this case would be the transaction_id. |
| Spreadsheet |	leave blank | *system generated* | **Non-keyed Lists**: Leaving the record_id blanks will cause the sytem to generate one automatically.


**_Important notes:_**

- Usually, an entire file of records will be assigned the same **DATA_SOURCE** which is why it is
    marked optional above. Both the G2Loader in the direct install and StreamLoader in the docker
    install offer the ability to assign a default DATA_SOURCE to a file. 

*why do we want this here, shouldnt this information go in the 'loading data' section not in mapping?*


### Attributes for names of individuals or organizations

A name is a highly desirable feature to map. Most resolution rules will require a matching name.


| Attribute name | Type | Example | Notes |
| ----------- | ----------- | ----------- |----------- |
| NAME_TYPE| String | PRIMARY, ALIAS, NICK | Most data sources have only one name, but when there are multiple, there is usually one primary name and the rest are aliases. Whatever terms are used here should be standardized across the data sources included in your project.|
| NAME_ORG | String | Acme Tire Inc. | This is the organization name. |
| NAME_LAST | String | Smith | This is the last or sur name of an individual.|
| NAME_FIRST | String Robert | This is the first or given name of an individual.|
| NAME_MIDDLE | String J | This is the middle name of an individual.|
| NAME_PREFIX | String | Mr | This is a prefix for an individual's name such as the titles: Mr, Mrs, Ms, Dr, etc.|
| NAME_SUFFIX | String | MD | This is a suffix for an individual's name and may include generational references such as: JR, SR, I, II, III and/or professional designations such as: MD, PHD, PMP, etc. |
|NAME_FULL| String | Robert J Smith | **IMPORTANT** Parse names are prefered where the Last, First and Middle names seperate attributes in the data. Full name should only be used if parsed name is not avaiable. This is the full name of an individual. The system will not allow both a full name and the parsed names to be populated in the same set of name fields. [See handling duplicate columns later in this document.]|

**_Important notes:_**

 - The "PRIMARY” **NAME_TYPE** helps select the best name to display for an entity. See Special
    attribute types and labels for when to use this. It is best to always specify name type.

 - The NAME_FULL attribute is provided if the parsed name fields are unavailable. You would not map both NAME_FULL and any other name fields in the same name segment.

 - If there is a common or nick name field, it represents a “second” name the individual is known by. In this case, map a second set of name columns duplicating the last name with the common name.

 - If using NAME_ORG then this record should be about an organization, not an individual i.e., do not map any of the individual name fields. You would not map both a NAME_ORG and any other name fields in the same name segment.

 - Sometimes there is both an organization name and a person name on a record, such as a contact list
where you have the person and who they work for. In this case, you would map the person’s name
as a name and the company as their employer. See Attributes for group associations for more information on this important distinction.


### Attributes for addresses

Addresses are important, especially when identifiers are not available. One of the more common
resolutions will be made on name and address.

| Attribute name | Type | Example | Notes |
| ----------- | ----------- | ----------- |----------- |
|ADDR_TYPE | String | HOME | This is a code that describes how the address is being used such as: HOME, MAILING, BUSINESS*, etc. Whatever terms are used here should be standardized across the data sources included in your project.|
| ADDR_LINE1| String | 111 First St | This is the first address line and is required if an address is presented.|
| ADDR_LINE2 | String | Suite 101 | This is the second address line if needed.|
| ADDR_LINE3 | String | This is the third address line if needed.|
| ADDR_LINE4| String | This is the fourth address line if needed.|
| ADDR_LINE5 | String |This is the fifth address line if needed. |
| ADDR_LINE6 | String |This is the sixth address line if needed. |
| ADDR_CITY | String | Las Vegas | This is the city of the address.|
| ADDR_STATE | String | NV |This is the state or province of the address.|
| ADDR_POSTAL_CODE | String | 89111 | This is the zip or postal code of the address.|
|ADDR_COUNTRY| String | US | This is the country of the address.|
|ADDR_FULL | String| |This is a single string containing the all address lines plus city,state, zip and country. Sometimes data sources have thisrather than parsed address. Only populate this field if theparsed address lines are not available.|
|ADDR_FROM_DATE | Date | 2016-01-14 | This is the date the entity started using the address if known. It is the used to determine the latest value of this type being used by the entity.|
|ADDR_THRU_DATE| Date |This is the date the entity stopped using the address if known.|

**_Important notes:_**

- The **ADDR_FULL** attribute is provided if the parsed address fields are unavailable. You would not
    map both an **ADDR_FULL** and any other address fields in the same address segment.
    
- The "BUSINESS” **ADDR_TYPE** adds weight to physical business addresses. See Special attribute
    types and labels for when to use this.


### Attributes for phone numbers

Like addresses, phone numbers can be important, especially when identifiers are not available. A common
resolution will be based on name, phone, and date of birth.

| Attribute name | Type | Example | Notes |
| ----------- | ----------- | ----------- |----------- |
| PHONE_TYPE | String | FAX, HOME, MOBILE | This is a code that describes how the phone is being used such as: HOME, FAX, MOBILE*, etc. Whatever terms are used here should be standardized across the data sources included in your project.|
|PHONE_NUMBER| String | 111-11-1111| This is the actual phone number. They can include country codes.|
|PHONE_FROM_DATE | Date | 2016-01-14 | This is the date the entity started using the phone number if known. It is the used to determine the latest value of this type being used by the entity. |
| PHONE_THRU_DATE | Date | 2021-07-06 |This is the date the entity stopped using the phone number if known.|

**_Important notes:_**

- The "MOBILE” phone type adds weight to mobile phones. See Special attribute types and labels for
    when to use this.


### Attributes for physical and other attributes

Physical attributes can like DATE_OF_BIRTH help reduce over matching (false positives). Usually gender
and date of birth are available and should be mapped if possible.

| Attribute name | Type | Example | Notes |
| ----------- | ----------- | ----------- |----------- |
|GENDER | String | M | This is the gender such as M for Male and F for Female. You can pass through  single letters or longer strings if expressing tranitioned or transitioning individuals |
|DATE_OF_BIRTH | String | 1980-05-14 | This is the date of birth for a person. NOTE: P|artial dates such as just month and day or just month and year are accepted.|
|DATE_OF_DEATH | String | 2010-05-14 | This is the date of death for a person. Again, partial dates are accepted|
|NATIONALITY| String | US |This is where the person was born and should contain a country name or code|
|CITIZENSHIP| String | UK | This is the country the person is a citizen of and should contain a country name or code.|
|PLACE_OF_BIRTH|String| US |This is where the person was born. Ideally it is a country name or code. However, they often contain city names as well.|
| RECORD_TYPE | String | PERSON | This is a good value to include if known and persons areresolving to organizations. Use standardized terms like PERSON and ORGANIZATION across all your data sources.|
|REGISTRATION_DATE | String | 2010-05-14 This is the date the organization was registered, like date of birth is to a person.|
| REGISTRATION_COUNTRY | String | US This is the country the organization was registered in, like place of birth is to a person. |

### Attributes for government issued identifiers

Government issued IDs help to confirm or deny matches. The following identifiers should be mapped if
available.


| Attribute name | Type | Example | Notes |
| ----------- | ----------- | ----------- |----------- |
|PASSPORT_NUMBER | String | 123456789 | This is the passport number.
| PASSPORT_COUNTRY | String | US | This is the country that issued the ID.|
| DRIVERS_LICENSE_NUMBER | String | 123456789| This the driver’s license number.|
| DRIVERS_LICENSE_STATE | String | NV | This is the state or province that issued the driver’s license.|
|SSN_NUMBER | String |123-12-1234 | This is the US Social Security number.|
| SSN_LAST4 |String | 1234|  This is just the last4 digits of the SSN for use when the full SSN is not available.|
| NATIONAL_ID_NUMBER | String | 123121234 | This is the national insurance number issued by many countries. It is similar to an SSN in the US.|
| NATIONAL_ID_COUNTRY| String | CA | This is the country that issued the ID. |
| TAX_ID_TYPE | String | EIN | This is the tax id number for a company, asopposed to an SSN or NIN for an individual.|
| TAX_ID_NUMBER| String | 123121234 | This is the actual ID number.|
| TAX_ID_COUNTRY | String | US | This is the country that issued the ID.|
|  OTHER_ID_TYPE | String | CEDULA | This is the type of any other identifier, such as registration numbers issued by other authorities than listed above.| 
| OTHER_ID_NUMBER |  String | 123121234 | This is the actual ID number.|
| OTHER_ID_COUNTRY | String |  MX |  This is the country that issued the ID number.| 
| TRUSTED_ID_TYPE | String | TRUE_SSN | The type of ID that is to be trusted. See the note below
| TRUSTED_ID_NUMBER | String | 123-45-1234 | The trusted unique ID.

**_Important notes:_**

- A **TRUSTED_ID** is a very special identifier that will resolve records together even if they have
    different names, dobs, or other identifiers. For example, if the SSN of a data source is so trusted it
    should resolve records despite other differences, it can also be mapped as a
    **TRUSTED_ID_NUMBER** with the **TRUSTED_ID_TYPE** of “SSN” to resolve within and across data
    sources that are so trusted.
- A **TRUSTED_ID** can also be used to manually force records together or apart as described here...
    https://senzing.zendesk.com/hc/en-us/articles/360023523354-How-to-force-records-together-
    or-apart
- _Use *_ **_OTHER_ID_** _sparingly! It is just a catch all for identifiers you know nothing about but still want_
    _to use to help match. Therefore if you know anything about an identifier not listed above, you should_
    _add it as its own identifier as described here ..._ How to add a new identifier_._


### Attributes for identifiers issued by organizations

The following identifiers have been added over time and can also be mapped if available.
 
| Attribute name | Data Type | Example | Notes |
| ----------- | ----------- | ----------- |----------- |
| ACCOUNT_NUMBER | String |  1234-1234-1234-1234|This is an account number such as a bank account,credit card number, etc.|
| ACCOUNT_DOMAIN | String |  VISA | This is the domain the account number is valid in.
| DUNS_NUMBER | String | 123123 |  The unique identifier for a company. ([Learn more.](https://www.dnb.com/duns-number.html))|
| NPI_NUMBER | String | 123123 | A unique ID for covered health care providers.  ([Learn more.](https://www.cms.gov/Regulations-and-Guidance/Administrative-Simplification/NationalProvIdentStand/))| 
|LEI_NUMBER | String | 123123 | A unique ID for entities involved in financial transactions. ([Learn more](https://en.wikipedia.org/wiki/Legal_Entity_Identifier))|


### Attributes for websites, emails and other social handles

The following social media attributes are available.

| Attribute name | Data Type | Example | Notes |
| ----------- | ----------- | ----------- |----------- |
| WEBSITE_ADDRESS | String | somecompany.com |  This is a website address, usually only present for organization entities.|
| **EMAIL_ADDRESS** | String | someone@somewhere.com | This is the actual email address.|
| **LINKEDIN** | String | xxxxx | This is the unique identifier in for this socal domain.|
| **FACEBOOK** |String | xxxxx | This is the unique identifier in for this socal domain.|
| **TWITTER** |String | xxxxx | This is the unique identifier in for this socal domain.|
| **SKYPE** |String | xxxxx | This is the unique identifier in for this socal domain.|
| **ZOOMROOM** |String | xxxxx | This is the unique identifier in for this socal domain.|
| **INSTAGRAM** |String | xxxxx | This is the unique identifier in for this socal domain.|
| **WHATSAPP** |String | xxxxx | This is the unique identifier in for this socal domain.|
| **SIGNAL** |String | xxxxx | This is the unique identifier in for this socal domain.|
| **TELEGRAM** |String | xxxxx | This is the unique identifier in for this socal domain.|
| **TANGO** |String | xxxxx | This is the unique identifier in for this socal domain.|
| **VIBER** |String | xxxxx | This is the unique identifier in for this socal domain.|
| **WECHAT** |String | xxxxx | This is the unique identifier in for this socal domain.|

### Attributes for group associations

Groups a person belongs to can also be useful for resolving entities. Consider two contact lists that only
have name and who they work for as useful attributes.

| Attribute name | Data Type | Example | Notes |
| ----------- | ----------- | ----------- |----------- |
| EMPLOYER_NAME| String |  ABC | Company | This is the name of the organization the person is employed by.|
| GROUP_ASSOCIATION_TYPE | String | MEMBER | This is the type of group an entity belongs to.| 
| GROUP_ASSOCIATION_ORG_NAME | String | Group | name | This is the name of the organization an entity belongs to.|
| GROUP_ASSN_ID_TYPE | String | DUNS | When the group a person is associated with has a registered DUNS identifier, place the type of identifier here.| 
| GROUP_ASSN_ID_NUMBER | String | 12345 | When the group a person is associated with has registered identifier, place the identifier here.| 

**_Important Notes:_**

- Group associations should not be confused with disclosed relationships described later in this
    document. Group associations help resolve entities whereas disclosed relationships help relate
    them.
- If all you have in common between two data sources are name and who they work for, a group
    association can help resolve the Joe Smiths that work at ABC company together.
- Group associations are subject to generic thresholds to help reduce false positives and keep the
    system fast. Therefore they will not help resolve _all_ the employees of a large company across data
    sources. But they could help to resolve the smaller groups of executives, primary contacts, or
    owners of large companies across data sources.

## Mapping source data

### Creating JSON files

Below is the basic structure of a json record the engine can consume. Note that most attributes are at the
root level. However, lists must be used when there are multiple values for the same attributes.

### Sample JSON structure for a People

~~~
// a data source code is required, must be UPPER CASE and not greater than 25 char
"DATA_SOURCE": "CUSTOMER ",

////a record ID is always desired
"RECORD_ID": 1001,
//multiple name segments placed in a list
"NAME_LIST": [{
    "NAME_TYPE": "PRIMARY",
    "NAME_LAST": "Jones",
    "NAME_FIRST": "Robert",
    "NAME_MIDDLE": "M",
    "NAME_PREFIX": "Mr",
    "NAME_SUFFIX": "Jr" }],

"GENDER": "M",
"DATE_OF_BIRTH": "1/2/1981",
"PASSPORT_NUMBER": "PP11111",
"PASSPORT_COUNTRY": "US",
"DRIVERS_LICENSE_NUMBER": "DL11111",
"DRIVERS_LICENSE_STATE": "NV",
"SSN_NUMBER": "111-11-1111",

//multiple Address segments placed in a list, where there are multiple addresses you will choose a "TYPE" like HOME and MAIL
"ADDRESS_LIST": [{
    "ADDR_TYPE": "HOME",
    "ADDR_LINE1": "111 First St",
    "ADDR_CITY": "Las Vegas",
    "ADDR_STATE": "NV",
    "ADDR_POSTAL_CODE": "89111",
    "ADDR_COUNTRY": "US"
	},
	{ "ADDR_TYPE": "MAIL",
    "ADDR_LINE1": "PO Box 111",
    "ADDR_CITY": "Las Vegas",
    "ADDR_STATE": "NV",
    "ADDR_POSTAL_CODE": "89111",
    "ADDR_COUNTRY": "US" }],

////multiple phone numbers are placed in a list with a tagged TYPE. E.g. 'WORK' and 'CELL'
"PHONE_LIST": [{
    "PHONE_TYPE": "WORK",
    "PHONE_NUMBER": "800-201-2001" },

    { "PHONE_TYPE": "CELL",
    "PHONE_NUMBER": "702-222-2222" }],

"EMAIL_ADDRESS": "bob@jonesfamily.com",
"SOCIAL_HANDLE": "@bobjones27",
"SOCIAL_NETWORK": "twitter"}
~~~
*You can grap a person sample here : Github*

### Sample JSON Describing Companies

###JSON
~~~
{"DATA_SOURCE": "COMPANYDATA ",
"RECORD_ID": 2001,
"NAME_LIST": [{
"NAME_TYPE": "PRIMARY",
"NAME_ORG": "Presto Company" }],

"TAX_ID_NUMBER": "EIN11111",
"TAX_ID_COUNTRY": "US",
"ADDRESS_LIST": [{
    "ADDR_TYPE": "PRIMARY",
    "ADDR_LINE1": "Presto Plaza - 2001 Eastern Ave",
    "ADDR_CITY": "Las Vegas", "ADDR_STATE": "NV",
    "ADDR_POSTAL_CODE": "89111",
    "ADDR_COUNTRY": "US" },

	{"ADDR_TYPE": "MAIL",
    "ADDR_LINE1": "Po Box 111"
    "ADDR_CITY": "Las Vegas",
	"ADDR_STATE": "NV",
	"ADDR_POSTAL_CODE": "89111",
	"ADDR_COUNTRY": "US"
	}],

 "PHONE_LIST": [{
 "PHONE_TYPE": "PRIMARY",
 "PHONE_NUMBER": "800-201-2001" }],
 "WEBSITE_ADDRESS": "Prestofabrics.com",
 "SOCIAL_HANDLE": "@prestofabrics",
 "SOCIAL_NETWORK": "twitter"}

{"RECORD_ID": 2002,
"NAME_LIST": [{
	"NAME_TYPE": "PRIMARY",
	"NAME_ORG": "Presto Fabrics"
	}],

"ADDRESS_LIST": [{
	"ADDR_TYPE": "PRIMARY",
	"ADDR_LINE1": "2001 Eastern",
	"ADDR_CITY": "Las Vegas",
	"ADDR_STATE": "NV",
	"ADDR_POSTAL_CODE": "89222"
	}],

"PHONE_LIST": [{
	"PHONE_TYPE": "PRIMARY",
	"PHONE_NUMBER": "800-201-2001" }
	]}

{"RECORD_ID": 2003,
"NAME_LIST": [{
"NAME_TYPE": "PRIMARY",
"NAME_ORG": "Fabrics Unlimited, Inc"
}],

"TAX_ID_NUMBER": "EIN33333",
"TAX_ID_COUNTRY": "US",
"OTHER_ID_NUMBER": 33333,
"OTHER_ID_TYPE": "D&B",

"ADDRESS_LIST": [{
	"ADDR_TYPE": "PRIMARY",
	"ADDR_LINE1": "2003 Southern Highlands, Pkwy",
	"ADDR_CITY": "Las Vegas",
	"ADDR_STATE": "NV",
	"ADDR_POSTAL_CODE": "89333",
	"ADDR_COUNTRY": "US" }],

"PHONE_LIST": [{
	"PHONE_TYPE": "PRIMARY",
	"PHONE_NUMBER": "800-301-3001" }],
"WEBSITE_ADDRESS": "fabrics-unlimited.com"}

{"RECORD_ID": 2004,
	"NAME_LIST": [{
		"NAME_TYPE": "PRIMARY",
		"NAME_ORG": "Fabrics Unlimited"
		}],
"ADDRESS_LIST": [{
	"ADDR_TYPE": "PRIMARY", "ADDR_LINE1":
		"2004 Horizon Ridge",
		"ADDR_CITY": "Las Vegas",
		"ADDR_STATE": "NV",
		"ADDR_POSTAL_CODE": "89444"
		}],

"PHONE_LIST": [{
	"PHONE_TYPE": "PRIMARY",
	"PRIMARY_PHONE_NUMBER": "800-301-3001" }],

"WEBSITE_ADDRESS": "fabrics-unlimited.com"}
~~~~

### Creating CSV files

Mapping csv files is normally accomplished by replacing the column header names with the registered
attributes names contained in this specification. 

Column names in a CSV must be unique, so multiple values for attributes such as multiple names, multiple addresses, must be prefixed with a term denoting the
type of name, address, phone number. 

Here is the list of column headers that could be used to flatten out the JSON example above in the previous code blocks.


| Attribute      | Description|
| ----------- | ----------- |
| DATA_SOURCE   | Defaults to the setting in the project file if not supplied on every record |
| RECORD_ID   |  A record ID is always desired |
| PRIMARY_NAME_LAST   |  In the JSON this appears "NAME_TYPE": PRIMARY. Now PRIMARY is moved to the column prefix. |
| PRIMARY_NAME_FIRST   |  For each name feature in the PRIMARY list seen the JSON, PRIMAY becomes the column prefix|
| PRIMARY_NAME_MIDDLE  | see above  |
| AKA_NAME_LAST | Continue pattern of using the "NAME_TYPE": VALUE as the prefix continues across multi-attribute features|
| AKA_NAME_FIRST | |
| AKA_NAME_LAST | |
| AKA_NAME_MIDDLE | |
| DATE_OF_BIRTH | |
| SSN_NUMBER | |
| HOME_ADDR_LINE1 | Continue pattern of using the "ADDRESS_TYPE": VALUE "HOME" as the prefix across multi-attribute ADDRESSES|
| HOME_ADDR_CITY | |
| HOME_ADDR_STATE | |
| HOME_ADDR_POSTAL_CODE | |
| MAIL_ADDR_LINE1 | "ADDRESS_TYPE": MAIL, used for the column prefix of all attributes of the address|
| MAIL_ADDR_CITY | |
| MAIL_ADDR_STATE | |
| MAIL_ADDR_POSTAL_CODE | |
| CUST_SINCE_DATE | Fields not used for resolution can still be included with their original names |
| CUST_STATUS | See above |
| AGE_BRACKET | |
| INCOME_LEVEL | |

A mapped csv file actually looks this and is usually accomplished by simply replacing the current column
headers with corresponding attribute labels and names in this specification.

| RECORD_ID | PRIMARY_LAST_NAME | PRIMARY_FIRST_NAME | PRIMARY_MIDDLE_NAME|
| ----------- | ----------- | ----------- | ----------- |
| 1001 | Adams | Alan | James |
| 1002| Baker | Betty | Lynn |
| 1003 | Cooper | Cindy | Rose |


The sample_person.csv attached to this article can be used to map your data to if desired. There is also a
corresponding sample_person.json file if you prefer. These files contain the likely fields you will run into
when mapping persons to the generic entity format. In fact, you should try to map as many of these fields
as possible. The sample person structure contains fields for ...

- Primary name
- Date of birth and gender
- Passport, driver’s license, social security number, national insurance number
- Home and mailing addresses
- Home and cell phone numbers
- Email and social media handles
- Groups that they are associated with such as their employer name
- Any key dates, statuses or amounts that can help you find meaning in the matches. For instance, a
    vendor related to an employee who has influence over purchases is more important than the same
    vendor related to an employee that doesn’t.

### Sample structure for an organization

The sample_organization.csv attached to this article can be used to map your data to if desired. There is
also a corresponding sample_organization.json file if you prefer. These files contain the likely fields you will
run into when mapping organizations to the generic entity format. In fact, you should try to map as many of
these fields as possible. The sample organization structure contains fields for ...

- Primary name
- Tax ID number, like employer identification number
- Other ID numbers, like a Dunn and Bradstreet number
- Primary and mailing addresses
- Primary and other phone numbers
- Website and social media handles
- Any key dates, statuses or amounts that can help you find meaning in the matches. For instance, a
    current company you do business with who is on a watch list for bad reasons is more important
    than the same match to a company you did business with several years ago.

[Need Help?](https://senzing.zendesk.com/hc/en-us/requests/new) Schedule a mapping review session with Customer Success.

## Disclosed Relationship Mapping

### Understanding Disclosed Relationships


Disclosed relationships are essential when systems or people make decisions based on a contextual awareness from the data of known relationships between entities. Some data sources keep track of known relationships between entities, such as familial relationships and company hierarchies. This structure allows you to tell the Senzing software about such relationships. Look for a table within the source system that defines such relationships use them to implement your disclosed relationship mappings.

Examples of disclosed relationships include but are not limited to:

-   Spouse
-   Mother
-   Father
-   Parent (company)
-   Employer
-   Beneficial owner

These relationships can be hierarchical or one-directional, like in the case of "Parent Company."

![Heirarchical Relationship](https://github.com/missulmer/SZGESv3/blob/main/Simple%20Hierarchy.png)

At the same time, others may be bi-directional, like Father, Son, Husband, or Wife. Directional relationships go from one entity to another.

![Detailed bi-directional roles](https://github.com/missulmer/SZGESv3/blob/main/Domain%20Familial%20Detailed%20roles.png)
  
  
**Technical Terms**


| Name      | Description|
| ----------- | ----------- |
|Anchor Domain | Anchor Domains describe the feature grouping which identifies entities uniquely. (E.g. Customer_ID or Company_ID.|
|Anchor Key | An anchor key is the unique identifier from within the domain that anchors the connection to an entity.|
|Pointer Domain |The pointer domain points back to the feature grouping (domain) which identifies entities. It will match the anchor domain.|
| Pointer Key |See Anchor Key above. The pointer key points to a specific entity by their ID to connect the relationship.|
|Pointer Role | The pointer role defines the relationship, like SPOUSE or PARENT.|



### Attributes for Implmenting Disclosed Relationships


| Attribute name | Data Type | Example | Notes |
| ----------- | ----------- | ----------- |----------- |
| REL_ANCHOR_DOMAIN | String | CUSTOMER_ID | This code describes the domain of the rel_anchor_key.Thekey must be unique within the domain. For instance,customer systems might use the customer_id to define relationships.| 
| REL_ANCHOR_KEY | String | 1001| The rel_anchor_key along with the associated domain comprises a unique value that other records can “point” to in order to create a disclosed relationship. | | | | | REL_POINTER_DOMAIN | String | CUSTOMER_ID | See rel_anchor_domain above.
| REL_POINTER_KEY | String | 1001 | See rel_anchor_key above. A rel_pointer_domain and key on one record point to a rel_anchor_domain and key on another record to in order to create a relationship between them.|
| REL_POINTER_ROLE | String | SPOUSE | This is the role the anchor record plays in relationship to the pointer record. Note: Be careful not to use very long names here as so they are should appear on the line between two nodes on a graph.

### Disclosed Relationship JSON Examples

#### Parent Company

![Parent Company Disclosed Relationship](https://github.com/missulmer/SZGESv3/blob/main/Domain%20Company.png)

~~~
JSON sample
~~~
#### Familial Spouse (Bi-Directional)

![Parent Company Disclosed Relationship](https://github.com/missulmer/SZGESv3/blob/main/Spouse%20Bi-directional.png)

~~~
JSON sample
~~~
#### Familial Detailed Roles (Bi-Directional)

![Parent Company Disclosed Relationship](https://github.com/missulmer/SZGESv3/blob/main/Domain%20Familial%20Detailed%20roles.png)

~~~
JSON sample
~~~

#### Need Help?  [Click here](https://senzing.zendesk.com/hc/en-us/requests/new) for support implmementing disclosed relationships.


## Attributes for values that are not used for entity resolution

Sometimes it is desirable to include additional attributes that can help determine the importance of a
resolution or relationship. These attributes are not used for entity resolution because they are not
configured in Senzing. These attributes may include values such as additional dates, statuses, types, flags,
or aggregated amounts at the entity level.

For example:

- The LIFETIME_VALUE of a customer can help determine what kind of discount should be applied to
    their order or if a new customer is related to a high value customer.
- The TERMINATION_REASON of an employee can help determine if a new job applicant should be
    hired or not.
- The BUSINESS_RISK or GEOGRAPHICAL_RISK of a customer may help determine if a high dollar
    transaction should be reviewed before it is executed.
- A vendor related to an employee who has influence over purchases is more important than the
    same vendor related to an employee that doesn’t.
- A current company you do business with who is on a watch list for bad reasons is more important
    than the same match to a company you did business with several years ago.

On smaller Senzing systems, you may want to include and store additional non-Senzing attributes. On
larger Senzing systems, it is best practice to load only the configured attributes used for entity resolution,
and use a data warehouse or other external system to access additional non-Senzing attributes.


## Special attribute types and labels

Some features have special labels that add weight to them. For instance, you might find a whole family at a
“home” address, but only one company (or company facility) at its physical “business” address. The
following special labels can be used to augment a feature’s weight ...

| Feature | Lable | Notes | Usage |
| ----------- | ----------- | ----------- |----------- |
| NAME |PRIMARY | People can have aka’s and nicknames; companies can have dbas.When the system resolves multiple records into an entity, the most complete “primary” name will be chosen over any other type.| Common, needed To help select the best name to
display for an entity. |
| ADDRESS BUSINESS | Companies with multiple facilities or outlets often share corporate phone numbers and website addresses. | Use this label to help break matches based on their physical location. | Common, used to prevent overmatching of companies.|
| PHONE MOBILE | Home and work phone numbersare usually shared. Use this labelto add weight to mobile or “cell” phones as they are shared far less often. | Rare.Only apply if data source reliably uses mobile phones to distinguish entities.|

**How to use:**

Labels are either used as an attribute prefix for CSVs such as:
...
"BUSINESS_ADDR_LINE1": "111 First St ",
"BUSINESS_ADDR_CITY": "Anytown",
...

Or by its “type”attribute in a json list such as:
"ADDRESS_LIST": [{
"ADDR_TYPE": "BUSINESS", 
"ADDR_LINE1": "111 First St",
"ADDR_CITY": "Anytown",
...


## Loading Data

*This section needs to point to specific stack and data pub/sub/log based data pipeline methods. It doesnt belong in the Generic Ent Spec *

A project file is not necessary, but the G2Loader.py python script can either load files one by one with a -f
parameter or multiple files listed in a project file with the -p parameter. The project file itself may either
be a JSON file or a CSV file using the following column or tag names.

| Attribute name | Data Type | Example | Notes |
| ----------- | ----------- | ----------- |----------- |
| DATA_SOURCE | String | CUSTOMER | This is an important designation for reporting. For instance, you may want to know how many customers are onwatch lists, or how many customers inone data set match customers from another. Choose your data source codes based on how you want your reports to appear.|
| FILE_FORMAT | String | CSV |  This is the format of the files for this source. Valid values are JSON, CSV, TAB,or PIPE.
| FILE_NAME | String | custdump_20160511.csv | This is the name of the file that contains the data for this data source.| 


### Data Loading  -- *Rethink this section since we have MQ, SQS, G2Loader, Stream-loader?*

When creating a file of JSON messages for the G2Loader program, each message should be on one line. It is
expecting a file of JSON messages, like so ...

```
{"DATA_SOURCE": "CUSTOMER ", "RECORD_ID": "1001", "NAME": [{"NAME_TYPE": "PRIMARY", "NAME_LAST": "Adams", ...
{"DATA_SOURCE": "CUSTOMER ", "RECORD_ID": "100 2 ", "NAME": [{"NAME_TYPE": "PRIMARY", "NAME_LAST": "Baker", ...
{"DATA_SOURCE": "CUSTOMER ", "RECORD_ID": "100 3 ", "NAME": [{"NAME_TYPE": "PRIMARY", "NAME_LAST": "Cooper", ...
```




### Multisource data loads- Adding a data source
* This needs to move

Adding a new data source is a simple as registering the code you want to use for it. Most of the reporting
you will want to do is based on matches within or across data sources. For instance ...

- If you want to know when a customer record matches a watchlist record, you should have a data
    source named “CUSTOMER” and another one named “WATCHLIST”.
- Or if you are matching two customer data sources to find the overlap, you should have one data
    source named CUSTOMER1 and another named CUSTOMER2 or to be more descriptive you might
    name them based on the line of business such as “BANKING-CUSTOMER” and “MORTGAGE-
    CUSTOMER”.

To add a new data source named “CUSTOMER”, at the the G2ConfogTool prompt type ...

```
addDataSource CUSTOMER
```
## Additional configuration

Senzing comes pre-configured with all the features, attributes, and settings you will likely need to begin
resolving persons and organizations immediately. The only configuration that really needs to be added is
what you named your data sources.

The way you configure Senzing is through the G2ConfigTool.py script located on the python folder in the
directory tree for your project. To use it, go to the python folder and type ...

```
> G2ConfigTool.py
```
Then type “help” at the prompt. There is a lot you can do in there, but most of it you should not use unless
directed to do so by Senzing support. For instance, adding new rules or adjusting thresholds should not be
attempted without first contacting support for guidance.

However, you will often use this tool to add new data sources and sometimes to add new identifiers that
are not in our default for configuration.

### How to add a new identifier

// *This section seems to need to not be part of the Generic Entity Specification.  It should be linked from this md.
This fits in the bucket of managing configuration of the engine.  It doesn't relate specficially to our out of the box behavior and data definitions.*//

Occasionally, you will need to add a new kind of identifier. This consists of adding a feature and its
associated attributes and there are templates to help you. At the G2ConfigTool prompt type “templateAdd
list” to see the current list of templates.

The Senzing engine performs behavior based matched which you can read more about here ... Principle-
Based-Entity-Resolution

For identifiers, the three behaviors you can choose from are ...

- **F1** is the default and means that only one person should have this identifier, but it is possible or
    even likely for them to have two. A match on name and address will not be broken there are
    conflicting F1s
- **F1E** adds the “exclusive” behavior which means they should only have one. A match on name and
    address will be broken if each record has a different F1E
- **F1ES** – Adds the “exclusive” and “stable” behaviors which adds more weight to the identifier. It will
    not only break matches, but it will help cement them. If two records share an F1ES but have a
    conflicting F1E, the match will not be broken. Conversely, if two records share an F1E but have a
    conflicting F1ES the match will be broken.

For identifiers, the two comparisons you can choose from are ...

- **ID_COMP** is the default and does a fuzzy comparison of identifier strings.
- **EXACT_COMP** which requires the identifier strings to match exactly. It is rare that you will want to
    switch to this as transposition errors in human entered strings are quite common.


While that is a lot to take in, look at it like this ...

- If you want the new identifier to break matches, make it an **F1E**
- If you want the new identifier to help ensure a match, make it an **F1ES**
- And if you are not entirely sure that it should do either, make it a plain **F1**. It is pretty rare to find
    reliable **F1E** and **F1ES** identifiers other than government issued ones we come pre-configured with.

Most new identifiers will be added like this ...

```
templateAdd {"feature": "MY_NEW_ID", "template": "global_id"}
```

This will create a feature and an attribute you can map to called: MY_NEW_ID. It will help make
matches; but will not help break matches. A person or company can have more than one of them.

Since ID_COMP is the default comparison, transposition errors will be considered close. If you want
to force an exact comparison, you would add it like this:

```
templateAdd {"feature": "MY_NEW_ID", "template": "global_id", “comparison”: “EXACT_COMP}
```
If you have a state bounded identifier, where the same ID number might be issued by different states you
would add it like this:

```
templateAdd {"feature": "MY_NEW_ID", "template": "state_id"}
```

This will create a feature called MY_NEW_ID with the attributes: MY_NEW_ID_NUMBER and
MY_NEW_ID_STATE that you can map to.

It can only compared with ID_COMP due to the fact that some sources might supply the state field
while others may not and human entered ID numbers and state codes are often mistyped.

If you wanted to add weight to this new identifier, you would add it like this ...

```
templateAdd {"feature": "MY_NEW_ID", "template": "state_id", “behavior”: “F1E}
```
And for a country bounded identifier, you would just replace the template **state_id** with **country_id** above.


