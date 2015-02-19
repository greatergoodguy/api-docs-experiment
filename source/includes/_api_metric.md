# Metric

<!--===================================================================-->
## Overview

This service defines metrics that can be queried from the system.

<!--===================================================================-->
## Camera Bandwidth

> Request

```shell
curl -G https://login.eagleeyenetworks.com/g/metric/camerabandwidth -d "A=[VIDEOBANK_SESSIONID]&id=[CAMERA_ID]"
```

Used to query the bandwidth usage for a particular camera device.

### HTTP Request

`GET https://login.eagleeyenetworks.com/g/metric/camerabandwidth`

Parameter       | Data Type   	| Description  	| Is Required
---------       | ----------- 	| -----------  	| ----------- 
**id**   		| string      	| Bridge Id 	| true
start_timestamp | string      	| Start timestamp of query, in EEN format: YYYYMMDDHHMMSS.NNN. Defaults to 7 days ago.
end_timestamp  	| string   		| End timestamp of query, in EEN format: YYYYMMDDHHMMSS.NNN. Defaults to now.
group_by 		| string, enum  | Hour or Day, indicating how the results should be grouped. <br><br>enum: day, hour, minute
motion_interval | int      		| Motion Interval used for Motion Activity metric, in milliseconds. Defaults to 15000.
metric          | string, enum  | String delimited list used to filter which metrics gets returned. Setting this parameter to 'core,motion' will return data only for core and motion. <br><br>enum: core, packets, motion

> Json Response

```json
{
    "core": [
        [
            "20141002190000.000",
            0.0,
            0.0,
            215910545.0,
            76733036.0,
            32049659.0,
            52716510.0
        ],
        [
            "20141002200000.000",
            0.0,
            0.0,
            252051927.0,
            164214711.0,
            36128066.0,
            223484133.0
        ],
        [...],
        [...],
        [...],
        [
            "20141009190000.000",
            0.0,
            0.0,
            41425890.0,
            10373660.0,
            5029677.0,
            78599812.0
        ]
    ],
    "packets": [
        [
            "20141002190000.000",
            0.00183
        ],
        [
            "20141002200000.000",
            0.0018439999999999999
        ],
        [...],
        [...],
        [...],
        [
            "20141009190000.000",
            0.0
        ]
    ],
    "motion": []
}
```

### Response Json Attributes

Parameter       | Data Type                     | Description   
---------       | -----------                   | -----------  
core            | array[[CameraCore](#cameracore-json-array-elements)]     | Array of core metrics
packets         | array[[CameraPackets](#camerapackets-json-array-elements)]  | Array of packet metrics
motion          | array[[CameraMotion](#cameramotion-json-array-elements)] | Array of motion metrics

### CameraCore Json Array Elements

Index       | Data Type     | Description
---------   | -----------   | -----------  
0           | string        | EEN Timestamp: YYYYMMDDHHMMSS.NNN
1           | float         | Average Kilobytes on Disk
2           | float         | Average Days on Disk
3           | float         | Bytes Stored
4           | float         | Bytes Shaped
5           | float         | Bytes Streamed
6           | float         | Bytes Freed

### CameraPackets Json Array Elements

Index       | Data Type     | Description
---------   | -----------   | -----------  
0           | string        | EEN Timestamp: YYYYMMDDHHMMSS.NNN
1           | float         | Packet loss percentage (decimal)

### CameraMotion Json Array Elements

Index       | Data Type     | Description
---------   | -----------   | -----------  
0           | string        | EEN Timestamp: YYYYMMDDHHMMSS.NNN
1           | int           | motion activity value

### Error Status Codes

HTTP Status Code    | Data Type   
------------------- | ----------- 
200 | Request succeeded
400 | Unexpected or non-identifiable arguments are supplied
401 | Unauthorized due to invalid session cookie
403 | Forbidden due to the user missing the necessary privileges
404 | Metrics not found

<!--===================================================================-->
## Bridge Bandwidth

> Request

```shell
curl -G https://login.eagleeyenetworks.com/g/metric/bridgebandwidth -d "A=[VIDEOBANK_SESSIONID]&id=[BRIDGE_ID]"
```

Used to query the bandwidth usage for a particular bridge device.

### HTTP Request

`GET https://login.eagleeyenetworks.com/g/metric/bridgebandwidth`

Parameter       | Data Type   	| Description  	| Is Required
---------       | ----------- 	| -----------  	| ----------- 
**id**   		| string      	| Bridge Id 	| true
start_timestamp | string      	| Start timestamp of query, in EEN format: YYYYMMDDHHMMSS.NNN. Defaults to 7 days ago.
end_timestamp  	| string   		| End timestamp of query, in EEN format: YYYYMMDDHHMMSS.NNN. Defaults to now.
group_by 		| string, enum  | Hour or Day, indicating how the results should be grouped. <br><br>enum: day, hour, minute

> Json Response

```json
{
    "core": [
        [
            "20141002170000.000",
            711610368.0,
            673860608.0,
            21533380.0,
            10299.0,
            10064282.0,
            9903.0
        ],
        [
            "20141002180000.000",
            711610368.0,
            673802922.66666698,
            139693604.0,
            16579176.0,
            30849223.0,
            70446106.0
        ],
        [...],
        [...],
        [...],
        [
            "20141009170000.000",
            711610368.0,
            674052608.0,
            20663486.0,
            8637204.0,
            18879693.0,
            19383808.0
        ]
    ],
    "bandwidth": [
        [
            "20141002180000.000",
            253117.37089200001
        ],
        [
            "20141002220000.000",
            240255.52353499999
        ],
        [...],
        [...],
        [...],
        [
            "20141009150000.000",
            232692.09302299999
        ]
    ],
    "storage": [
        [
            "20141002170000.000",
            21523477
        ],
        [
            "20141002180000.000",
            69247498
        ],
        [...],
        [...],
        [...],
        [
            "20141009170000.000",
            1279678
        ]
    ]
}
```

### Response Json Attributes

Parameter       | Data Type         | Description   
---------       | -----------       | -----------  
core            | array[[BridgeCore](#bridgecore-json-array-elements)]       | Array of core metrics
bandwith        | array[[BridgeBandwidth](#bridgecore-json-array-elements)]  | Array of bandwidth metrics
storage         | array[BridgeStorage](#bridgestorage-json-array-elements)]    | Array of storage metrics

### BridgeCore Json Array Elements

Index       | Data Type     | Description
---------   | -----------   | -----------  
0           | string        | EEN Timestamp: YYYYMMDDHHMMSS.NNN
1           | float         | Average Kilobytes on Disk
2           | float         | Average Days on Disk
3           | float         | Bytes Stored
4           | float         | Bytes Shaped
5           | float         | Bytes Streamed
6           | float         | Bytes Freed

### BridgeBandwidth Json Array Elements

Index       | Data Type     | Description
---------   | -----------   | -----------  
0           | string        | EEN Timestamp: YYYYMMDDHHMMSS.NNN
1           | float         | Bytes per second

### BandwidthStorage Json Array Elements

Index       | Data Type     | Description
---------   | -----------   | -----------  
0           | string        | EEN Timestamp: YYYYMMDDHHMMSS.NNN
1           | float         | Bytes Diff

### Error Status Codes

HTTP Status Code    | Data Type   
------------------- | ----------- 
200 | Request succeeded
400 | Unexpected or non-identifiable arguments are supplied
401 | Unauthorized due to invalid session cookie
403 | Forbidden due to the user missing the necessary privileges
404 | Metrics not found