# Setup, test and develop with 1NCE Zephyr Blueprints
  
## Setup nRF Connect SDK on Ubuntu 22.04 and newer  

1. Make sure your system is up-to-date:
```
sudo apt update
sudo apt upgrade
```
2. Install required dependencies: 
```
sudo apt install --no-install-recommends git cmake ninja-build gperf \
  ccache dfu-util device-tree-compiler wget \
  python3-dev python3-pip python3-setuptools python3-tk python3-wheel xz-utils file \
  make gcc gcc-multilib g++-multilib libsdl2-dev libmagic1
```
3. Verfiy versions of the main dependencies installed with:
```
cmake --version
python3 --version
dtc --version
```
## Install west

Open a terminal window and enter the following command to install west:

```
pip3 install --user west
echo 'export PATH=~/.local/bin:"$PATH"' >> ~/.bashrc 
source ~/.bashrc
```
## Get the nRF Connect SDK code  

*(blueprints have been tested on the 2.2.0 version of the nRF Connect SDK)*  

1. Create a folder named `ncs` for all nRF Connect SDK repositories. Change the path how you like:
```
mkdir ~/ncs
```
2. Go into the newly created folder:
```
cd ncs/
```
3. Initialize the nRF Connect SDK 2.2.0 repository with west inside the `ncs/` folder: 
```
west init -m https://github.com/nrfconnect/sdk-nrf --mr v2.2.0
```
There should be a folder named `nrf/`, go into it.  
4. After that clone the project repositories:
```
west update
```
*(this might take some time)*  
5. Export a Zephyr CMake package. This allows CMake to automatically load the boilerplate code required for building nRF Connect SDK applications:
```
west zephyr-export
```
## Installing additional Python dependencies

Go into the `ncs/` folder:
```
cd ncs/
```
and ender the following commands:
```
pip3 install --user -r zephyr/scripts/requirements.txt
pip3 install --user -r nrf/scripts/requirements.txt
pip3 install --user -r bootloader/mcuboot/scripts/requirements.txt
```
## Installing the toolchain

*(you can skip this if you already have the Zephyr SDK installed and setup)*  
*(blueprints have been tested on Zephyr SDK 0.16.4)*

1. Download and verify the Zephyr SDK 0.16.4 bundle:
```
cd ~
wget https://github.com/zephyrproject-rtos/sdk-ng/releases/download/v0.16.4/zephyr-sdk-0.16.4_linux-x86_64.tar.xz
wget -O - https://github.com/zephyrproject-rtos/sdk-ng/releases/download/v0.16.4/sha256.sum | shasum --check --ignore-missing
```
2. Extract the Zephyr SDK bundle archive in the home folder:
`tar xvf zephyr-sdk-0.16.4_linux-x86_64.tar.xz`

3. Run the Zephyr SDK bundle sectup script:
```
cd zephyr-sdk-0.16.4/
./setup.sh
```
4. Install the udev rules, which allow you to flash most Zephyr boards as a regular user:
```
sudo cp ~/zephyr-sdk-0.16.4/sysroots/x86_64-pokysdk-linux/usr/share/openocd/contrib/60-openocd.rules /etc/udev/rules.d
sudo udevadm control --reload
```
5. Navigate to the `ncs` folder and enter the following command:  
```
source zephyr/zephyr-env.sh
```
## Installing nRF Command Line Tools 
*(blueprints have been tested on version 10.24.2 Linux x86 64)*  

