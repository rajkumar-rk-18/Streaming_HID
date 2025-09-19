sudo nano /boot/firmware/config.txt 


[cm4]
# Enable host mode on the 2711 built-in XHCI USB controller.
# This line should be removed if the legacy DWC2 controller is required
# (e.g. for USB device mode) or if USB support is not required.
#otg_mode=1   ---> Comment
                                            
[cm5]
#dtoverlay=dwc2,dr_mode=host   ---> Comment
                                            
[all]
dtoverlay=w1-gpio
enable_uart=1
dtoverlay=uart1
dtoverlay=disable-bt
dtoverlay=dwc2,dr_mode=peripheral ---> Add

sudo nano /boot/firmware/cmdline.txt      -> Add "modules-load=dwc2" after rootwait

After above two steps are done --> sudo reboot
===========================================================================================================================
Connect again

cd /etc/systemd/system

sudo nano start_streaming.service       -> (Copy the content and save & exit (Ctrl +X))

sudo nano streaming_hid.service         -> (Copy the content and save & exit (Ctrl +X))

sudo nano usb_mass_storage.service      -> (Copy the content and save & exit (Ctrl +X))

sudo nano composite-gadget.service      -> (Copy the content and save & exit (Ctrl +X))
====================================================================================================
In rpi root folder

mkdir gadgets
cd gadgets
sudo nano setup_composite_gadget.sh             -> (Copy and save the script)
sudo chmod +x setup_composite_gadget.sh

====================================================================================================

mkdir Streaming_HID
cd Streaming_HID
sudo nano app.py      			      -> (Copy the code and save)
sudo nano start_streaming.py      -> (Copy the code and save)
mkdir templates
cd templates
sudo nano index.html              -> (Copy the code and save)
cd ../                            -> (Come back to Streaming_HID folder)
git clone https://github.com/pikvm/ustreamer.git
sudo apt update
sudo apt install -y build-essential libevent-dev libjpeg-dev libbsd-dev libv4l-dev git pkg-config
cd ustreamer
make
cd ../                            -> (Come back to Streaming_HID)
python3 -m venv venv
source venv/bin/activate
pip install -r requirements.txt

python3 app.py (Test locally for installed dependencies)
=====================================================================================================

Comeback to rpi root

mkdir OS_Flashing
cd OS_Flashing
sudo nano app.py                    -> (Copy the code and save)
mkdir templates
cd templates
sudo nano index.html                -> (Copy and save)
cd ../                              -> (Come back to OS_Flashing)
python3 -m venv venv
source venv/bin/activate
pip install -r requirements.txt

sudo python3 app.py (Test locally for installed dependencies)
======================================================================================================

Service Commands:


sudo systemctl daemon-reexec
sudo systemctl daemon-reload
sudo systemctl enable usb_mass_storage.service
sudo systemctl enable start_streaming.service
sudo systemctl enable streaming_hid.service
sudo systemctl enable composite-gadget.service

sudo systemctl start usb_mass_storage.service
sudo systemctl start start_streaming.service
sudo systemctl start streaming_hid.service
sudo systemctl start composite-gadget.service

sudo systemctl status usb_mass_storage.service
sudo systemctl status start_streaming.service
sudo systemctl status streaming_hid.service
sudo systemctl status composite-gadget.service

journalctl -u usb_mass_storage.service -f
journalctl -u start_streaming.service -f
journalctl -u streaming_hid.service -f
journalctl -u composite-gadget.service -f

======================================================================================================
To check usb capture card: It will list the connected capture cards as usb.

for dev in /dev/video*; do   echo -e "\n=== $dev ===";   v4l2-ctl --device="$dev" --all 2>/dev/null | grep -E 'Driver name|Card type|Bus info'; done

Commands to check hid connections:

cat /sys/kernel/config/usb_gadget/composite_gadget/UDC  -> Expected = "fe980000.usb"
ls /sys/kernel/config/usb_gadget/composite_gadget/configs/c.1/  -> Expected = "hid.usb0  hid.usb1  mass_storage.usb0"
ls -l /dev/hidg*   -> Expected output = "crw------- 1 root root 236, 0 Sep 19 15:15 /dev/hidg0
                                         crw------- 1 root root 236, 1 Sep 19 15:15 /dev/hidg1"
										 
===================================================================================================


