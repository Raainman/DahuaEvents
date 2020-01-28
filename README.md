# DahuaEvents
This is a branch of CameraEvents repo, because the snapshot url's there are wrong for my camera's. I am going to use this repo as a basis to do some Dahua api programming.

Docker/Python service for attaching to Dahua camera events api.  Posts messages to mqtt upon events.

Can post an image into the mqtt payload also (which https://github.com/jpmens/mqttwarn can post to slack and pushover).

# Running

I build and run in docker mostly.  It's built on a amd64 linux machine but I create images for arm32v7 and arm64v8.  Which is why there is a Dockerfile.cross instead of just a regular docker file.  

Copy the sample config.master.ini to config.ini

Run command:
```
docker run -v <config path>:/opt/cameraevents/conf psyciknz/cameraevents
```
Config path is the path where the config.ini file will exist.

Otherwise, clone into a directory, create the config file, and run with python CameraEvents.py

The Dockerfile.cross is what I use for cross architecture builds.  So it is not directly buildable as it needs some modifications to remove the qemu emulation for building an arm32 image on an amd.  The instructions for building are a little hard to describe here.



# Configuration.

Copy the config-master.ini to config.ini in the directory with the script (or where the <config path> is set to if using the docker run command)

Set your mqtt options below:
```
[MQTT Broker]
IP: localhost                           ;MQTT Server name
Port: 1883                              ;MQTT Port
#not currently supported
Mqtt_Username = ''                      ;MQTT username, without qoutes
Mqtt_Password = ''                      ;MQTT password, without qoutes
BaseTopic = 'CameraEvents'
```

User and password are not yet supported on the mqtt server.

List your cameras in the next section:
```
[Cameras]
camera1=Example1
camera2=Example2
```

Each section then needs it's own configuration:
```
[Example1]
host=192.168.1.108
protocol=http
name=NVR
isNVR=True
port=80
user=USER
pass=PASSWORD
auth=digest
events=VideoMotion,CrossLineDetection,AlarmLocal,VideoLoss,VideoBlind
			
[Example2]
host=192.168.1.109
protocol=http
name=IPC
port=80
isNVR=False
user=USER
pass=PASSWORD
auth=basic
events=VideoMotion,CrossLineDetection,AlarmLocal,VideoLoss,VideoBlind
```

# MQTT Events
To subscribe to all events use the following:
```
 mosquitto_sub -v -h <mqttserver> -t CameraEvents/#
```

There is an online/offline message
```
CameraEvents/$online False
```
Which is the last will and testament that can show you the service is alive.

An alert message will be posted to the topic as follows:
```
CameraEvents/<code>/<channel> 
```
Where code is VideoMotion (as per the config events).  Channel, will either be the channel name or the channel number.
The payload will be ON/OFF for using as a motion detector.

eg
```
CameraEvents/VideoMotion/NVR:3 ON
CameraEvents/NVR:3/Image {"message": "Motion Detected: NVR:3", "imagebase64": "<base64 encoded image data>"}
CameraEvents/VideoMotion/NVR:3 OFF
```

IVS Messages are a bit more message like.
eg
```
CameraEvents/IVS/Garage CrossLineDetection With Human in RightToLeft direction for Gate region
```
I'll add specifics later for IVS, as I've seen the "human" be vehicle and smoke also...I'll problably add a fitler for these.
# Problems/Change History

2019-02-02 
- Found Solution to snapshot problem, but new firmware (for me) is index+1 - this isn't currently configurable.
  - Added IVS topic.
  - Added travis testing.

2019-01-15 
- As per some new firmware, the snapshot image command has stopped working, and the channel list.  I'm attempting to find a work around.

