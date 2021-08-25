# Senzing

# Mapping Specification & Attribute Dictonary
*Formerly the Generic Entity Specification*

```
Version: 3.0
```
```
Date: July 3, 2021
```

# Table of Contents

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
- Mapping Source Data
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
	

#  Overview

This document is for data experts, Data Engineering, ETL engineers, and systems engineering professionals evaluating or using Senzing Software to integrate their data sources.  

The Senzing Software (TM) performs entity resolution – determining when entities are the same or related within
and across data sources. This specification focuses on entities that are persons or organizations, such as customers, prospects, vendors, employees, and watch lists. It contains a dictionary of pre-configured attributes that are used to resolve and relate persons or companies and outlines the process of creating data sets with them so that they are readily consumable by the Senzing engine. The dictionary also serves to identify what features in the data is desirable to perform Entity Resolution. Data must be presented to the engine in either a JSON or CSV file format using the dictionary registered attributes contained in this specification prior to loading. 

## Dictionary of Registered Attributes

Senzing is an entity repository that helps locate records for the same entity across data sources. Think of it as a pointer system to where an entity’s records can be found. These are the fields required to tie the records in Senzing back to the contributing sources.

## Attributes for the record key

| Attribute Name| Type | Required | Example | Notes |
| ----------- | ----------- | ----------- | ----------- | ----------- |
| DATA_SOURCE| String | Optional | CUSTOMER | **IMPORTANT**: Data Source IDs must not exceed 25 charaters and be in UPPER CASE. This is an important designation for reporting. Naming your data sources allows you to distiqush between records that orgniate from different systems or special loads like watchlists.  |
| RECORD_ID | String | Desired | 1001 | If the records in your data source have a unique record ID use that value here. |


## Use of Record IDs

When loading records from specific sources, it is useful to prefix the record ID with the source.  

Examples:

| Data Source| Record ID | Mapped Record ID | Notes |
| ----------- | ----------- | ----------- |----------- |
| EMPLOYEES |	012934 | employees_012934|Include the data source name with the record id from that source|
| CUSTOMERS |	0012746 | customers_0012746|(same as above)|
| TRANSACTIONS |	gh342301 | transactions_gh342301| **Non-keyed Lists**: An event or transaction with identifying information about a non-keyed or external entity such as a money transfer to an external party. The record_id in this case would be the transaction_id. |
| SPREADSHEET |	leave blank | *system generated* | **Non-keyed Lists**: Leaving the record_id blanks will cause the sytem to generate one automatically.


**Important**

- Usually, an entire file of records will be assigned the same **DATA_SOURCE** which is why it is marked optional above. Both the G2Loader in the direct install and StreamLoader in the docker  install offer the ability to assign a default DATA_SOURCE to a file.  This is explained further in the *Loading Data Handbook.* {Link}


## Attributes for names of individuals or organizations

A name is a highly desirable feature to map. Most resolution rules will require a matching name.

| Attribute name | Type | Example | Notes |
| ----------- | ----------- | ----------- |----------- |
| NAME_TYPE| String | PRIMARY, ALIAS, NICK | Most data sources have only one name, but when there are multiple, there is usually one primary name and the rest are aliases. Whatever terms are used here should be standardized across the data sources included in your project.|
| NAME_ORG | String | Acme Tire Inc. | This is the organization name. |
| NAME_LAST | String | Smith | This is the last or sur name of an individual.|
| NAME_FIRST | String | Robert | This is the first or given name of an individual.|
| NAME_MIDDLE | String | John |This is the middle name of an individual.|
| NAME_PREFIX | String | Mr | This is a prefix for an individual's name such as the titles: Mr, Mrs, Ms, Dr, etc.|
| NAME_SUFFIX | String | SR | This is a suffix for an individual's name and may include generational references such as: JR, SR, I, II, III and/or professional designations such as: MD, PHD, PMP, etc. |
|NAME_FULL| String | Robert J Smith | **IMPORTANT** Parsed names are prefered where the Last, First, and Middle names seperate attributes in the data. Full name should only be used if parsed name is not avaiable. This is the full name of an individual. The system will not allow both a full name and the parsed names to be populated in the same set of name fields. [See handling duplicate columns later in this document.]|

