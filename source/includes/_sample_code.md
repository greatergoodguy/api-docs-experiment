# How To's and Example Code

<!--===================================================================-->
## Making API Calls With Curl
In this section, we will walk you through the process of making API requests using the ‘curl’ command line tool. The Eagle Eye APIs are platform agnostic and we use them to create the web, Android, and iOS Eagle Eye clients. Curl is a tool for transferring data to and from a server, using a wide range of supported protocols, including HTTP/HTTPS, which is what we are interested in. Curl can be installed by going to this site. http://curl.haxx.se/.

With curl installed, the next step is to log in and have a valid session, so that we can freely use any of the APIs. Logging in, is a two step process consisting of authentication and authorization. The authentication API takes in 2 parameters. Our curl common will look like this. The [USERNAME] and [PASSWORD] need to be valid for the API request to return successfully.

`
curl --request POST https://login.eagleeyenetworks.com/g/aaa/authenticate --data 'username=[USERNAME]&password=[PASSWORD]'
`

The ‘–request’ flag specifies the type of request and can be set to GET, POST, PUT, and DELETE. The ‘data’ flag species the parameters of the API query. Upon running this command with valid credentials, we receive a Json formatted response, containing a key/value pair for ‘token’, which will look something like this.

`
{ “token”: “YrZF/8jf7W0rKcqNTugqidq…………4dZWeNOcNsuenTXc9fQVtvp2vI75g==” }
`

This token is a required parameter for making the Authorize API request. Copy the value of the token in order to have it on hand when creating the Authorize API request. Now we make the authorization API request using this curl command.

`
curl -D - --request POST https://login.eagleeyenetworks.com/g/aaa/authorize --data-urlencode token=[TOKEN]
`

The output are the headers of the API request followed by the response body. The ‘-D’ flag is used to write the protocol headers. Here is the description from the man pages.

`
-D, –dump-header <file> Write the protocol headers to the specified file. This option is handy to use when you want to store the headers that a HTTP site sends to you. Cookies from the headers could then be read in a second curl invocation by using the -b, –cookie option! The -c, –cookie-jar option is however a better way to store cookies.
`

Note that the ‘-‘ after the ‘-D’ indicates that the output “file” is stdout. One of the header elements will be “Set-Cookie: videobank_sessionid=[VIDEOBANK_SESSIONID]“. Copy ‘videobank_sessionid=[VIDEOBANK_SESSIONID]‘ into the clipboard as this cookie will need to be set for all other API requests. The curl request for getting a list of devices will look as such.

`
curl --cookie "videobank_sessionid=[VIDEOBANK_SESSIONID]" --request GET https://login.eagleeyenetworks.com/g/list/devices
`

The ‘videobank_sessionid’ cookie will need to be set for any other Eagle Eye API that requires a valid session.

<!--===================================================================-->
## Constructing Layouts

> Get /layout/list

```json
[
    [
        "fc7fa3a4-66ea-11e3-8ca8-523445989f37",
        "Default",
        [
            "iphone"
        ],
        "SWRD"
    ],
    [
        "14fbd682-66eb-11e3-8ca8-523445989f37",
        "Ekta 2",
        [
            "iphone"
        ],
        "SWRD"
    ],
    [
        "23535930-66eb-11e3-977b-ca8312ea370c",
        "Ekta 3",
        [
            "desktop"
        ],
        "SWRD"
    ]
]
```

When a user logs onto the Eagle Eye system, they are greeted with a grid of cameras, with each cell representing a camera pane. These panes can be of varying size so that the user can customize the layout to their liking. In this tutorial, we will demonstrate how to use the APIs to build these layouts so they are consistent on all platforms.

Upon being logged in, we make a request to the GET /layout/list API. This returns an array of Layout objects. Do note that this is not the same model as what is returned by the GET /layout API request. The one returned by the /layout/list API is an abridged version with only the most important attributes. The response of the request will look like this.

`
Get /layout/list
`

We take the layout id attribute for each layout of interest and pass it to the Get /layout API request. This will contain the information we need to construct the layout.

`
Get /layout
`

> Get /layout

```json
{
    "id": "811dbe48-66eb-11e3-8ca8-523445989f37",
    "shares": [
        [
            "ca0c35cb",
            "SWRD"
        ]
    ],
    "name": "mob dev 1",
    "permissions": "SWRD",
    "current_recording_key": null,
    "types": [
        "desktop"
    ],
    "configuration": {
        "settings": {
            "camera_row_limit": 3,
            "automatic_rotation": false,
            "camera_name": false,
            "camera_border": "false",
            "camera_aspect_ratio": "0.75"
        },
        "panes": [
            {
                "type": "preview",
                "pane_id": 0,
                "name": "Conference Room",
                "cameras": [
                    "100f6136"
                ],
                "size": 2
            },
            {
                "type": "preview",
                "pane_id": 1,
                "name": "NW Parking",
                "cameras": [
                    "10003254"
                ],
                "size": 2
            }
        ]
    }
}
```

