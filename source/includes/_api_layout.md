# Layout

<!--===================================================================-->
## Overview

Layouts contain panes, which is a group of cameras arranged for viewing on screen. Layouts are associated with an account and account users are granted view/write/share permissions for the layout. Users who would otherwise have no access to a cameras, gain access to all cameras included in layouts shared with them.

The ordering of the panes is determined by the order of the array of LayoutJsonPane returned by the API. Each pane will have a size of 1, 2, or 3. A size of 1 is the smallest, and fills up 1x1 on the layout grid. A size of 3 is the largest and fills up 3x3 on the layout grid. If the grid does not have enough columns to fit the pane, then the size of the pane is decreased until it is able to fit on the grid.

Visual Examples:
<img src="images/api_layout/example_1.png" alt="Example 1" width="1000">
<img src="images/api_layout/example_2.png" alt="Example 2" width="400">

<!--===================================================================-->
## Layout Model

> Layout Model

```json
{
    "id": "0b58ec7a-61e4-11e3-8f7d-523445989f37",
    "name": "Everything",
    "types": [
        "mobile"
    ],
    "permissions": "SWRD",
    "current_recording_key": null,
    "shares": [
        [
            "ca01ce6d",
            "SWRD"
        ],
        [
            "ca0c7d2c",
            "R"
        ],
        [...],
        [...],
        [...],
        [
            "ca05e8c2",
            "R"
        ]
    ],
    "configuration": {
        "panes": [
            {
                "cameras": [
                    "100ace90"
                ],
                "type": "preview",
                "pane_id": 0,
                "name": "",
                "size": 1
            },
            {
                "cameras": [
                    "10045dd6"
                ],
                "type": "preview",
                "pane_id": 0,
                "name": "",
                "size": 1
            },
            {...},
            {...},
            {...},
            {
                "cameras": [
                    "100891b7"
                ],
                "type": "preview",
                "pane_id": 0,
                "name": "",
                "size": 1
            }
        ],
        "settings": {
            "camera_row_limit": 3,
            "camera_name": true,
            "camera_aspect_ratio": 0.5625,
            "camera_border": false
        }
    }
}
```

### Layout Attributes

Parameter               | Data Type             | Description
---------               | -----------           | -----------
id                      | string                | Unique identifier for the Layout
name                    | string                | Name of the layout
types                   | array[string]         | Specifies target(s) for layout. Multiple values are allowed.
permissions             | string                | String of zero or more characters. Each character defines a permission. Permissions include: 'R' - user can view this layout. 'W' - user can modify this layout. 'D' - user can delete this layout. 'S' - user can share this layout
current_recording_key   | string          | String key representing a recording currently being made with the cameras in the layout, which was initiated using the action/recordnow service.
shares                  | array[array[string]]) | Array of arrays, one per account user for whom sharing is enabled for this layout. Each string contains two field separated by comma. The first field is a user aid and the second field are permissions for the user. Two special user ID exist: ‘account’ specifies that the layout is shared with all users of the account. Second field contains permissions for users in the account. Example: [‘cafedead’,’RWDS’] = user can view, change, delete or share this layout. [‘cafe0001’,’RW’] = user can view this layout and change this layout. [‘account’, ‘R’] = All users of the account can view this layout. Permissions for the user issuing the /layout GET are not included in this array.
configuration           | LayoutConfiguration   | JSON object that defines the layout configuration

### LayoutConfiguration Attributes

Parameter               | Data Type                     | Description
---------               | -----------                   | -----------
panes                   | array[LayoutConfigurationPane]| Array of Panes
settings                | LayoutConfigurationSettings   | Settings object

### LayoutConfigurationPane Attributes

Parameter               | Data Type                     | Description
---------               | -----------                   | -----------
name                    | string                        | Layout pane name
type                    | string                        | ‘preview’ - shows live preview images form cameras. ‘carousel’- rotates between preview images, ids of cameras needs to be include in the cameras array along with an integer in the delay array. The delay is an integer value of milliseconds as too how long the Camera will be displayed before switching to the next Camera. A “carousel” with only one camera is the same as preview. ‘click’ - respond to click for other cameras in layout. ‘motion’ - respond to motion for other cameras in layout. ‘map’ - a static map with camera icons located on it. ‘url’ - displays the contents of the url in the pane as a frame.
pane_id                 | int                           | ID given to pane when created from Layout Manager
size                    | int                           | ['1' or '2' or '3']: Size to display image: 1 = small, 2 = medium, 3 = large
cameras                 | array[string]                 | )Array of camera ids. For ‘carousel’, cycle through the camera ids with the delay setting in the corresponding ‘delay’ property

### LayoutConfigurationSettings Attributes

Parameter               | Data Type         | Description
---------               | -----------       | -----------
camera_border           | boolean           | Show camera pane borders
camera_name             | boolean           | Show camera name
camera_aspect_ratio     | float             | ['0.5625' or '0.75']: Aspect ratio of images. .5625 = 16x9, .75 = 4x3
camera_row_limit        | int               | ['3' or '4' or '5']: Max number of cameras to show per row

<!--===================================================================-->
## Get Layout

> Request