**Important**

 - The "PRIMARY” **NAME_TYPE** helps select the best name to display for an entity. See Special
    attribute types and labels for when to use this. It is best to always specify name type.

 - The NAME_FULL attribute is provided if the parsed name fields are unavailable. You would not map both NAME_FULL and any other name fields in the same name segment.

 - If there is a common or nick name field, it represents a “second” name the individual is known by. In this case, map a second set of name columns duplicating the last name with the common name.

 - If using NAME_ORG then this record should be about an organization, not an individual i.e., do not map any of the individual name fields. You would not map both a NAME_ORG and any other name fields in the same name segment.

 - Sometimes there is both an organization name and a person name on a record, such as a contact list where you have the person and who they work for. In this case, you would map the person’s name as a name and the company as their employer. See Attributes for group associations for more information on this important distinction.


## Attributes for addresses

Addresses are important, especially when identifiers are not available. One of the more common resolutions will be made on name and address.

| Attribute name | Type | Example | Notes |
| ----------- | ----------- | ----------- |----------- |
|ADDR_TYPE | String | HOME | This is a code that describes how the address is being used such as: HOME, MAILING, BUSINESS*, etc. Whatever terms are used here should be standardized across the data sources included in your project.|
| ADDR_LINE1| String | 111 First St | This is the first address line and is required if an address is presented.|
| ADDR_LINE2 | String | Suite 101 | This is the second address line if needed.|
| ADDR_LINE3 | String | This is the third address line if needed.|
| ADDR_LINE4| String ||This is the fourth address line if needed.|
| ADDR_LINE5 | String ||This is the fifth address line if needed. |
| ADDR_LINE6 | String ||This is the sixth address line if needed. |
| ADDR_CITY | String | Las Vegas | This is the city of the address.|
| ADDR_STATE | String | NV |This is the state or province of the address.|
| ADDR_POSTAL_CODE | String | 89111 | This is the zip or postal code of the address.|
|ADDR_COUNTRY| String | US | This is the country of the address.|
|ADDR_FULL | String| |This is a single string containing the all address lines plus city,state, zip and country. Sometimes data sources have thisrather than parsed address. Only populate this field if theparsed address lines are not available.|
|ADDR_FROM_DATE | Date | 2016-01-14 | This is the date the entity started using the address if known. It is the used to determine the latest value of this type being used by the entity.|
|ADDR_THRU_DATE| Date |This is the date the entity stopped using the address if known.|

**Important**

- The **ADDR_FULL** attribute is provided if the parsed address fields are unavailable. You would not map both an **ADDR_FULL** and any other address fields in the same address segment.
    
- The "BUSINESS” **ADDR_TYPE** adds weight to physical business addresses. See Special attribute types and labels for when to use this.


## Attributes for phone numbers

Like addresses, phone numbers can be important, especially when identifiers are not available. A common resolution will be based on name, phone, and date of birth.

| Attribute name | Type | Example | Notes |
| ----------- | ----------- | ----------- |----------- |
| PHONE_TYPE | String | FAX, HOME, MOBILE | This is a code that describes how the phone is being used such as: HOME, FAX, MOBILE*, etc. Whatever terms are used here should be standardized across the data sources included in your project.|
|PHONE_NUMBER| String | 111-11-1111| This is the actual phone number. They can include country codes.|
|PHONE_FROM_DATE | Date | 2016-01-14 | This is the date the entity started using the phone number if known. It is the used to determine the latest value of this type being used by the entity. |
| PHONE_THRU_DATE | Date | 2021-07-06 |This is the date the entity stopped using the phone number if known.|

**Important**

- The "MOBILE” phone type adds weight to mobile phones. See Special attribute types and labels for when to use this.


## Attributes for physical and other attributes

Physical attributes can like DATE_OF_BIRTH help reduce over matching (false positives). Usually gender and date of birth are available and should be mapped if possible.

| Attribute name | Type | Example | Notes |
| ----------- | ----------- | ----------- |----------- |
|GENDER | String | M | This is the gender such as M for Male and F for Female. You can pass through  single letters or longer strings if expressing tranitioned or transitioning individuals |
|DATE_OF_BIRTH | String | 1980-05-14 | This is the date of birth for a person. NOTE: P|artial dates such as just month and day or just month and year are accepted.|
|DATE_OF_DEATH | String | 2010-05-14 | This is the date of death for a person. Again, partial dates are accepted|
|NATIONALITY| String | US |This is where the person was born and should contain a country name or code|
|CITIZENSHIP| String | UK | This is the country the person is a citizen of and should contain a country name or code.|
|PLACE_OF_BIRTH|String| US |This is where the person was born. Ideally it is a country name or code. However, they often contain city names as well.|
| RECORD_TYPE | String | PERSON | This is a good value to include if known and persons areresolving to organizations. Use standardized terms like PERSON and ORGANIZATION across all your data sources.|
|REGISTRATION_DATE | String | 2010-05-14 | This is the date the organization was registered, like date of birth is to a person.|
| REGISTRATION_COUNTRY | String | US | This is the country the organization was registered in, like place of birth is to a person. |