We get a wealth of good information, but the information specific to setting up the pane layout is in the ‘configuration’ json. Within that json, there is an attribute called ‘panes’ which is an array of individual pane objects. Each pane specifies the camera_id and the size of the pane. The size represents the width and height in number of cells. A size of 2 means that the pane is 2 cells in width and height, so it occupies a total of 4 cells. A size of 3 would occupy 9 cells.

The other important factor to know is the size of grid holding the panes, specifically the number of columns. For the Eagle Eye web client, a browser can be resized to be a narrow strip or the full width of the screen. The layout will dynamically adjust the number of columns based on the width of the window. Mobile devices have fixed screen sizes, so for the iOS and Android smartphone clients, we set the number of columns to three.

Now that we have the order of the panes, the size of each pane, and the size of the grid, we can construct our layout. This proved to be of varying difficulty depending on the platform. The web client uses a robust packing library, Packery, which is based on a bin packing algorithm. http://metafizzy.co/blog/packery-released/. This library minimizes empty space while preserving the order as best as possible. Using Packery reduced the development time for this feature significantly.

At the time of this writing, Android does not have a robust library for packing the panes so the algorithm to do so was written from scratch. The goal was to mimic the Packery library as best as possible. The Android algorithm works as such:


 1. Remove the next image from the ‘panes’ array and place it in the ‘panes_for_analysis’ list.
 2. Analyze the panes in ‘panes_for_analysis’. If there is a fully packed block, remove those panes and add them to the layout
 3. If the ‘panes’ array is not empty, GOTO 1, else GOTO 4
 4. Add the remaining panes from ‘panes_for_analysis’ to the layout.

This is the algorithm at a high level, though the specifics can get a little more complex, such as determining whether a fully packed block exists. The state of a fully packed block is also dependent on the number of columns for the grid.

The ease of constructing layouts is highly dependent on the robustness of the 3rd party library. In the case that one does not exist, we fall back to our home grown packing algorithm.

<!--===================================================================-->
## Playing Live Video
Video playback functionality can be accessed through the ‘/asset/play/video.{video_format}’ API. We will show you how to use this API to play live video, though the same API can also be used to play historic video.

Below is the Javascript code that creates the URL for playing live video footage with a HTML flash video player. You can run the javascript code on this site to generate the URL string. http://writecodeonline.com/javascript/.

The caller of the API need to supply 2 parameters. which are [DEVICE_ID] and [VIDEOBANK_SESSIONID]. The [DEVICE_ID] represents the id of the camera of interest. The [VIDEOBANK_SESSIONID] is used for authentication and can be found in the response header of the /aaa/authorization API.

`
eagleEyeLiveVideoApiUrl = "https://login.eagleeyenetworks.com/asset/play/video.flv" +
    "?id=[DEVICE_ID]" +
    "&start_timestamp=stream_"+(new Date().getTime()) +
    "&end_timestamp=+300000" +
    "&A=[VIDEOBANK_SESSIONID]";
`

`     
htmlFlashVideoPlayerUrl = "https://login.eagleeyenetworks.com/strobe/embed.html?src="+encodeURIComponent(eagleEyeLiveVideoApiUrl)+"&autoPlay=true&bufferingOverlay=false&streamType=live&bufferTime=1&initialBufferTime=1&expandedBufferTime=5&liveBufferTime=2&liveDynamicStreamingBufferTime=4&minContinuousPlaybackTime=5";
`

`
document.write(htmlFlashVideoPlayerUrl);
`

Notice that we have 2 variables; ‘eagleEyeLiveVideoApiUrl’ and ‘htmlFlashVideoPlayerUrl’. The ‘htmlFlashVideoPlayerUrl’ variable contains the video player being used to play the Flash Player. Users are free to use any video player of this liking, and the one referenced in the code is just an example video player we are using. The output of this JS code is a URL that looks like this. Use this to embed live video into your application.

`
https://login.eagleeyenetworks.com/strobe/embed.html?autoPlay=true&src=https%3A%2F%2Flogin.eagleeyenetworks.com%2Fasset%2Fplay%2Fvideo.flv%3Fc%3D[DEVICE_ID]%3Bt%3Dstream_1401291315740%3Be%3D%2B300000%3BA%3D[VIDEOBANK_SESSIONID]&bufferingOverlay=false&streamType=live&bufferTime=1&initialBufferTime=1&expandedBufferTime=5&liveBufferTime=2&liveDynamicStreamingBufferTime=4&minContinuousPlaybackTime=5
`

