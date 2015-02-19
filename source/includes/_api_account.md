# Account

<!--===================================================================-->
## Overview

The account service allows managing accounts by superusers and account_superusers.

<!--===================================================================-->
## Account Model

> Account Model

```json
{
	"id": "00004206",
	"name": "Greater Good",
	"owner_account_id": "00000001",
	"contact_first_name": null,
	"contact_last_name": null,
	"contact_email": "john.doe@eagleeyenetworks.com",
	"contact_street": [],
    "contact_city": null,
	"contact_state": null,
	"contact_postal_code": null,
	"contact_country": "US",
	"timezone": "US/Pacific",
	"utc_offset": -25200,
	"access_restriction": [
        "enable_mobile"
    ],
    "allowable_ip_address_range": [],
    "session_duration": 720,
    "holiday": [
        "20130101",
        "20130527",
        "20130704",
        "20130902",
        "20131128",
        "20131225"
    ],
    "work_days": "1111111",
    "work_hours": [
        "0800",
        "1730"
    ],
    "alert_mode": [
        "default"
    ],
    "active_alert_mode": "default",
    "camera_shares": [],
    "default_colo": null,
    "default_camera_passwords": null,
    "is_master": 0,
    "is_active": 1,
    "is_inactive": 0,
    "is_suspended": 0,
    "product_edition": null,
    "camera_quantity": null,
    "is_custom_brand_allowed": 0,
    "is_custom_brand": 0,
    "brand_logo_small": null,
    "brand_logo_large": null,
	"brand_subdomain": null,
    "brand_corp_url": null,
    "brand_name": null,
	"contact_phone": null,
    "cc_info": [
        {
            "city": "",
            "name": "",
            "default": "",
            "street1": "",
            "street2": "",
            "number": "",
            "state": "",
            "tag": "",
            "postal_code": "",
            "expire": "",
            "country": "US",
            "type": ""
        }
    ],
    "brand_support_email": null,
    "brand_support_phone": null,
    "map_lines": null,
    "contact_mobile_phone": null,
    "is_rtsp_cameras_enabled": 0,
    "contact_utc_offset": null
}
```

### Account Attributes

