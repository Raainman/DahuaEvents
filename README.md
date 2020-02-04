# DahuaEvents
This is a branch of CameraEvents repo, because the snapshot url's there are wrong for my camera's. I am going to use this repo as a basis to do some Dahua api programming.

Docker/Python service for attaching to Dahua camera events api.  Posts messages to mqtt upon events.

My personal setup is to connect DahuaEvents via MQTT / NodeRed to Domoticz (Home automation).

if you have any questions or remarks drop me a mail at raainman.github@gmail.com


# Running

I build and run in docker mostly, and run on x86. 

Copy the sample config.master.ini to config.ini

Run command:
```
docker run -v <config path>:/opt/cameraevents/conf raainman/dahuaevents
```
Config path is the path where the config.ini file will exist.

Otherwise, clone into a directory, create the config file, and run with python CameraEvents.py


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
CameraEvents/<code>/<channel>/<Rule>
```
Where code is VideoMotion (as per the config events).  Channel, will either be the channel name or the channel number.
The payload will be ON/OFF for using as a motion detector.

eg
```
CameraEvents/VideoMotion/NVR:3 ON
CameraEvents/NVR:3/Image {"message": "Motion Detected: NVR:3", "imagebase64": "<base64 encoded image data>"}
CameraEvents/VideoMotion/NVR:3 OFF
```

IVS Messages send a larger message with more info.
eg
```
CameraEvents/IVS/<Todo>
```
I'll add specifics later for IVS, as I've seen the "human" be vehicle and smoke also...I'll problably add a fitler for these.

# NewFile

Currently adding a message for all the files that are broadcasted by the camera. Subscribe to the event 'NewFile' in the config file, any dav, or snapshot by the camera will be posted on this event. 
Note: My camera's have storage, I have put a Micro-SD in them for local storage. Have not tested if the NVR's also post this event.

```
<maintopic>/NewFile/<channel>/{ 'Code':NewFile,'File':"/mnt/sd/2020-02-03/001/dav/16/16.07.20-16.07.50[M][0@0][0].jpg",'Extension':'jpg','Size':6656491 }
```

This file can be downloaded bij calling RPC_Loadfile like this;

```
192.168.1.108/cgi-bin/RPC_Loadfile/mnt/sd/2020-02-03/001/jpg/16/07/20[M][0@0][0].jpg
```

Note: Work in progress ... I'm testing this functionality

# Problems/Change History

04-02-2020	Adding NewFile - Work in Progress