<!--===================================================================-->
## Long Polling

> Json Request for Post /poll

```json
{
     "cameras": {
         "100c30ea": {
             "resource": [
                 "event",
                 "pre",
                 "thumb",
                 "status"
             ],
             "event": [
                 "ECON",
                 "ECOF",
                 "AELI",
                 "AELO",
                 "EMES",
                 "EMEE",
                 "CECF",
                 "CSAT",
                 "CSDT",
                 "CONN",
                 "COFF",
                 "ESES",
                 "ESEE"
             ]
         },
         "100f6136": {
             "resource": [
                 "event",
                 "pre",
                 "thumb",
                 "status"
             ],
             "event": [
                 "ECON",
                 "ECOF",
                 "AELI",
                 "AELO",
             ]
         },
         "1009c1ab": {
             "resource": [
                 "event",
                 "pre",
                 "thumb",
                 "status"
             ],
             "event": [
                 "EMES",
                 "EMEE",
                 "CECF",
                 "CSAT",
                 "CSDT",
                 "CONN",
                 "COFF",
                 "ESES",
                 "ESEE"
             ]
         }
}
```

Upon entering the Eagle Eye system, the user is presented with a grid of cameras. These cameras are retrieving images in real time through a poll stream. In this tutorial, we will walk you through the steps to set up the poll stream for long polling using the /poll API.

Long polling is used in the mobile clients. The process is to first register to the poll stream using the POST /poll API followed by constant API requests for GET /poll.

The POST /poll API is used to initialize the poll stream and to register the events we want to listen for. For the mobile clients, we are listening to resource type ‘pre’ and ‘status’, which are the preview images and status bits. Since the data we are sending to the server is a json, the content-type of this request will be application/json. Here is an example of what the the data may look like.

Once the POST /poll request has been made successfully, a token is returned to the user. This token can be used to successfully make all subsequent GET /poll requests. If the token is not desired, the same requests can be made if you have the ‘ee-poll-ses’ cookie from the POST /poll request.

The GET /poll request should be called frequently so that new data can arrive as soon as possible. The response may be empty or it may look something like this.

> Response for Get /poll

```json

{
    "cameras": {
        "10003254": {
            "event": {
                "PRFR": {
                    "cameraid": "10003254",
                    "timestamp": "20140528224954.312",
                    "file_offset": 17804500,
                    "frame_size": 6152,
                    "previewid": 1401314400
                }
            }
        },
        "100a9541": {
            "event": {
                "PRFR": {
                    "cameraid": "100a9541",
                    "timestamp": "20140528224955.507",
                    "file_offset": 12004385,
                    "frame_size": 3706,
                    "previewid": 1401314400
                }
            },
            "pre": "20140528224955.507"
        }
    }
}
```

Only attributes with updated information will be returned in the response payload. For the mobile apps, we monitor the ‘pre’ attribute for new timestamps, and when a new timestamp does come in, we make the appropriate API call to retrieve the camera image.

The power of this API lies in the ability of being able to control what events and resource types to listen to. This allows updates to the camera to be known in real time.


