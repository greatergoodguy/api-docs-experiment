# Images and Video

<!--===================================================================-->
## Overview

Asset services provide access to media assets - previews and video in appropriate formats. Asset services are used in conjunction with list transactions to enumerate and identify assets.

Assets are identified by a tuple of timestamp, cameraid, quality, and format.

  * Timestamp: Eagle Eye timestamps have the format YYYYMMDDhhmmss.xxx and are always specifined in GMT time. In most contexts, special tokens can also be used to specify relative times - “now” is the current time, a value starting with + or - is an offset from the current time.
  * CameraID: Cameras are identified by a 8 character hexadecimal string, representing a unique 32 bit id associated with a specific camera. Note CameraID are not necessarily linked to specific hardware devices to allow device upgrade and replacement without disruption of history storage.
  * Quality: (low,med,high) Images and video may have multiple quality levels, each representing the same base asset. Video can be transcoded between quality levels on demand (at some point) to support reduced bandwidth for mobile devices. Normally cameras will capture at medium or high quality. For video, low quality is targeted at around 100kbps, medium quality is under 500kbps, and high quality is around 1mbps. Additional quality levels will be supported in time.
  * Format: Images are always returned as JPEG images. Video can currently returned as either FLV format (playback in browsers via Flash), MP4 (download and export format), and m3u/mpegts (HttpStreaming for iOS and newer android devices).

### Request Image

The image request model provides an efficient mechanism for accessing image sequences for several usage models. Image requests can be done directly using the next/after/prev virtual model. This returns images before or after specified timestamps. Alternatively, the timestamp and event information can be fetched through the “list” interface (to get events for history) and “poll” interface to track new images as they become available in real-time. The following cookbooks provides typical usage models for various events.

  * low bandwidth video playback: The preview stream is a sequential set of jpeg images. If played back in order, low resolution video is accomplished.
    * The simplest implementatation is to fetch “next” with a timestamp of “now” (e. g. /asset/next/image.jpeg?t=now;c=12345678;a=pre) - waiting for the subsequent image after the current time. Each time an image is returned, a new request should be made. If the downstream bandwidth is very low, the image fetch will automatically slow down (because delivery of image A happens after image B has been received, so the next call will fetch image C, skipping display of image B entirely). This approach works well for tracking a single image stream. As a tip, the first request should be done as a “prev” request to make sure an image is displayed, before the sequential next requests. The downside of this model is it requires a dedicated socket for each image stream being played. Many browsers have a limited pool of open sockets.
    * A more efficient mechanism for tracking multiple image streams is to use the “poll” interface. The Poll interface will provide the timestamp of the next image available for a set of camera, which can then be fetched via the /asset/asset call. Since the poll request supports multiple cameras in a single request, it requires only a single socket for any number of cameras. The client application should implement a “fair” algorithm across the returned timestamps to address low bandwidth situations (that is, make sure every image stream gets updated before you fetch a new image for the same stream). This algorithm will provide smooth frame rate degradation across any number of cameras, whether the performance bottleneck is client CPU or bandwidth. The best model for this is
      * receive update notifications for all cameras being tracked via a single sequential poll session
      * for each camera, keep track of the “lastest” image notification, replacing the last one even if it has not been fetched yet.
      * with a limited pool of “requests”, do a fair rotation between all cameras, fetching only the most recent image for each, and skipping the fetch if the image is already loading.
  * Random access image discovery. The preview and thumb image streams can provide a visual navigation tool for accessing recorded video. The typical implementation requires a map from a timestamp to the “best” image for that timestamp. To implement this approach, the client should use the “after” and “prev” requests with the timestamp of the user playhead. Both calls provide header data for x-ee-timestamp, x-ee-next, and x-ee-prev which identify the current and subsequent images in both directions when it can be easily determined. The usage paradigm for this should be
    * On navigation event (large jump), determine the timestamp of the user playhead and do an “/asset/prev” call to get the appropriate image (hmm, this should really be a best, figure out how ugly that would be). Store the x-ee-timestamp, x-ee-next and x-ee-prev values for the image.
    * As the user moves the playhead, if the time change is within the next/prev halfway bounds, no new request is required. when the use moves outside of the time range, do an image fetch with the new timestamp.
  * Thumbnail navigation. The system provides a “thumbnail” image for each event which is intended to provide a small representation of the event. The easiest mechanism to get a thumbnail for an event is to do an /asset/after/image.jpeg?a=thumb... image request with the starting timestamp of the event.

### Retrieve List of Images