Get the latest debian package from [here](https://www.nordicsemi.com/Products/Development-tools/nRF-Command-Line-Tools/Download?lang=en#infotabs). Select "Linux x86 64" and click on the .deb file.  
Or click [here](https://nsscprodmedia.blob.core.windows.net/prod/software-and-other-downloads/desktop-software/nrf-command-line-tools/sw/versions-10-x-x/10-24-2/nrf-command-line-tools_10.24.2_amd64.deb) to download it directly.  

Next go to the location you downloaded it from and execute this command:
```
sudo dpkg -i nrf-command-line-tools_10.24.2_amd64.deb 
```
after that execute this command:
```
sudo apt install /opt/nrf-command-line-tools/share/JLink_Linux_V794e_x86_64.deb --fix-broken
```
The nRF Command Line Tools should now be installed along with its dependencies.  
You can test it with this command: 
```
nrfjprog -v
```
You should see something like this:
```
$ nrfjprog -v
nrfjprog version: 10.24.2 external
JLinkARM.dll version: 7.94e
```
## Integrating the 1NCE IoT C SDK 

1. Change into your home directory.  
```
cd ~/
```
2. Execute this command:  
```
west init -m https://github.com/1NCE-GmbH/1nce-iot-c-sdk
```
3. After that is finished change into the newly created directory:  
```
cd ~/1nce-iot-c-sdk/
``` 
4. In the directory execute this command:  
```
west update
```
5. Next go into the previously created and set up `~/ncs/nrf` directory:  
```
cd ~/ncs/nrf/
```
6. Open the `west.yml` file with a text editor of your choice and 
search for the keybord `name-allowlist` and add `nce-sdk` to the list.  
**Keep in mind to add it in alphabetically**  
*(under nanopb)*  
Save and close the file.
7. Change directory to `cd ~/ncs/zephyr/submanifests/`.  
8. Rename the example.yaml.sample to example.yaml.  
```
mv example.yaml.sample example.yaml
```
9. Open the `example.yaml` with your text editor of choice and edit it so that is looks like this:  
```
manifest:
  projects:
    - name: nce-sdk
      url: https://github.com/1NCE-GmbH/1nce-iot-c-sdk
      revision: main

```
Save and close the file.  
10. Inside the `~/ncs/` directory run `west update`. This will integrate the 1NCE IoT C SDK.  

## Cloning and Flashing the Zephyr Blueprints
1. Go into the `~/ncs/` directory and clone the blueprint repository.  
```
git clone https://github.com/1NCE-GmbH/blueprint-zephyr.git
```
2. Change directory to the `nrf/` folder:  
```
cd nrf/
```
3. And execute this command to build the UDP Blueprint:     
Build for the nRF9160 DK board:  
```
west build -p -b nrf9160dk_nrf9160_ns ../blueprint-zephyr/nce_udp_demo/
```

Build for the thingy:91 board:  
```
west build -p -b thingy91_nrf9160_ns ../blueprint-zephyr/nce_udp_demo/
```
### Flash the nRF9160 DK board    
```
west flash
```
### Booting the Thingy:91 into MCU-Boot Mode
1. Make sure the Thingy:91 is completely turned off.  
2. Connect the Micro-USB cable.  
3. Hold the middle button (SW3) on the Thingy:91 board.  
4. Switch on the power (SW1) to the Thingy:91.   
This will get the Thingy:91 into MCU-Boot Mode.    
### Flash the Thingy:91 board    
*(i hope there is a better way for flashing this)*   

To do this you first need to download the nRF Connect Desktop App [here](https://www.nordicsemi.com/Products/Development-tools/nRF-Connect-for-Desktop/Download?lang=en#infotabs), select "Linux" and download the AppImage.   
*(this has been tested on nRF Connect Desktop App Version 5.0.0)*   
Open the AppImage by typing in this command:  
```
exec ~/Downloads/nrfconnect-5.0.0-x86_64.appimage
```
After starting the app install the "Programmer" tool by clicking on "Install" next to the Programmer tool option. And click on "Open".   

**Make sure the board is in MCU-Boot Mode**  
 
1. Click on "Select Device" and select the Thingy:91 board.  
2. Click on "Add file" on the top left corner and navigate to the `build/` folder and select the `app_signed.hex`.    
For this example it is at `~/ncs/nrf/build/zephyr/app_signed.hex`.   
3. Finally press the "Write" button on the left side.  
This will flash the .hex file onto the Thingy:91 board.  
*(it might take some time)*  

## Working with the UDP 1NCE Zephyr blueprint  
### Setup the 1NCE Portal to allow communication  

1. Go to the [1NCE Portal](https://portal.1nce.com/portal/customer/dashboard) and navigate to the "1NCE OS" Tab.  
2. Click on "Device Integrator".  
3. Click on "New Integration" and choose UDP (for this example).  
4. Finally click on "Activate and use this protocol".  

Now you have setup your 1NCE Account to allow communication via UDP.  


### Obtaining an Access Token
To communicate with the 1NCE API resources you need to generate an access token. To do this go to this site [here](https://help.1nce.com/dev-hub/reference/postaccesstokenpost) on the right side you can put in your username and password which you should have from ordering the 1NCE SIM-Card kit. The username is your email address.   
Below that there will be a code snippet generated which will look something like this:  
```
curl --request POST \
     --url https://api.1nce.com/management-api/oauth/token \
     --header 'accept: application/json' \
     --header 'authorization: Basic dXNlcm5hbWU6cGFzc3dvcmQ=' \
     --header 'content-type: application/json' \
     --data '
{
  "grant_type": "client_credentials"
}
'
```
You can copy it and execute this command.  
This will give you an access token aka a Bearer Token. Copy the string after ("access_token":"< generated_token >") and save it somewhere.

### Getting the Device ID 
To get the Device ID aka the ICCID:  
1. Go to your 1NCE Portal [here](https://portal.1nce.com/portal/customer/dashboard).  
2. Click on "My Sims".  
3. Find the sim you want to use and copy the ICCID number.

## Sending a command to the board
To test the blueprint we can send a simple string to the board. To do this we can write a small script with `curl` in bash that does that.  
The script looks something like this:  
```
#!/bin/bash

# Replace <YOUR_TOKEN_HERE> with the generated token!
# Replace <YOUR_DEVICE_ID_HERE> with the ICCID of your SIM Card!

curl -X 'POST' 'https://api.1nce.com/management-api/v1/integrate/devices/<YOUR_DEVICE_ID_HERE/actions/UDP' -H 'accept: application/json' -H 'Authorization: Bearer <YOUR_TOKEN_HERE>' -H 'Content-Type: application/json' -d '{
  "payload": "This is a sample string.",
  "payloadType": "STRING",
  "port": 3000,
  "requestMode": "SEND_NOW"
}'
```
Save it, make it executable by doing `chmod +x script.sh` and run it.    
**Keep in mind that the access token expires in 3600 seconds (1 hour). After that you need to generate a new token.**  

## Looking at the output of the blueprint

*(todo)* 