Parameter               | Data Type     | Description
----------------------- | -------------	| -----------
id 						| string 		| Unique identifier for the Account
name 					| string 		| Name of the account
owner_account_id 		| string 		| Id of parent account
contact_first_name 		| string 		| First name of primary contact for account
contact_last_name  		| string 		| Last name of primary contact for account
contact_email 			| string 		| Email of primary contact for account
contact_street 			| array[string] | Array of strings representing 1 or more parts of a street address of primary contact for account
contact_city 			| string 		| City of primary contact for account
contact_state 			| string 		| State/province of primary contact for account
contact_postal_code 	| string 		| Zip/postal code of primary contact for account
contact_country 		| string 		| Country of primary contact for account
timezone 				| string 		| ['US/Alaska' or 'US/Arizona' or 'US/Central' or 'US/Pacific' or 'US/Eastern' or 'US/Mountain' or 'US/Hawaii' or 'UTC']: Timezone of the account. Defaults to US/Pacific.
utc_offset 				| int 			| Signed integer offset in seconds of the timezone from UTC
access_restriction 		| array[string] | ['enable_mobile' or 'enable_ip_restrictions']: Each entry in the list contains a restriction. Possible values: 'enable_mobile' = If present this account can has access to mobile clients. 'enable_ip_restrictions' = if present, and if allowable_ip_address_ranges has been specified, limits logins to the address ranges specified.,
allowable_ip_address_range | array[string] | Each entry in the list specifies one address range. Entries use the ‘/’ format. For example, to limit access to 192.168.123.0-192.168.123.255, the entry would be 192.168.123.0/24. If no entries are present, 0.0.0.0/0 i s implied.
session_duration 		| int 			| session duration in minutes. Session duration of 0 means 'stay logged in forever'
holiday 				| array[string] | List of dates that during which holidays are observed. Format for dates is YYYYMMDD
work_days 				| string 		| String of length 7. Each position in the string corresponds to a day of the week. Monday is position 0, Tuesday is position 1, etc... Each character in the string can have a value of 1 or 0. 1 means that this day is a work day.
work_hours 				| array[string] | Two entries. Each entry contains a time expressed in local time. The first entry in the list is the work day start time . The second entry in the list is the stop time. Times are represented using a 24 hour clock and are formatted as HHMM. For example, 8AM would be 0800 and 5PM would be 1700.
alert_mode 				| array[string] | List of possible alert modes as defined for this account.
active_alert_mode 		| string 		| Must be blank or one of the values defined in alert_mode list.
default_colo  			| string 		| Name of the colo in which this account data for this account will be stored by default.
default_camera_passwords| string 		| Comma-delimited string of default camera passwords
camera_shares 			| array[array[string]]) | Array of arrays, with each sub array representing a camera to be shared to 1 or more email addresses. First element of sub array is action, with 'm' for add/update, and 'd' for delete. Second element of sub array is camera ID. Third element of sub array is an array of email addresses, but only applies to the 'm' action. Example: [['m', '12345678', ['test@testing.com','test2@testing.com]]]
is_master 				| int 			| ['0' or '1']: Indicates whether the account is a Master account (1) or not (0)
is_active 				| int 			| ['0' or '1']: Indicates whether the account is Active (1) or not (0)
is_inactive 			| int 			| ['0' or '1']: Indicates whether the account is inactive (1) or not (0)
is_suspended 			| int 			| ['0' or '1']: Indicates whether the account is Suspended (1) or not (0)
product_edition 		| string 		| The product edition the account is using
camera_quantity 		| int 			| The total number of cameras the account is allowed to use,
is_custom_brand_allowed | int 			| ['0' or '1']: Indicates whether the account is allowed to have branding (1) or not (0)
is_custom_brand 		| int 			| ['0' or '1']: Indicates whether the account has branding enabled (1) or not (0)
brand_logo_small 		| string 		| Base64 encoded image for the branded small logo
brand_logo_large 		| string 		| Base64 encoded image for the branded large logo
brand_subdomain 		| string 		| Sub domain for the branded url
brand_corp_url 			| string 		| Corporate web site url
brand_name 				| string 		| Branded company name

TODO Update Account Attributes

<!--===================================================================-->
## Get Account

> Request

```shell
curl -G https://login.eagleeyenetworks.com/g/account -d "id=[ID]&A=[VIDEOBANK_SESSIONID]"
```

Returns account object by Id

### HTTP Request

`GET https://login.eagleeyenetworks.com/g/account`

Parameter 	| Data Type   | Description | Is Required
--------- 	| ----------- | ----------- | -----------
**id**  	| string      | Account Id 	| true

### Error Status Codes

HTTP Status Code    | Data Type   
------------------- | ----------- 
200	| Request succeeded
400	| Unexpected or non-identifiable arguments are supplied
401	| Unauthorized due to invalid session cookie
403	| Forbidden due to the user missing the necessary privileges

<!--===================================================================-->
## Create Account

> Request

```shell
curl --cookie "videobank_sessionid=[VIDEOBANK_SESSIONID]" -X PUT -v -H "content-type: application/json" https://login.eagleeyenetworks.com/g/account -d '{"name": "[NAME]", "contact_first_name": "[CONTACT_FIRST_NAME]", "contact_last_name": "[CONTACT_LAST_NAME]", "contact_email": "[CONTACT_EMAIL]"}'
```

> Json Response

```json
{
    "id": "1234abcd"
}
```

Creates a new Account

### HTTP Request

`PUT https://login.eagleeyenetworks.com/g/account`

Parameter 				| Data Type   	| Description 	| Is Required
--------- 				| ----------- 	| ----------- 	| -----------
**name**  				| string  		| Name of Account	| true
owner_account_id  		| string      	| ID of parent account. If omitted, parent is set to the account of the creating user. | 
**contact_first_name**	| string      	| First name of primary contact for account | true
**contact_last_name**	| string      	| Last name of primary contact for account | true
**contact_email**		| string      	| Email of primary contact for account | true
contact_street			| array[string] | Array of strings representing 1 or more parts of a street address of primary contact for account
contact_city			| string      	| City of primary contact for account.
contact_state			| string      	| State/province of primary contact for account.
contact_postal_code		| string      	| Zip/postal code of primary contact for account.
contact_country			| string      	| Country of primary contact for account.
timezone				| string, enum  | Timezone of Account. Defaults to US/Pacific. <br><br>enum: "US/Alaska", "US/Arizona", "US/Central", "US/Pacific", "US/Eastern", "US/Mountain", "US/Hawaii", "UTC"
status					| array[string] | Account status. This can only be edited by superusers and account_superusers of the parent/owner account. 'realm_root' can only be set by superusers.
access_restriction		| array[string] | Each entry in the list contains a restriction. Possible values: 'enable_mobile' = If present this account can has access to mobile clients. 'enable_ip_restrictions' = if present, and if allowable_ip_address_ranges has been specified, limits logins to the address ranges specified.
allowable_ip_address_ranges | array[string] | Each entry in the list specifies one address range. Entries use the ‘/’ format. For example, to limit access to 192.168.123.0-192.168.123.255, the entry would be 192.168.123.0/24. If no entries are present, 0.0.0.0/0 i s implied.
session_duration		| int      		| Session duration in minutes. Session duration of 0 means 'stay logged in forever'
holiday					| array[string] | List of dates that during which holidays are observed. Format for dates is YYYYMMDD
work_days				| array[string] | String of length 7. Each position in the string corresponds to a day of the week. Monday is position 0, Tuesday is position 1, etc... Each character in the string can have a value of 1 or 0. 1 means that this day is a work day.
work_hours				| array[string] | Two entries. Each entry contains a time expressed in local time. The first entry in the list is the work day start time . The second entry in the ist is the stop time. Times are represented using a 24 hour clock and are formatted as HHMM. For example, 8AM would be 0800 and 5PM would be 1700.
alert_mode 				| array[string] | List of possible alert modes as defined for this account.
active_alert_mode 		| string      	| Must be blank or one of the values defined in alert_mode list.
default_colo			| string      	| Name of the colo in which this account data for this account will be stored by default.
default_camera_passwords| string      	| Comma-delimited string of default camera passwords
is_without_initial_user	| int      		| Indicates whether you want to NOT create an initial user with the new account. Defaults to 0, meaning an initial user (with is_account_superuser=1) will be created using the info from “contact_first_name/contact_last_name/contact_email” arguments of this new account. If this is set to 1, NO initial user will be created.
is_initial_user_not_admin| int      	| Indicates whether you want do NOT want the initial user to be an admin.

<!--===================================================================-->
## Update Account

> Request

```shell
curl --cookie "videobank_sessionid=[VIDEOBANK_SESSIONID]" -X POST -v -H "content-type: application/json" https://login.eagleeyenetworks.com/g/account -d '{"id": "[ACCOUNT_ID]", "contact_first_name": "[CONTACT_FIRST_NAME]"}'
```

> Json Response

```json
{
    "id": "1234abcd"
}
```

Update an Account

### HTTP Request

`POST https://login.eagleeyenetworks.com/g/account`

Parameter 				| Data Type   	| Description 	| Is Required
--------- 				| ----------- 	| ----------- 	| -----------
**id**  				| string  		| Unique identifier of Account | true
name  					| string  		| Name of Account
contact_first_name		| string      	| First name of primary contact for account
contact_last_name		| string      	| Last name of primary contact for account
contact_email			| string      	| Email of primary contact for account
contact_street			| array[string] | Array of strings representing 1 or more parts of a street address of primary contact for account
contact_city			| string      	| City of primary contact for account.
contact_state			| string      	| State/province of primary contact for account.
contact_postal_code		| string      	| Zip/postal code of primary contact for account.
contact_country			| string      	| Country of primary contact for account.
timezone				| string, enum  | Timezone of Account. Defaults to US/Pacific. <br><br>enum: "US/Alaska", "US/Arizona", "US/Central", "US/Pacific", "US/Eastern", "US/Mountain", "US/Hawaii", "UTC"
status					| array[string] | Account status. This can only be edited by superusers and account_superusers of the parent/owner account. 'realm_root' can only be set by superusers.
access_restriction		| array[string] | Each entry in the list contains a restriction. Possible values: 'enable_mobile' = If present this account can has access to mobile clients. 'enable_ip_restrictions' = if present, and if allowable_ip_address_ranges has been specified, limits logins to the address ranges specified.
allowable_ip_address_ranges | array[string] | Each entry in the list specifies one address range. Entries use the ‘/’ format. For example, to limit access to 192.168.123.0-192.168.123.255, the entry would be 192.168.123.0/24. If no entries are present, 0.0.0.0/0 i s implied.
session_duration		| int      		| Session duration in minutes. Session duration of 0 means 'stay logged in forever'
holiday					| array[string] | List of dates that during which holidays are observed. Format for dates is YYYYMMDD
work_days				| array[string] | String of length 7. Each position in the string corresponds to a day of the week. Monday is position 0, Tuesday is position 1, etc... Each character in the string can have a value of 1 or 0. 1 means that this day is a work day.
work_hours				| array[string] | Two entries. Each entry contains a time expressed in local time. The first entry in the list is the work day start time . The second entry in the ist is the stop time. Times are represented using a 24 hour clock and are formatted as HHMM. For example, 8AM would be 0800 and 5PM would be 1700.
alert_mode 				| array[string] | List of possible alert modes as defined for this account.
active_alert_mode 		| string      	| Must be blank or one of the values defined in alert_mode list.
default_colo			| string      	| Name of the colo in which this account data for this account will be stored by default.
default_camera_passwords| string      	| Comma-delimited string of default camera passwords
camera_shares 			| array[array[string]] | Array of arrays, with each sub array representing a camera to be shared to 1 or more email addresses. First element of sub array is action, with 'm' for add/update, and 'd' for delete. Second element of sub array is camera ID. Third element of sub array is an array of email addresses, but only applies to the 'm' action. Example: [['m', '12345678', ['test@testing.com','test2@testing.com]]]
is_revoke_admins		| int      		| Indicates whether you want to revoke all Admin permissions for the users in the account.
is_custom_brand			| int      		| Indicates whether branding is enabled for the account
brand_logo_small		| string      	| Base64 encoded image for the branded small logo
brand_logo_large		| string      	| Base64 encoded image for the branded large logo
brand_subdomain			| string      	| Sub domain for the branded url
brand_corp_url			| string      	| Corporate web site url
brand_name				| string      	| Branded company name


<!--===================================================================-->
## Delete Account

> Request

```shell
curl --cookie "videobank_sessionid=[VIDEOBANK_SESSIONID]" -X DELETE -v -H "content-type: application/json" https://login.eagleeyenetworks.com/g/account -d "id=[ACCOUNT_ID]" -G
```

Delete an Account

### HTTP Request

`DELETE https://login.eagleeyenetworks.com/g/account`

Parameter     | Data Type   | Description
---------     | ----------- | -----------
**id**        | string      | Account Id

### Error Status Codes

HTTP Status Code    | Data Type   
------------------- | ----------- 
200	| Delete succeeded
400	| Unexpected or non-identifiable arguments are supplied
401	| Unauthorized due to invalid session cookie
403	| Forbidden due to the user missing the necessary privileges
404	| Account not found with the supplied ID

<!--===================================================================-->
## Get List of Accounts

> Request

```shell
curl --cookie "videobank_sessionid=[VIDEOBANK_SESSIONID]" --request GET https://login.eagleeyenetworks.com/g/account/list
```

> Json Response

```json
[
    [
        "00004206",
        "Greater Good",
        2,
        2,
        2,
        0,
        0,
        1,
        null,
        1,
        1,
        1,
        0,
        0,
        1,
        "20141022214539.000",
        14.0
    ],
    [...],
    [...]
]
```

Returns array of arrays, with each sub-array representing an account available to the user. Please note that the ListAccount model definition below has property keys, but that's only for reference purposes since it's actually just a standard array.

### HTTP Request

`GET https://login.eagleeyenetworks.com/g/account/list`

### Account Array Attributes

Array Index		| Attribute   			| Data Type  	| Description
---------------	| --------------------- | -----------  	| -----------
0               | id          			| string       	| Unique identifier for the Account
1 				| name 					| string 		| Name of the account
2 				| camera_online_count	| int 			| Number of cameras currently online in the account
3				| camera_count 			| int 			| Number of cameras currently in the account
4 				| user_count 			| int 			| Number of users currently in the account
5 				| is_suspended 			| int 			| Indicates the account is Suspended (1) or not (0) 
6 				| is_inactive 			| int 			| Indicates the account is Inactive (1) or not (0)
7 				| is_active 			| int 			| Indicates the account is Active (1) or not (0)
8 				| product_edition 		| string 		| Product edition in use by the account

TODO Update Account Array Attributes

### Error Status Codes

HTTP Status Code    | Data Type   
------------------- | ----------- 
200	| Request succeeded
400	| Unexpected or non-identifiable arguments are supplied
401	| Unauthorized due to invalid session cookie
403	| Forbidden due to the user missing the necessary privileges