<!--===================================================================-->
## Creating Timelapse Video
In this tutorial, we will walk you through how to generate a timelapse video from images in a time range. The code will be shown in Objective-C and can be compiled for either iOS or OS X. We will need a library for making asynchronous network requests. In this example, we’ll be using AFNetworking: [http://cocoadocs.org/docsets/AFNetworking/2.2.4/](http://cocoadocs.org/docsets/AFNetworking/2.2.4/).

The full source code is available here: [https://github.com/cazares/EENTimelapse](https://github.com/cazares/EENTimelapse)

The first step is to initialize our HTTP request operation manager that we will be using to make asynchronous requests. We will store this manager in a property. We’ll also use three other properties – sessionID, timelapseID, and cameraID to help us make subsequent API requests.

`
Step 1
`

> Step 1

```objc
@property (nonatomic, strong) AFHTTPRequestOperationManager *manager;
@property (nonatomic, strong) NSString *sessionID;
@property (nonatomic, strong) NSString *timelapseID;
@property (nonatomic, strong) NSString *cameraID;
 
- (id)init {
  self = [super init];
  if (self) {
    self.manager = [AFHTTPRequestOperationManager manager];
    self.manager.requestSerializer = [AFJSONRequestSerializer serializer];
    self.manager.responseSerializer = [AFHTTPResponseSerializer serializer];
  }
  return self;
}
```

After initialization, we’ll need to log in to the Eagle Eye system through the authentication and authorization API calls. To log in, we will make a POST /aaa/authenticate request followed by a POST /aaa/authorize request. The /aaa/authorize response will contain a cookie. We will save its value in our local sessionID property to use in subsequent API calls via the ‘A’ parameter.

`
Step 2
`

> Step 2

```objc
- (void)authenticateWithUsername:(NSString *)username
                        password:(NSString *)password {
  NSDictionary *parameters = @{ @"username": username,
                                @"password": password };
  [self.manager POST:@"https://login.eagleeyenetworks.com/g/aaa/authenticate"
parameters:parameters
  success:^(AFHTTPRequestOperation *operation, id responseObject) {
    NSError *error;
    NSDictionary *response = [NSJSONSerialization JSONObjectWithData:responseObject
                                                             options:NSJSONReadingAllowFragments
                                                               error:&error];
    [self authorizeUserWithToken:response[@"token"]];
  }
  failure:^(AFHTTPRequestOperation *operation, NSError *error) {
    NSLog(@"Error: %@", error);
  }];
}
 
- (void)authorizeUserWithToken:(NSString *)token {
  NSDictionary *parameters = @{ @"token": token };
  [self.manager POST:@"https://login.eagleeyenetworks.com/g/aaa/authorize"
parameters:parameters
  success:^(AFHTTPRequestOperation *operation, id responseObject) {
    NSArray *cookies = [NSHTTPCookie cookiesWithResponseHeaderFields:operation.response.allHeaderFields
                                                              forURL:operation.response.URL];
    if (cookies.count > 0) {
      NSHTTPCookie *cookie = (NSHTTPCookie *)cookies[0];
      self.sessionID = cookie.value;
    }
    [self getDeviceListAndRequestTimelapseOnSuccess];
  }
  failure:^(AFHTTPRequestOperation *operation, NSError *error) {
    NSLog(@"Error: %@", error);
  }];
}
```

Now that we are logged in, we will select a camera to be the source of the timelapse video. We need to first get a list of all camera devices by making a request to /device/list. We’ll want to only get devices of type ‘camera’ by using the ‘t’ parameter. Additionally, we’ll do a quick check of the camera status to make sure we don’t grab any offline cameras.

`
Step 3
`

> Step 3

```objc
- (void)getDeviceListAndRequestTimelapseOnSuccess {
  // specify device type of camera
  NSDictionary *parameters = @{ @"t": @"camera",
                                @"A": self.sessionID };
  [self.manager GET:@"https://login.eagleeyenetworks.com/g/device/list"
parameters:parameters
   success:^(AFHTTPRequestOperation *operation, id responseObject) {
     NSError *error;
     NSArray *response = [NSJSONSerialization JSONObjectWithData:responseObject
                                                         options:NSJSONReadingMutableContainers
                                                           error:&error];
     // see https://apidocs.eagleeyenetworks.com/apidocs/#!/device/deviceList_get_4
     // for indexing of subarrays and parsing more information
     // from the /device/list api call
     for (NSArray *subarray in response) {
       int cameraStatus = [subarray[10] intValue];
       BOOL cameraRegistered = (cameraStatus & (1 << 0)) != 0;
       BOOL cameraOnline = (cameraStatus & (1 << 1)) != 0;
       BOOL cameraOn = (cameraStatus & (1 << 2)) != 0;
       if (!cameraOn || !cameraRegistered || !cameraOnline) {
         continue;
       }
       // for the purposes of this example, we'll just request timelapse for first camera
       self.cameraID = subarray[1];
       [self putTimelapseWithStartTimeString:@"20140530163000.000"
                               endTimeString:@"20140530163500.000"];
       break;
     }
   }
   failure:^(AFHTTPRequestOperation *operation, NSError *error) {
     NSLog(@"Error: %@", error);
   }];
}
```

We’ll pick the first non-offline camera from the /device/list response and request an arbitrary start and end time for the timelapse video. PUT /time_lapse starts a timelapse video job to create the video from images in the time frame. In this method, we will kick off a local timer to repeatedly check the status of the timelapse job.

`
Step 4
`

> Step 4

```objc
bool timelapseReadyToDownload = false;
- (void)putTimelapseWithStartTimeString:(NSString *)startTimeString
                          endTimeString:(NSString *)endTimeString {
  NSDictionary *parameters = @{ @"device_id": self.cameraID,
                                @"start_timestamp": startTimeString,
                                @"end_timestamp": endTimeString,
                                @"A": self.sessionID };
  [self.manager PUT:@"https://login.eagleeyenetworks.com/g/time_lapse"
parameters:parameters
   success:^(AFHTTPRequestOperation *operation, id responseObject) {
     NSError *error;
     NSDictionary *response = [NSJSONSerialization JSONObjectWithData:responseObject
                                                              options:NSJSONReadingAllowFragments
                                                                error:&error];
     self.timelapseID = response[@"id"];
     [NSTimer scheduledTimerWithTimeInterval:1.0
                                      target:self
                                    selector:@selector(checkTimelapseProgress)
                                    userInfo:nil
                                     repeats:YES];
   }
   failure:^(AFHTTPRequestOperation *operation, NSError *error) {
     NSLog(@"Error: %@", error);
   }];
}
```

After the timelapse job is created, we need to check on its status. We can check status of the job via the GET /time_lapse API request. The server responds to the GET request with a percent_complete and a download_url. We’ll repeatedly check the percent_complete until it reaches 100 and we have a download_url.

`
Step 5
`

> Step 5

```objc
- (void)checkTimelapseProgress {
  if (timelapseReadyToDownload) {
    return;
  }
  NSDictionary *parameters = @{ @"id": self.timelapseID,
                                @"device_id": self.cameraID,
                                @"A": self.sessionID };
  [self.manager GET:@"https://login.eagleeyenetworks.com/g/time_lapse"
parameters:parameters
   success:^(AFHTTPRequestOperation *operation, id responseObject) {
     NSError *error;
     NSDictionary *response = [NSJSONSerialization JSONObjectWithData:responseObject
                                                              options:NSJSONReadingAllowFragments
                                                                error:&error];
     CGFloat percentComplete = (CGFloat)[response[@"percent_complete"] floatValue];
     NSString *downloadUrl = (NSString *)response[@"url"];
     if (percentComplete >= 100.0f && downloadUrl) {
       NSLog(@"Ready for download!");
       NSLog(@"Here's your url to download the video: %@", downloadUrl);
       NSLog(@"Your session id is: %@", self.sessionID);
       timelapseReadyToDownload = true;
       downloadUrl = [NSString stringWithFormat:@"%@?A=%@", downloadUrl, self.sessionID];
       [self playVideoFromDownloadUrl:downloadUrl];
     }
     else {
       NSLog(@"percent complete: %f", percentComplete);
     }
   }
   failure:^(AFHTTPRequestOperation *operation, NSError *error) {
     NSLog(@"Error: %@", error);
   }];
}
```

Now that we have a download url, the last step is to actually download the video and play it. On iOS, we will need to explicitly download the video and then tell a new instance of MPMoviePlayerViewController to play the video file. On OS X, Safari will automatically handle the downloading for us if we just tell our shell to open up the download url.

`
Step 6
`

> Step 6

```objc
- (void)playVideoFromDownloadUrl:(NSString *)downloadUrl {
#if TARGET_OS_IPHONE
  NSURLRequest *request = [NSURLRequest requestWithURL:[NSURL URLWithString:downloadUrl]];
  AFHTTPRequestOperation *operation = [[AFHTTPRequestOperation alloc] initWithRequest:request];
  NSArray *paths = NSSearchPathForDirectoriesInDomains(NSDocumentDirectory, NSUserDomainMask, YES);
  NSString *path = [[paths objectAtIndex:0] stringByAppendingPathComponent:@"een-timelapse.mp4"];
  operation.outputStream = [NSOutputStream outputStreamToFileAtPath:path append:NO];
  [operation setCompletionBlockWithSuccess:^(AFHTTPRequestOperation *operation, id responseObject) {
    NSLog(@"Successfully downloaded file to %@", path);
    NSURL *contentUrl = [NSURL fileURLWithPath:path];
    MPMoviePlayerViewController *videoController = [[MPMoviePlayerViewController alloc] initWithContentURL:contentUrl];
    videoController.moviePlayer.movieSourceType = MPMovieSourceTypeFile;
    [videoController.moviePlayer prepareToPlay];
    [videoController.moviePlayer play];
    [self presentMoviePlayerViewControllerAnimated:videoController];
  } failure:^(AFHTTPRequestOperation *operation, NSError *error) {
    NSLog(@"%@", error);
  }];
  [operation start];
#else
  NSString *commandString = [NSString stringWithFormat:@"open %@", downloadUrl];
  system([commandString cStringUsingEncoding:NSUTF8StringEncoding]);
#endif
}
```

And that was it! We’ve now gone through the entire flow of creating and playing a timelapse video. Now you can use this knowledge to use in your own Eagle Eye app.