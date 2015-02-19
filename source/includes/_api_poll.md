# Poll

<!--===================================================================-->
## Overview

The poll service provides a mechanism for an application to receive notifications of events or spans from the Eagle Eye service. These entities are grouped by resource. There are six main resources:

  * thumb - Thumbnail resource. Provides a timestamp for a thumbnail image. One can use the timestamp to grab the actual thumbnail image.
  * pre - Preview resource. Provides a timestamp for a preview image. One can use the timestamp to grab the actual preview image.
  * video - Video resource. Provides a start and end timestamp of a video event.
  * event - Event resource. Provides full event information.
  * [status](#status-bitmask) - A bitmask flag defining the state of a bridge or a camera.

Events can come from lots of sources:

  * Devices or Cameras (camera alerts, start and stop recording, etc.)
  * System Events (maintenance, server changes)
  * Account Events (other user changes, account alerts, layout changes).

Device and Camera events include, on, off, online, offline, currently recording, currently sensing motion, start/stop schedule event, being controlled with PTZ)

This service will continually be extended. Any UI should not care about stuff that it does not understand.

Poll is a stateful request for updates any time a matching event occurs within the service. The initial poll request is a POST with a JSON formatted body indicating the resources to track. Resources that are video, pre, and thumbnail automatically register the api caller to their respective events. However, resource type ‘event’ requires the api caller to tell the API what events to listen for.

Each object consists an id element and a list of resource types to be monitored. The POST transaction receives an immediately responds with a JSON formatted body indicating the current timestamp for all requested resources. The response also includes a cookie, which can be used to track changes to the indicated resources via GET transaction.

Each resource type has a specific object format in response:

Type        | Response          | Notes
---------   | -----------       | -----------     
video       | [startts, endts]  | List of start, end Timestamps for video segment. Updates at start and per key frame received until end.
thumb       | thumbts           | Timestamp of latest thumbnail image
pre         | prets             | Timestamp of latest preview image
[status](#status-bitmask) | bitmask           | A numerical bitmask defining the status. Bit position defines status. The meaning of each bit is defined in the table below.
event       | object            | Events are a key value pair, where the key is the four CC of the event, and event structure are the actual meta data for that specific event. Available events are shown in the table below.

## Status Bitmask

HEX Value | Status
--------- | ----------
0x000001  | Camera Online **(DEPRECATED)**
0x000002  | Stream Attached (Camera Communicating With Bridge) **(DEPRECATED)**
0x000004  | Camera On (User Setting) **(DEPRECATED)**
0x000008  | Camera Recording **(DEPRECATED)**
0x000010  | Camera Sending Previews
0x000020  | Camera Located (Bridge has found the camera)
0x000040  | Camera Not Supported
0x000080  | Camera Configuration in Process (Bridge Configuring Camera)
0x000100  | Camera Needs ONVIF Password
0x000200  | Camera Available But Not Yet Attached
0x000400  | *Internal Status*
0x000800  | Camera Error
0x001000  | *Internal Status*
0x002000  | *Internal Status*
0x004000  | *Reserved*
0x008000  | *Reserved*
0x010000  | Invalid State (Unknown State)
0x020000  | Camera On (User Setting)
0x040000  | Camera Streaming Video
0x080000  | Camera Recording
0x100000  | Camera Online

This status bit mask is used to determine what the high-level/overall camera status is, using the following logic:

IF "Camera On" (bit 17)==0 THEN "Off" (orange forbidden icon)
<br>ELSE IF "Registered" (bit 20)==0 THEN "Internet Offline" (red exclamation icon)
<br>ELSE IF "Streaming" (bit 18)==1 THEN "Online" (green check icon)
<br>ELSE IF "Password" (bit 8)==1 THEN "Password Needed" (effectively "Offline") (red padlock icon)
<br>ELSE "Offline" (red X icon)

Recording status uses the following logic:

IF "Recording" (bit 19) THEN Recording (green circle icon)

IF "Invalid" (bit 16)==1 THEN no status change (use whatever status bits were set previously)

## Event Objects

Four CC   | Description
--------- | -----------      
VRES      | Video start event
VREE      | Video end event
VRKF      | Video key frame event
VRSU      | Video update event
PRSS      | Preview stream start event
PRTH      | Thumbnail event
PRFR      | Preview event
PRSE      | Preview stream end event
PRSU      | Preview stream update event
EMES      | Motion start event
EMEE      | Motion end event
ESES      | Event stream start
EVVS      | Video swap event
ESEE      | Event stream stop
ECON      | The camera is online
ECOF      | The camera is offline
AELI      | Account event log in
AELO      | Account event log out
AEDO      | Download video event
AEUC      | Create user event
AEUD      | Delete user event
AECC      | User config change event
AELD      | Live display event
AEPT      | Pan tilt zoom event
AAEC      | Layout create event
AEED      | Layout delete event
AEEL      | Layout change event
AEAC      | Account create event
AEAD      | Account delete event
AEAH      | Account change event
AEDC      | Device create event
AEDD      | Device delete event
AEDH      | Device change event
CECF      | Camera found event
CSAT      | Camera stream attach event
CSDT      | Camera stream detach event
COFF      | Camera off event
CONN      | Camera on event
COBC      | Camera bounce event
CSTS      | Camera settings event
CSTC      | Camera settings change event
CPRG      | Camera purge event
CDLT      | Camera data lost event
CBWS      | Camera bandwidth sample event
BBWS      | Bridge bandwidth sample event
AEDA      | Device alert event
AEDN      | Device alert notification event
MRBX      | Motion Box event
MRSZ      | Motion size reports
ROMS      | Region of interest motion start
ROME      | Region of interest motion end
ALMS      | Alert Motion Start
ALME      | Alert Motion End
ALRS      | Alert Region Of Interest Start
ALRE      | Alert Region Of Interest End

<!--===================================================================-->
## Initialize Poll

> Request Json

```json
{
    "cameras": {
        "100e1e23": {
            "resource": [
                "pre",
                "thumb",
                "status",
                "event"
            ],
            "event": [
            	"MRBX"
            ]
        },
        "10097d15": {
            "resource": [
                "pre",
                "thumb",
                "status",
                "event"
            ],
            "event": [
            	"MRBX"
            ]
        },
        "[camera_id]": {...},
        "[camera_id]": {...},
        "[camera_id]": {...}
    }
}
```

### HTTP Request

`POST https://login.eagleeyenetworks.com/poll`

Subscribe to poll service, which is required for GET /poll. 
Response headers: set_cookie: ee-poll-ses=xxxxxx

Parameter       | Data Type   | Description
---------       | ----------- | -----------
cameras 		| [PostPollCameras](#postpollcameras-attributes) | Cameras

### PostPollCameras Attributes

This json attribute contains as many PostPollCamera json objects as the user desires. The key for each PostPollCamera is the camera_id.

Parameter       | Data Type   | Description
---------       | ----------- | -----------
[camera_id 1] 	| [PostPollCamera](#postpollcamera-attributes) | Camera 1
[camera_id 2] 	| [PostPollCamera](#postpollcamera-attributes) | Camera 2
... | ... | ...
[camera_id N]   | [PostPollCamera](#postpollcamera-attributes) | Camera N

### PostPollCamera Attributes

Parameter       | Data Type   | Description
---------       | ----------- | -----------
resource 		| array[string, enum] | enum: pre, thumb, status, event
event 			| array[string, enum] | enum: [event objects](#event-objects)

> Json Response

```json
{
    "cameras": {
        "100e1e23": {
            "status": 1441847
        },
        "10097d15": {
            "status": 1441847
        },
        "[camera_id]": {...},
        "[camera_id]": {...},
        "[camera_id]": {...}
    },
    "token": "ooe0eoEAMNsF"
}
```

### Response Json Attributes

Parameter       | Data Type   	| Description
---------       | ----------- 	| -----------
cameras         | [PostPollResponseCameras](#postpollresponsecameras-json-attributes) | Objects keyed by camera id
token 			| string 		| Token to be used for subsequent GET /poll requests

### PostPollResponseCameras Json Attributes

Parameter       | Data Type   | Description
---------       | ----------- | -----------
[camera_id 1] 	| [PostPollResponseCamera](#postpollresponsecamera-json-attributes) | Camera 1
[camera_id 2] 	| [PostPollResponseCamera](#postpollresponsecamera-json-attributes) | Camera 2
... | ... | ...
[camera_id N]   | [PostPollResponseCamera](#postpollresponsecamera-json-attributes) | Camera N

### PostPollResponseCamera Json Attributes

Parameter       | Data Type   	| Description
---------       | ----------- 	| -----------
status          | string 		| A bitmask flag defining the state of a bridge or a camera. [More Info](#status-bitmask)
event 			| [PostPollResponseCameraEvents](#postpollresponsecameraevents-json-attributes) |  Object of events keyed by event id

### PostPollResponseCameraEvents Json Attributes

Parameter       | Data Type   | Description
---------       | ----------- | -----------
[event_id 1] 	| [PostPollResponseCameraEvent](#postpollresponsecameraevent-json-attributes) | Event 1
[event_id 2] 	| [PostPollResponseCameraEvent](#postpollresponsecameraevent-json-attributes) | Event 2
... | ... | ...
[event_id N] 	| [PostPollResponseCameraEvent](#postpollresponsecameraevent-json-attributes) | Event N

### PostPollResponseCameraEvent Json Attributes

Parameter       | Data Type   	| Description
---------       | ------------- | -----------
timestamp 		| string 		| Timestamp in EEN format: YYYYMMDDHHMMSS.NNN
cameraid 		| string 		| internal unique identifier

<!--===================================================================-->
## Polling

Used to receive updates on real-time changes. This API call requires a valid 'ee-poll-ses' cookie from POST /poll.

### HTTP Request

`GET https://login.eagleeyenetworks.com/poll`

> Json Response

```json
{
    "cameras": {
        "100e1e23": {
            "pre": "20141121165011.233",
            "event": {
                "MRBX": {
                    "timestamp": "20141121165011.499",
                    "cameraid": "100c299e",
                    "boxes": [
                        {
                            "x": 24575,
                            "y": 29126,
                            "w": 4095,
                            "h": 5825
                        }
                    ]
                },
                "PRFU": {
                    "timestamp": "20141121165011.233",
                    "cameraid": "100c299e",
                    "file_offset": 26311872,
                    "frame_size": 7838
                },
                "PRFR": {
                    "timestamp": "20141121165011.233",
                    "cameraid": "100c299e",
                    "previewid": 1416585600,
                    "file_offset": 26311872,
                    "frame_size": 7830
                }
            }
        },
        "10097d15": {
            "pre": "20141121165011.281",
            "event": {
                "PRFU": {
                    "timestamp": "20141121165011.281",
                    "cameraid": "1002a2a4",
                    "file_offset": 6126297,
                    "frame_size": 4014
                },
                "PRFR": {
                    "timestamp": "20141121165011.281",
                    "cameraid": "1002a2a4",
                    "previewid": 1416585600,
                    "file_offset": 6126297,
                    "frame_size": 4006
                }
            }
        }
    }
}
```

### Response Json Attributes

Parameter       | Data Type   	| Description
---------       | ----------- 	| -----------
cameras         | [GetPollResponseCameras](#getpollresponsecameras-json-attributes) | Objects keyed by camera id

### GetPollResponseCameras Json Attributes

Parameter       | Data Type   | Description
---------       | ----------- | -----------
[camera_id 1] 	| [GetPollResponseCamera](#getpollresponsecamera-json-attributes) | Camera 1
[camera_id 2] 	| [GetPollResponseCamera](#getpollresponsecamera-json-attributes) | Camera 2
... | ... | ...
[camera_id N]   | [GetPollResponseCamera](#getpollresponsecamera-json-attributes) | Camera N

### GetPollResponseCamera Json Attributes

Parameter       | Data Type   	| Description
---------       | ----------- 	| -----------
pre          	| string 		| Timestamp in EEN format: YYYYMMDDHHMMSS.NNN
event 			| [GetPollResponseCameraEvents](#getpollresponsecameraevents-json-attributes) |  Object of events keyed by event id

### GetPollResponseCameraEvents Json Attributes

Parameter       | Data Type   | Description
---------       | ----------- | -----------
[event_id 1] 	| [GetPollResponseCameraEvent](#getpollresponsecameraevent-json-attributes) | Event 1
[event_id 2] 	| [GetPollResponseCameraEvent](#getpollresponsecameraevent-json-attributes) | Event 2
... | ... | ...
[event_id N] 	| [GetPollResponseCameraEvent](#getpollresponsecameraevent-json-attributes) | Event N

### GetPollResponseCameraEvent Json Attributes

Parameter       | Data Type   	| Description
---------       | ------------- | -----------
timestamp 		| string 		| Timestamp in EEN format: YYYYMMDDHHMMSS.NNN
cameraid 		| string 		| internal unique identifier

### Error Status Codes

HTTP Status Code    | Data Type   
------------------- | ----------- 
200 | Request succeeded
400	| Unexpected or non-identifiable arguments are supplied
401	| Unauthorized due to invalid session cookie
403	| Forbidden due to the user missing the necessary privileges