```shell
curl -G https://login.eagleeyenetworks.com/g/layout -d "A=[VIDEOBANK_SESSIONID]&id=[LAYOUT_ID]"
```

Returns layout object by Id

### HTTP Request

`GET https://login.eagleeyenetworks.com/g/layout`

Parameter     | Data Type   | Description
---------     | ----------- | -----------
**id**        | string      | Layout Id

### Error Status Codes

HTTP Status Code    | Data Type   
------------------- | ----------- 
200 | Request succeeded
400 | Unexpected or non-identifiable arguments are supplied
401 | Unauthorized due to invalid session cookie
403 | Forbidden due to the user missing the necessary privileges

<!--===================================================================-->
## Create Layout

> Request

```shell
curl --cookie "videobank_sessionid=[VIDEOBANK_SESSIONID]" -X PUT -v -H "content-type: application/json" https://login.eagleeyenetworks.com/g/layout -d '{"name": "[NAME]", "json":"{\"panes\":[ {} ] }", "types":[""]}'
```

> Json Response

```json
{
    "id": "80ca9ee0-4f28-11e4-81bf-523445989f37"
}
```

Creates a new layout

### HTTP Request

`PUT https://login.eagleeyenetworks.com/g/layout`

Parameter     | Data Type     | Description | Is Required
---------     | -----------   | ----------- | -----------
**name**      | string        | Layout Name | true
**types**     | array[string] | Specifies target(s) for layout. Multiple values are allowed. | true
configuration | json          | JSON object that defines the layout configuration
shares        | ???           | Array of arrays, one per account user for whom sharing is enabled for this layout. Each string contains two fields separated by a comma. The first field is a user id and the second field is the list of permissions for the user. Two special user IDs exist: 'account' specifies that the layout is shared with all users of the account. The second field contains permissions for users in the account. Example: ['cafedead', 'RWDS'] = user can view, change, delete or share this layout. ['cafe0001', 'RW'] = user can view and change this layout. ['account', 'R'] = All users in the account can view the layout. Permissions for the user issuing the /layout GET are not included in this array.

### Error Status Codes

HTTP Status Code    | Data Type   
------------------- | ----------- 
200 | Request succeeded
400 | Unexpected or non-identifiable arguments are supplied
401 | Unauthorized due to invalid session cookie
403 | Forbidden due to the user missing the necessary privileges

<!--===================================================================-->
## Update Layout

> Request TODO

```shell
```

> Json Response TODO

```json
```

### Error Status Codes

HTTP Status Code    | Data Type   
------------------- | ----------- 
200 | Request succeeded
400 | Unexpected or non-identifiable arguments are supplied
401 | Unauthorized due to invalid session cookie
403 | Forbidden due to the user missing the necessary privileges

<!--===================================================================-->
## Delete Layout

> Request

```shell
curl --cookie "videobank_sessionid=[VIDEOBANK_SESSIONID]" -X DELETE -v -H "content-type: application/json" https://login.eagleeyenetworks.com/g/layout -d "id=[LAYOUT_ID]" -G
```

### HTTP Request

`DELETE https://login.eagleeyenetworks.com/g/layout`

Parameter     | Data Type   | Description
---------     | ----------- | -----------
**id**        | string      | Layout Id

### Error Status Codes

HTTP Status Code    | Data Type   
------------------- | ----------- 
200 | Request succeeded
400 | Unexpected or non-identifiable arguments are supplied
401 | Unauthorized due to invalid session cookie
403 | Forbidden due to the user missing the necessary privileges
404 | Layout matching the ID was not found

<!--===================================================================-->
## Get List of Layouts

> Request

```shell
curl --cookie "videobank_sessionid=[VIDEOBANK_SESSIONID]" --request GET https://login.eagleeyenetworks.com/g/layout/list
```

> Json Response

```json
[
    [
        "0b58ec7a-61e4-11e3-8f7d-523445989f37",
        "Everything",
        [
            "mobile"
        ],
        "SWRD"
    ],
    [
        "75c03026-719a-11e3-afba-ca8312ea370c",
        "Glenn",
        [
            "mobile"
        ],
        "SWRD"
    ],
    [...],
    [...],
    [...]
]
```

Returns array of arrays, with each sub-array representing a layout available to the user. Please note that the ListLayout model definition below has property keys, but that's only for reference purposes since it's actually just a standard array.

### HTTP Request

`GET https://login.eagleeyenetworks.com/g/layout/list`

### Response: Layout Model

Array Index     | Attribute   | Data Type       | Description
---------       | ----------- | -----------     | -----------
0               | id          | string          | Unique identifier for the Layout
1               | name        | string          | Name of the layout
2               | types       | array[string]   | Array of types defined for the layout. This attribute has not yet been implemented.
3               | permissions | string          | String of zero or more characters. Each character defines a permission that the current user has for the layout. Permissions include: 'R' - user can view this layout. 'W' - user can modify this layout. 'D' - user can delete this layout. 'S' - user can share this layout.

### Error Status Codes

HTTP Status Code    | Data Type   
------------------- | ----------- 
200 | Request succeeded
400 | Unexpected or non-identifiable arguments are supplied
401 | Unauthorized due to invalid session cookie
403 | Forbidden due to the user missing the necessary privileges