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