There are a couple different ways to get a list of images. There first is to get all preview images between April 1st and April 2nd:
`https://login.eagleeyenetworks.com/asset/list/image.jpeg?c=100676b2;t=20140401000000.000;e=20140402000000.000;a=all;`

The second way is to get 500 images before April 1st: 
`https://login.eagleeyenetworks.com/asset/list/image.jpeg?c=100676b2;t=20140401000000.000;count=-500;a=all;`

The third way is to get the next 500 images after April 1st: 
`https://login.eagleeyenetworks.com/asset/list/image.jpeg?c=100676b2;t=20140401000000.000;count=500;a=all;`

### Retrieve Video

Video is accessed via the “play” command. Video is captured in segments, and is limited to 5 minutes per segment in storage. The video command will seamlessly rewrite headers within the video format to join segments as necessary to deliver the requested data span.

If the end time of the segment is in the future, the video will follow the data stream as it arrives, delivering live video streaming with minimal latency. MP4 format cannot be live streamed. Note: if the camera is not streaming video, the video will stop (and start again) as video is captured, which is typically not what is desired.

The key word “stream_<streamid>” can be used for the starting timestamp. This forces the camera to capture video and stream it to the cloud live. The stream id should be globally unique(ish) string - combination of a timestamp and userid works well. It is only critical for M3U requests, where it assure continuity between the m3u poll transactions.

The start timestamp must match the starting timestamp of a video if the video already exists. Subsegments of a video span can be specified by using the “to” (time offset) argument. For example, assume a 5 minute video has been recorded from 12:30 to 12:35. The query "?t=20121120123000.000;e=20121120123400.000;to=180000;..." will play one minute of video (timestamped at 12:33), 3 minutes into the video starting at 12:30, clipping off the last minute of the recorded segment.

Time offsets and end timestamps can be at any time, but the system will seek to the nearest key frame to actually start the video.

### Video Formats

The video system is based on h264 video and AAC audio. These streams are encapsulated in different formats for compatibility with different playback modes

  * FLV: the native format for the system. Playable in any Flash player and by VLC etc.
  * Live HTTP Streaming m3u: M3U files are index files into a mpegts data stream. The system will generate ts urls on approximately a 2 second basis (depending on key frame rate of the underlying video). Note, due to the polling nature of m3u for “live” streams, you can only use now relative requests for streaming (where the streamid is used to maintain transaction state). So "/asset/play/video.m3u?t=stream_34567890332244567;e=+300000;c=12345678" will create a five minute stream, but "/asset/play/video.m3u?t=-50000;e=+300000" will not.
  * ts: MPEG Transport Stream format video and audio. Intended for playback via http streaming in concert with m3u transactions, per the HTTP Live Streaming functionality of iOS and android. You can list multiple streams for a single video (typically for different resolutions/bandwidth).
  * mp4: MPEG4 files have very broad playback compatibility - all major video player are compatible. However, mp4 is NOT a streamable format, so it is only used for download functionality and will return an error if the video is live.
  * m3u8: Use the M3U8 play list format. Use this for mobile devices as it uses the HTTP layer to stream MPEG TS files with instructions in the M3U8 playlist file. Continue polling for this playlist until the playlist indicates it is complete.

### Video Quality

The H264 codec has the concept of profiles and levels to convey whether a playback devices is compatible with a specific video stream.

  * Low: Low quality video has a maximum profile of baseline (640x480 max)
  * Med: medium quality video has a maximum profile of main
  * High: high quality video has a maximum profile of high

<!--===================================================================-->
## EEN Timestamp

All Assets have an EEN timestamp attached. Timestamps are always in UTC and maintained to the nearest millisecond. Timestamps are rendered in text as YYYYMMDDhhmmss.mmm.

+/- offsets from “now” are valid in ms.

<!--===================================================================-->
## Get Image

> Request

```shell
curl -v -G "https://login.eagleeyenetworks.com/asset/prev/image.jpeg?id=[CAMERA_ID];timestamp=[TIMESTAMP];quality=[QUALITY];asset_class=[ASSET_CLASS];A=[VIDEOBANK_SESSIONID]"
```

Get an image jpeg based on the specified timestamp. This will return binary image data in JPEG format. 

Headers: cache control headers to allow asset caching if no now relative:

  * x-ee-timestamp: type-timestamp specifies asset type and timestamp of the provided image. Type is one of 'video', 'preview', 'thumb', 'event'. 
  * x-ee-prev: [type-timestamp | unknown ] of previous image matching the class filter OR 'unknown' if the previous image was complex to figure out. 
  * x-ee-next: [type-timestamp | unknown ] as with x-ee-prev, but specifying the following image. 
