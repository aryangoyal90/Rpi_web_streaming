# Rpi Global Web Streaming

### Required Hardware
- Raspberry Pi (RPI 4 or 5 recommended)
- Raspberry Pi Camera Module (Official Camera Module 3 recommended)
- GSM Module with 4G SIM (SIM7600 or Ublox Series)
- MicroSD Card (At least 16GB with Raspberry Pi OS installed)

### Setting Up Camera
- After connecting the camera module to RPI's CSI port
- Enabling the camera in Raspberry Pi OS:
```
sudo raspi-config
```
- Go to Interfacing Options → Camera → Enable.
- Reboot the RPI
- Testing Camera by below command:
```
rpicam-hello
```

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
- Start the connection
```
sudo wvdial myconnection
```
------------------------------------------------------------------------------
## Install MediaMTX for WebRTC Streaming
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
------------------------------------------------------------------------------
## Make the Stream Accessible Globally
Your Raspberry Pi is behind a private IP from the GSM network, so you need a way to expose it to the internet.

#### Using Ngrok(Option 1):
1. Install Ngrok::
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