## Attributes for government issued identifiers

Government issued IDs help to confirm or deny matches. The following identifiers should be mapped if available.


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

**Important**

- A **TRUSTED_ID** is a very special identifier that will resolve records together even if they have different names, dates of birth, or other identifiers. For example, if the SSN of a data source is so trusted it  should resolve records despite other differences, it can also be mapped as a **TRUSTED_ID_NUMBER** with the **TRUSTED_ID_TYPE** of “SSN” to resolve within and across data  sources that are so trusted.

- A **TRUSTED_ID** can also be used to manually force records together or apart. Prior to use, please [read the documentation.](https://senzing.zendesk.com/hc/en-us/articles/360023523354-How-to-force-records-together-or-apart)
    
- Use  **OTHER_ID** sparingly! It is just a catch-all for identifiers you don't understand well but still want to use to help match records to entities. If you have custom identifiers, you need to configure your identifier in Senzing.  (See Additional Configuration for more information.) {link}


## Attributes for identifiers issued by organizations

The following identifiers have been added over time and can also be mapped if available.
 
| Attribute name | Data Type | Example | Notes |
| ----------- | ----------- | ----------- |----------- |
| ACCOUNT_NUMBER | String |  1234-1234-1234-1234|This is an account number such as a bank account,credit card number, etc.|
| ACCOUNT_DOMAIN | String |  VISA | This is the domain the account number is valid in.
| DUNS_NUMBER | String | 123123 |  The unique identifier for a company. ([Learn more.](https://www.dnb.com/duns-number.html))|
| NPI_NUMBER | String | 123123 | A unique ID for covered health care providers.  ([Learn more.](https://www.cms.gov/Regulations-and-Guidance/Administrative-Simplification/NationalProvIdentStand/))| 
|LEI_NUMBER | String | 123123 | A unique ID for entities involved in financial transactions. ([Learn more](https://en.wikipedia.org/wiki/Legal_Entity_Identifier))|


## Attributes for websites, emails and other social handles

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

## Attributes for group associations

The groups a person belongs to can also be useful for resolving entities. For example, two contact lists that only
have name and who they work for as useful attributes. 

Group associations do not create disclosed relationships{link}. Group associations help resolve entities whereas disclosed relationships help relate them.This does not create a disclosed relationship {link}, rather makes it easier to resolve entities within organizations. For example, If all you have in common between two data sources are name and who they work for, a group association can help resolve the Joe Smiths that work at ABC company together.

| Attribute name | Data Type | Example | Notes |
| ----------- | ----------- | ----------- |----------- |
| EMPLOYER_NAME| String |  ABC | Company | This is the name of the organization the person is employed by.|
| GROUP_ASSOCIATION_TYPE | String | MEMBER | This is the type of group an entity belongs to.| 
| GROUP_ASSOCIATION_ORG_NAME | String | Group | name | This is the name of the organization an entity belongs to.|
| GROUP_ASSN_ID_TYPE | String | DUNS | When the group a person is associated with has a registered DUNS identifier, place the type of identifier here.| 
| GROUP_ASSN_ID_NUMBER | String | 12345 | When the group a person is associated with has registered identifier, place the identifier here.| 

**Important**

- Group associations are subject to default entity thresholds to help reduce false positives and keep the system fast. Therefore they will not help resolve _all_ the employees of a large company across data sources. But they could help to resolve the smaller groups of executives, primary contacts, or owners of large companies across data sources.

## Mapping Source Data
We are here to help anyone! [Request a free mapping review session](https://senzing.zendesk.com/hc/en-us/requests/new)  with our Success Team.

Senzing entity resolution works best when as many relevant features as possible are mapped from your source data. In the next section, we will look at examples of features of map people and companies. We've provided the corresponding samples for people and companies in [CSV](https://github.com/missulmer/SZGESv3/tree/main/sample_csvs) and [JSON](https://github.com/missulmer/SZGESv3/tree/main/json_sample) via Github. 

These files contain the likely fields you will run into when mapping organizations to the default entity format. In fact, you should try to map as many of these fields as possible. 

**Sample Person Features**
- Primary name
- Date of birth and gender
- Passport, driver’s license, social security number, national insurance number
- Home and mailing addresses
- Home and cell phone numbers
- Email and social media handles
- Groups that they are associated with such as their employer name
- Any key dates, statuses or amounts that can help you find meaning in the matches. For instance, a vendor related to an employee who has influence over purchases is more important than the same vendor related to an employee that doesn’t.

** Sample Features for An Organization**

The [sample_organization.csv](https://github.com/missulmer/SZGESv3/tree/main/sample_csvs) can be used to map your data to if desired. There is also a corresponding [sample_organization.json](https://github.com/missulmer/SZGESv3/blob/main/json_sample/sample_company.json) file if you prefer.  The sample organization structure contains the follow fields:

- Primary name
- Tax ID number, like employer identification number
- Other ID numbers, like a Dunn and Bradstreet number
- Primary and mailing addresses
- Primary and other phone numbers
- Website and social media handles
- Any key dates, statuses or amounts that can help you find meaning in the matches. For instance, a current company you do business with who is on a watch list for bad reasons is more important than the same match to a company you did business with several years ago.

**Recommendation**

We recommend JSON due to its hierarchical structure which allows for multiple names, addresses, phones, etc to be presented in a single structure as one record may have only one address and another may have five. Where as CSV files are flat and multiple values must be presented as additional columns, so if the maximum number of addresses is five, then all rows in the csv file will include space for five addresses, whether needed for that record, or not.


## Creating JSON files

Below is the basic structure of a json record the engine can consume. Note that most attributes are at the root level. However, lists must be used when there are multiple values for the same attributes.

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
**[View on Github](https://github.com/missulmer/SZGESv3/blob/main/json_sample/sample_person.json)**

### Sample JSON Describing Companies

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
**[View on Github](https://github.com/missulmer/SZGESv3/blob/main/json_sample/sample_company.json)**

Have questions? [Ask our Success Team](https://senzing.zendesk.com/hc/en-us/requests/new). We are happy to help anyone.

## Creating CSV files

Mapping csv files is normally accomplished by replacing the column header names with the registered attributes names contained in this specification. 

Column names in a CSV must be unique, so multiple values for attributes such as multiple names, multiple addresses, must be prefixed with a term denoting the type of name, address, phone number. 

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

## Mapping CSV Files

A mapped csv file actually looks this and is usually accomplished by simply replacing the current column
headers with corresponding attribute labels and names in this specification.

| RECORD_ID | PRIMARY_LAST_NAME | PRIMARY_FIRST_NAME | PRIMARY_MIDDLE_NAME|
| ----------- | ----------- | ----------- | ----------- |
| 1001 | Adams | Alan | James |
| 1002| Baker | Betty | Lynn |
| 1003 | Cooper | Cindy | Rose |


Have questions? [Ask our Success Team](https://senzing.zendesk.com/hc/en-us/requests/new). We are happy to help anyone.

## Special attribute types and labels

Some features have special labels that add weight to them. For instance, you might find a whole family at a
“home” address, but only one company (or company facility) at its physical “business” address. The
following special labels can be used to augment a feature’s weight ...

| Feature | Lable | Notes | Usage |
| ----------- | ----------- | ----------- |----------- |
| NAME |PRIMARY | People can have aka’s and nicknames; companies can have dbas.When the system resolves multiple records into an entity, the most complete “primary” name will be chosen over any other type.| Common, needed To help select the best name to display for an entity. |
| ADDRESS | BUSINESS | Companies with multiple facilities or outlets often share corporate phone numbers and website addresses. | Use this label to help break matches based on their physical location. | Common, used to prevent overmatching of companies.|
| PHONE | MOBILE | Home and work phone numbersare usually shared. Use this labelto add weight to mobile or “cell” phones as they are shared far less often. | Rare.Only apply if data source reliably uses mobile phones to distinguish entities.|

**How to use:**

**JSON Example**
~~~
"ADDRESS_LIST": [{
//Use the “type”attribute in a json list
    "ADDR_TYPE": "BUSINESS", 
    "ADDR_LINE1": "111 First St",
    "ADDR_CITY": "Anytown",
~~~

**CSV Example**
| BUSINESS_ADDR_LINE1 | BUSINESS_ADDR_CITY |
| ----------- |  ----------- | 
|111 First St | Anytown |

## Disclosed Relationship Mapping

### Understanding Disclosed Relationships

What do we mean by the term 'disclosed relationship'?  Records describing people or organizations may contain specific fields that designate the presence of a relationship, like a spouse, parental role, company owner, and more.  Disclosed relationships differ from what Senzing calls 'discovered relationships' because the connection between the entities is *disclosed* in the record data instead of *discovered* in the entity resolution process using data from multiple records.

Disclosed relationships are essential when systems or people make decisions based on a contextual awareness known relationships between entities. Some data sources keep track of known relationships between entities, such as familial relationships and company hierarchies. This structure allows you to tell the Senzing software about such relationships. Look for a table within the source system that defines such relationships and use them to implement your disclosed relationship mappings.

Examples of disclosed relationships include but are not limited to:

-   Spouse
-   Mother
-   Father
-   Parent (company)
-   Employer
-   Beneficial owner

These relationships can be hierarchical or one-directional, like in the case of "Parent Company."

![Heirarchical Relationship](https://github.com/missulmer/SZGESv3/blob/main/md_images/Simple%20Hierarchy.png)

At the same time, others may be bi-directional, like Father, Son, Husband, or Wife. Directional relationships go from one entity to another.

![Detailed bi-directional roles](https://github.com/missulmer/SZGESv3/blob/main/md_images/Spouse%20Bi-directional.png)
  
  
###**Technical Terms**


| Name      | Description|
| ----------- | ----------- |
|Anchor Domain | Anchor Domains describe the feature grouping which identifies entities uniquely. (E.g. Customer_ID or Company_ID.)|
|Anchor Key | An anchor key is the unique identifier from within the domain that anchors the connection to an entity.|
|Pointer Domain |The pointer domain points back to the feature grouping (domain) which identifies entities. It will match the anchor domain.|
| Pointer Key |See Anchor Key above. The pointer key points to a specific entity by their ID to connect the relationship.|
|Pointer Role | The pointer role defines the relationship, like SPOUSE or PARENT.|

#### Figure 1: Domain, Pointer, Anchor and Key
![Parent Company Disclosed Relationship](https://github.com/missulmer/SZGESv3/blob/main/md_images/Domain%20Familial%20Detailed%20roles%20Labled.png)

### Attributes for Implmenting Disclosed Relationships


| Attribute name | Data Type | Example | Notes |
| ----------- | ----------- | ----------- |----------- |
| REL_ANCHOR_DOMAIN | String | CUSTOMER_ID | This code describes the domain of the rel_anchor_key.Thekey must be unique within the domain. For instance,customer systems might use the customer_id to define relationships.| 
| REL_ANCHOR_KEY | String | 1001| The rel_anchor_key along with the associated domain comprises a unique value that other records can “point” to in order to create a disclosed relationship. | | | | | REL_POINTER_DOMAIN | String | CUSTOMER_ID | See rel_anchor_domain above.
| REL_POINTER_KEY | String | 1001 | See rel_anchor_key above. A rel_pointer_domain and key on one record point to a rel_anchor_domain and key on another record to in order to create a relationship between them.|
| REL_POINTER_ROLE | String | SPOUSE | This is the role the anchor record plays in relationship to the pointer record. Note: Be careful not to use very long names here as so they are should appear on the line between two nodes on a graph.

## Disclosed Relationship JSON Examples

#### Parent Company
##### Figure 2: Company Hierarchy
![Parent Company Disclosed Relationship](https://github.com/missulmer/SZGESv3/blob/main/md_images/Domain%20Company%20labled.png)

~~~
JSON sample
~~~
#### Familial Spouse 
##### Figure 3: Bi-directional relationship
![Parent Company Disclosed Relationship](https://github.com/missulmer/SZGESv3/blob/main/md_images/Spouse%20Bi-directional%20labled.png)

~~~
JSON sample
~~~
#### Familial Detailed Roles 
##### Figure 4: Bi-directional relationship with detailed roles
![Parent Company Disclosed Relationship](https://github.com/missulmer/SZGESv3/blob/main/md_images/Domain%20Familial%20Detailed%20roles%20Labled.png)

~~~
{"DATA_SOURCE": "CUSTOMERS", 
  "ENTITY_TYPE": "GENERIC", 
  "RECORD_ID": "1", 
  "PRIMARY_NAME_LAST": "Hawklin", 
  "PRIMARY_NAME_FIRST": "Rob",
  "NAME_SUFFIX" : "Sr"
  "HOME_ADDR_LINE1": "2280 Stoney Lonesome Road", 
  "HOME_ADDR_CITY": "Pittston", 
  "HOME_ADDR_STATE": "PA", 
  "HOME_ADDR_POSTAL_CODE": "18640", 
  "HOME_PHONE_NUMBER": "570-300-5826", 
  "DATE_OF_BIRTH": "4/13/69", 
  "SSN_NUMBER": "189-50-0966", 
  "DRIVERS_LICENSE_NUMBER": "N9123912",
  "DRIVERS_LICENSE_STATE": "NV",
  
  "RELATIONSHIP_LIST": [{
    "REL_ANCHOR_DOMAIN": "CUSTOMER_ID", 
    "REL_ANCHOR_KEY": "1001"}, 
    {"REL_POINTER_DOMAIN": "CUSTOMER_ID", 
    "REL_POINTER_KEY": "1003",
    "REL_POINTER_ROLE": "Father"}]

    "RECORD_ID": "2", 
    "PRIMARY_NAME_LAST": "Hawklin", 
    "PRIMARY_NAME_FIRST": "Robby",  
    "NAME_SUFFIX" : "Jr"
    "HOME_ADDR_LINE1": "2280 Stoney Lonesome Road", 
    "HOME_ADDR_CITY": "Pittston", 
    "HOME_ADDR_STATE": "PA", 
    "HOME_ADDR_POSTAL_CODE": "18640", 
    "HOME_PHONE_NUMBER": "570-300-5826", 
    "DATE_OF_BIRTH": "7/23/81", 
    "SSN_NUMBER": "379-90-6843", 
    "DRIVERS_LICENSE_NUMBER": "N2549821",
    "DRIVERS_LICENSE_STATE": "NV",
    
    "RELATIONSHIP_LIST": [
      {"REL_ANCHOR_DOMAIN": "CUSTOMER_ID", 
      "REL_ANCHOR_KEY": "1003"}, 
      {"REL_POINTER_DOMAIN": "CUSTOMER_ID", 
      "REL_POINTER_KEY": "1001",
      "REL_POINTER_ROLE": "Son"}]
      }
~~~

#### Have questions?  [Click here](https://senzing.zendesk.com/hc/en-us/requests/new) for help implmementing disclosed relationships.



## Attributes for values that are not used for entity resolution

Sometimes it is desirable to include additional attributes that can help add further context to what's understood about an entity or relationships in the data. These attributes are not used for entity resolution because they do not help with the entity resolution but are useful additional information stored on the entity to inform people or systems.  Senzing comes with a default configuration for features and entities, this is sometimes refered to as 'generic' entity types or features thoughout our documentation, comments and code. Features not configured to create entities or find relationships pass through to the entity's profile. Examples include additional dates, statuses, types, flags, or aggregated amounts at the entity level.

**Examples:**

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


## Loading Data

**Important: Loading data requires you to convert your source data to the Senzing input format before loading.**  If you have not yet completed this step, please review the Mapping Section of this document.

There are a number of methods for loading data into Senzing.  Choose the method that best aligns with your use context:

**Senzing Evaluations:**
* Baremetal on Linux 
* Senzing AWS Evaluation via Marketplace
* Docker-Compose-Demo instance using:
	* RabbitMQ
	* Kafka

**Production Deployments:**
* AWS SQS
* RabbitMQ
* Kaf

## Additional Configuration

Senzing comes pre-configured with all the features, attributes, and settings you will likely need to begin resolving persons and organizations immediately.  However, when you need to configure new features for your entities.  A typical example of a custom feature(s) are identifiers unique to your data.

If you have unique features that you feel are imporant for resolving your entities, [contact Customer Success](https://senzing.zendesk.com/hc/en-us/requests/new) and we will guide you through feature creation.

Likewise, you have other entity or feature configuration questions, [please be in touch.](https://senzing.zendesk.com/hc/en-us/requests/new)