content-type: image/jpeg 
  * location: /asset/asset/image.jpeg?t=20120917213405.700;q=low;c=thumb (identifies actual asset time of image in response).Returns user object by ID. Not passing an ID will return the current authorized user.

### HTTP Request

`GET https://login.eagleeyenetworks.com/asset/asset/image.jpeg`
<br> Get the image at the specified timestamp

`GET https://login.eagleeyenetworks.com/asset/prev/image.jpeg`
<br> Get the first image before the specified timestamp

`GET https://login.eagleeyenetworks.com/asset/after/image.jpeg`
<br> Get the first image after the specified timestamp

Parameter         | Data Type     | Description   | Is Required
---------         | -----------   | -----------   | -----------
**id**            | string        | Camera Id     | true
**timestamp**     | string        | Timestamp in EEN format: YYYYMMDDHHMMSS.NNN | true
**asset_class**   | string, enum  | Asset class of the image <br><br>enum: all, pre, thumb | true
quality           | string, enum  | Quality of image <br><br>enum: low, med, high

### Error Status Codes

HTTP Status Code    | Data Type   
------------------- | ----------- 
200 | Request succeeded
301 | Asset has been moved to a different archiver
400 | Unexpected or non-identifiable arguments are supplied
401 | Unauthorized due to invalid session cookie
403 | Forbidden due to the user missing the necessary privileges
404 | Image was not found

<!--===================================================================-->
## Get Video

> Request

```shell
curl -v -G "https://login.eagleeyenetworks.com/asset/play/video.flv?id=[CAMERA_ID];start_timestamp=[START_TIMESTAMP];end_timestamp=[END_TIMESTAMP];A=[VIDEOBANK_SESSIONID]"
```

Returns a video stream in the requested format. Formats include

  * flv
  * mp4
  * m3u8
  * webm

### HTTP Request

`GET https://login.eagleeyenetworks.com/asset/play/video.{video_format}`

Parameter                 | Data Type     | Description   | Is Required
---------                 | -----------   | -----------   | -----------
**id**                    | string        | Camera Id     | true
**start_timestamp**       | string        | Start Timestamp in EEN format: YYYYMMDDHHMMSS.NNN | true
**end_timestamp**         | string        | End Timestamp in EEN format: YYYYMMDDHHMMSS.NNN | true
quality                   | string, enum  | Indicates requested resolution if multiple are available. <br><br>enum: low, mid, high
playback_start_timestamp  | string        | EE timestamp where playback will begin. This value should be between the start_timestamp and end_timestamp.
time_offset               | string        | Start the video stream N ms into the specified timespan. Allows playing of a smaller segment.

### Error Status Codes

HTTP Status Code    | Data Type   
------------------- | ----------- 
200 | Request succeeded
301 | Asset has been moved to a different archiver
400 | Unexpected or non-identifiable arguments are supplied
401 | Unauthorized due to invalid session cookie
403 | Forbidden due to the user missing the necessary privileges
404 | Video was not found

<!--===================================================================-->
## Prefetch Image

> Request

```shell
curl -v -G "https://login.eagleeyenetworks.com/asset/cloud/image.jpg?start_timestamp=[START_TIMESTAMP];id=[CAMERA_ID];webhook_url=[WEBHOOK_URL]A=[VIDEOBANK_SESSIONID]"
```

> Webhook JSON POST Response

```json
{ "event:": "[EVENT]" }
```

This API call will ensure the image is in the cloud. If the image is not in the cloud it will do a background upload request to the bridge to aquire the image into the cloud. A webhook provided with the call will be triggered when the upload is successful or an error has occurred. The webhook will be triggered as a POST with JSON formatted data.

### HTTP Request
`GET https://login.eagleeyenetworks.com/asset/cloud/image.jpg`

Parameter           | Data Type     | Description   | Is Required
---------           | -----------   | -----------   | -----------
**id**              | string        | Camera Id     | true
**start_timestamp** | string        | Start Timestamp in EEN format: YYYYMMDDHHMMSS.NNN | true
**webhook_url**     | string        | The webhook url (must be urlencoded) to trigger | true

### JSON **EVENT** Values
Value                              | Description
---------                          | ---------
ASSET_CLOUD_EVENT_UPLOADED         | The image has been successfully uploaded into the cloud.
ASSET_CLOUD_EVENT_DEMAND_FAILED    | The image failed aquiring a connection to the bridge.
ASSET_CLOUD_EVENT_NOTHING_UPLOAD   | Nothing was uploaded since the image was already in the cloud.
ASSET_CLOUD_EVENT_INVALID_RANGE    | An invalid range (timestamp) was requested.
ASSET_CLOUD_EVENT_ABORT            | General error occurred.

