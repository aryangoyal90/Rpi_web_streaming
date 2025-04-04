# Rpi Global Web Streaming

### Required Hardware
- Raspberry Pi (RPI 4 or 5 recommended)
- RPi Camera Module 3 or any other camera module
- GSM Module with 4G SIM (SIM7600 or Ublox Series)
- MicroSD Card (At least 16GB with Raspberry Pi OS installed)
------------------------------------------------------------------------------
### Setting Up Camera
- After connecting the camera module to RPI's CSI port
- Enabling the camera in Raspberry Pi OS:
```
sudo raspi-config
```
- Go to Interfacing Options → Camera → Enable.
- Reboot the RPI
- Testing Camera by below command(if Rpi Camera):
```
rpicam-hello
```
##### Refrence article for setting up Camera
```
https://automaticaddison.com/connect-your-built-in-webcam-to-ubuntu-20-04-on-a-virtualbox/
```
------------------------------------------------------------------------------
### Setting up GSM for Internet
- Connect the GSM module to the Raspberry Pi via USB.
- Check if the device is recognized:
```
lsusb
```
- Install dependencies:
```
sudo apt update
sudo apt install usb-modeswitch wvdial
```
- Configure the connection:
```
sudo nano /etc/wvdial.conf
```
Add:
```
#Replace your_apn with your carrier’s APN (e.g., internet for Vodafone).
# Example: "your_apn" is like "airtelgprs.com" for airtel SIM's

[Dialer myconnection]
Init1 = ATZ
Init2 = AT+CFUN=1
Init3 = AT+CGDCONT=1,"IP","your_apn"
Modem Type = Analog Modem
Baud = 115200
Modem = /dev/ttyUSB3
Phone = *99#
Username = " "
Password = " "
Stupid Mode = 1
```
AT commands to check APN and if Username pass is required:
```
# using minicom to connect to our modem
sudo minicom -D /dev/ttyUSB0

AT+COPS?     #getting operator name i.e 'Airtel'
AT+CGDCONT?  #Listing APN

# if no APN appears we can set it manually (OPTIONAL)
AT+CGDCONT=1,"IP","airtelgprs.com"

# checking if username, Pass are nedded
AT+CGAUTH?
```

- Start the connection
```
sudo wvdial myconnection
```
------------------------------------------------------------------------------
## Install MediaMTX for WebRTC Streaming
First install AppArmor Utilities
```
sudo apt update
sudo apt install apparmor-utils
```
1. Download MediaMTX:
```
wget https://github.com/bluenviron/mediamtx/releases/latest/download/mediamtx_linux_arm64v8.tar.gz
```
2. Extract and move the files:
```
tar -xvzf mediamtx_linux_arm64v8.tar.gz
sudo mv mediamtx /usr/local/bin/
```
3. Download MediaMTX:
```
sudo nano mediamtx.yml
```
Add:
```
paths:
  cam:
    source: rpiCamera
```
For Integrated webcam
```
paths:
  cam:
    runOnInit: ffmpeg -f v4l2 -i /dev/video0 -s 320x240 -r 15 -c:v h264 -pix_fmt yuv420p -preset ultrafast -tune zerolatency -b:v 300k -f rtsp rtsp://localhost:$RTSP_PORT/$MTX_PATH
    runOnInitRestart: yes
```
4. Start MediaMTX:
```
mediamtx
```
5. Test the stream:
Open a browser and visit:
```
http://<your-pi-ip>:8889/cam

# Replace <your-pi-ip> with your public IP
```
##### Refrence stackoverflow
```
https://stackoverflow.com/questions/78259527/unable-to-stream-video-file-from-mediamtx-media-server-to-browser-via-webrtc
```
------------------------------------------------------------------------------
## Video Compression using FFmpeg
- Integrating FFmpeg with MediaMTX
```
ffmpeg -i input.mp4 -c:v libx264 -preset ultrafast -b:v 800k -c:a aac output.mp4
```
- Command explanation:
```
i input.mp4: Specifies the input video file
-c:v libx264: Encodes the video using the H.264 codec
-preset ultrafast: prioritizes speed over compression
**-b:v 800k: Sets the video bitrate to 800 kbps**
-c:a aac: Encodes the audio using AAC
output.mp4: Specifies the output file
```

#### Options for Video compression
- Use MediaMTX's Built-in Encoding Options
- **Optimize Resolution and Frame Rate(reducing resolution)**
- Gstreamer
------------------------------------------------------------------------------
## Make the Stream Accessible Globally
Our Raspberry Pi is behind a private IP from the GSM network, so we need a way to expose it to the internet.

#### Using Ngrok:
1. Install Ngrok:
```
wget https://bin.equinox.io/c/4VmDzA7iaHb/ngrok-stable-linux-arm.zip
unzip ngrok-stable-linux-arm.zip
sudo mv ngrok /usr/local/bin/
```
2. Authenticate Ngrok:
Note: we have to sign up on Ngrok for auth token
```
ngrok authtoken <your-ngrok-auth-token>
```
3. Start tunneling:
```
ngrok http 8889
```
4. Ngrok will provide a public URL like:
```
https://abcd1234.ngrok.io/cam
```
this will be our global Url

#### More Options:
includes Nginx (uses AWS cloud for streaming)

---------------------------------
# OpenCV or AI Integration in Streaming
Motion detection by this article:
```
https://pyimagesearch.com/2019/09/02/opencv-stream-video-to-web-browser-html-page/
```
![Image](https://github.com/user-attachments/assets/d2c9b44e-e249-42d5-8d80-6a64fe67f35d)