### HTTP Status Codes
HTTP Status Code    | Data Type   
------------------- | ----------- 
201 | Request has been created and webhook will be triggered upon completion or error.

<!--===================================================================-->
## Prefetch Video

> Request

```shell
curl -v -G "https://login.eagleeyenetworks.com/asset/cloud/video.flv?start_timestamp=[START_TIMESTAMP];end_timestamp=[END_TIMESTAMP];id=[CAMERA_ID];webhook_url=[WEBHOOK_URL]A=[VIDEOBANK_SESSIONID]"
```

> Webhook JSON POST Response

```json
{ "event:": "[EVENT]" }
```

This API call will ensure the video is in the cloud. If the video is not in the cloud it will do a background upload request to the bridge to aquire the video into the cloud. A webhook provided with the call will be triggered when the upload is successful or an error has occurred. The webhook will be triggered as a POST with JSON formatted data.

### HTTP Request
`GET https://login.eagleeyenetworks.com/asset/cloud/video.flv`

Parameter           | Data Type     | Description   | Is Required
---------           | -----------   | -----------   | -----------
**id**              | string        | Camera Id     | true
**start_timestamp** | string        | Start Timestamp in EEN format: YYYYMMDDHHMMSS.NNN | true
**end_timestamp**   | string        | End Timestamp in EEN format: YYYYMMDDHHMMSS.NNN | true
**webhook_url**         | string        | The webhook url (must be urlencoded) to trigger | true

### JSON **EVENT** Values
Value                              | Description
---------                          | ---------
ASSET_CLOUD_EVENT_UPLOADED         | The video has been successfully uploaded into the cloud.
ASSET_CLOUD_EVENT_DEMAND_FAILED    | The video failed aquiring a connection to the bridge.
ASSET_CLOUD_EVENT_NOTHING_UPLOAD   | Nothing was uploaded since the video was already in the cloud.
ASSET_CLOUD_EVENT_INVALID_RANGE    | An invalid range (timestamp) was requested.
ASSET_CLOUD_EVENT_ABORT            | General error occurred.

### HTTP Status Codes
HTTP Status Code    | Data Type   
------------------- | ----------- 
201 | Request has been created and webhook will be triggered upon completion or error.

<!--===================================================================-->
## Get List of Images

> Request

```shell
curl -v -G "https://login.eagleeyenetworks.com/asset/list/image?start_timestamp=[START_TIMESTAMP];end_timestamp=[END_TIMESTAMP];id=[CAMERA_ID];asset_class=[ASSET_CLASS];A=[VIDEOBANK_SESSIONID]"
```

> Json Response

```json
[{"t":"PRFR","s":"20141001000000.045"},{"t":"PRFR","s":"20141001000001.045"},{"t":"PRFR","s":"20141001000002.064"},{"t":"PRFR","s":"20141001000003.064"},{"t":"PRFR","s":"20141001000004.064"},{"t":"PRFR","s":"20141001000005.063"},{"t":"PRFR","s":"20141001000006.063"},{"t":"PRFR","s":"20141001000007.096"}]
```

This returns a list of objects, where each object contains the timestamp and type of an image jpeg. When formatting the request, either the 'end_timestamp' or 'count' parameter is required.

### HTTP Request

`GET https://login.eagleeyenetworks.com/asset/list/image`

Parameter           | Data Type     | Description   | Is Required
---------           | -----------   | -----------   | -----------
**id**              | string        | Camera Id     | true
**start_timestamp** | string        | Start Timestamp in EEN format: YYYYMMDDHHMMSS.NNN | true
**asset_class**     | string, enum  | Asset class of the image <br><br>enum: all, pre, thumb | true
end_timestamp       | string        | End Timestamp in EEN format: YYYYMMDDHHMMSS.NNN
count               | int           | Used instead or with an 'end_timestamp' argument. If used with an 'end_timestamp' argument, the count is a limit on the number of entries to return, starting at the starting timestamp. If used without the e argument, returns N entries. Support negative value, which returns N entries before, sorted in reverse order - example -5 return 5 events previous to the specified time.

### Error Status Codes

HTTP Status Code    | Data Type   
------------------- | ----------- 
200 | Request succeeded
301 | Asset has been moved to a different archiver
400 | Unexpected or non-identifiable arguments are supplied
401 | Unauthorized due to invalid session cookie
403 | Forbidden due to the user missing the necessary privileges

<!--===================================================================-->
## Get List of Videos

> Request

```shell
curl -v -G "https://login.eagleeyenetworks.com/asset/list/video?start_timestamp=[START_TIMESTAMP];end_timestamp=[END_TIMESTAMP];id=[CAMERA_ID];options=coalesce;A=[VIDEOBANK_SESSIONID]"
```

> Json Response

```json
[{"s":"20141001000016.768","e":"20141001000100.758"},{"s":"20141001000220.825","e":"20141001000242.774"},{"s":"20141001000256.811","e":"20141001000320.869"},{"s":"20141001000354.761","e":"20141001000422.812"},{"s":"20141001000526.821","e":"20141001000632.829"},{"s":"20141001000746.836","e":"20141001000834.757"},{"s":"20141001000904.749","e":"20141001000932.767"},{"s":"20141001000934.766","e":"20141001001002.777"}]
```

This returns a list of objects, where each object contains the start and end timestamp of a single video clip. When formatting the request, either the 'end_timestamp' or 'count' parameter is required.

### HTTP Request

`GET https://login.eagleeyenetworks.com/asset/list/video`

Parameter           | Data Type     | Description   | Is Required
---------           | -----------   | -----------   | -----------
**id**              | string        | Camera Id     | true
**start_timestamp** | string        | Start Timestamp in EEN format: YYYYMMDDHHMMSS.NNN | true
end_timestamp       | string        | End Timestamp in EEN format: YYYYMMDDHHMMSS.NNN
count               | int           | Used instead or with an 'end_timestamp' argument. If used with an 'end_timestamp' argument, the count is a limit on the number of entries to return, starting at the starting timestamp. If used without the e argument, returns N entries. Support negative value, which returns N entries before, sorted in reverse order - example -5 return 5 events previous to the specified time.
options             | string, enum  | Additional modifier options. 'coalesce' = coalesces spans together. <br><br>enum: coalesce

### Error Status Codes

HTTP Status Code    | Data Type   
------------------- | ----------- 
200 | Request succeeded
301 | Asset has been moved to a different archiver
400 | Unexpected or non-identifiable arguments are supplied
401 | Unauthorized due to invalid session cookie
403 | Forbidden due to the user missing the necessary privileges

<!--===================================================================-->
## Create Timelapse Video

This api is used to request the creation of a new time lapse video for a particular camera and time range. An ID will be returned that you can then use (see GET below) to find out the progress and the download URL of the video, which will be in .mp4 format.

### HTTP Request

`PUT https://login.eagleeyenetworks.com/asset/time_lapse`

Parameter           | Data Type     | Description   | Is Required
---------           | -----------   | -----------   | -----------
**device_id**       | string        | Id of the device the time lapse video should be created for | true
**start_timestamp** | string        | Start timestamp of the time range to be used for the time lapse video, in EEN format. | true
**end_timestamp**   | string        | End timestamp of the time range to be used for the time lapse video, in EEN format. | true
step                | int           | Specifies how many milliseconds between each data point used to create the time lapse. Optional. Defaults to 1000 (1 second).

### Response Json Attributes

Parameter       | Data Type   | Description
---------       | ----------- | -----------
id              | string      | UUID of the time lapse request

### Error Status Codes

HTTP Status Code    | Data Type   
------------------- | ----------- 
202 | Request accepted
400 | Unexpected or non-identifiable arguments are supplied
401 | Unauthorized due to invalid session cookie
403 | Forbidden due to the user missing the necessary privileges
404 | Device matching the device_id was not found
463 | Unable to communicate with the camera, contact support

<!--===================================================================-->
## Retrieve Timelapse Video

Using the ID returned when requesting the time lapse (see PUT above), this api returns information about the time lapse request. It allows you to find out the progress and how to download the resulting video, which will be in .mp4 format.

### HTTP Request

`GET https://login.eagleeyenetworks.com/asset/time_lapse`

Parameter           | Data Type     | Description   | Is Required
---------           | -----------   | -----------   | -----------
**id**              | string        | Id of the time lapse, which was returned when it was created/requested | true
**device_id**       | string        | Id of the device associated with the time lapse when it was created/requested | true

### Response Json Attributes

Parameter           | Data Type     | Description 
---------           | -----------   | ----------- 
percent_complete    | float         | Id of the time lapse, which was returned when it was created/requested
**device_id**       | string        | Id of the device associated with the time lapse when it was created/requested | true

### Error Status Codes

HTTP Status Code    | Data Type   
------------------- | ----------- 
200 | Request succeeded
400 | Unexpected or non-identifiable arguments are supplied
401 | Unauthorized due to invalid session cookie
403 | Forbidden due to the user missing the necessary privileges
404 | id or device_id not found
463 | Unable to communicate with the camera, contact support
