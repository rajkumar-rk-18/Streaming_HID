# Raspberry Pi Streaming + HID + Mass Storage Setup

## 1. Configure USB in Device Mode

Edit the boot configuration files:

### `/boot/firmware/config.txt`
```ini
[cm4]
# Enable host mode on the 2711 built-in XHCI USB controller.
# This line should be removed if the legacy DWC2 controller is required
# (e.g. for USB device mode) or if USB support is not required.
#otg_mode=1   --> Comment

[cm5]
#dtoverlay=dwc2,dr_mode=host   ---> Comment

[all]
dtoverlay=w1-gpio
enable_uart=1
dtoverlay=uart1
dtoverlay=disable-bt
dtoverlay=dwc2,dr_mode=peripheral   ---> Add
```

### `/boot/firmware/cmdline.txt`
Add `modules-load=dwc2` **after** `rootwait`.

Reboot after changes:
```bash
sudo reboot
```

---

## 2. Create Systemd Service Files

Navigate to:
```bash
cd /etc/systemd/system
```

Create and edit these services with `sudo nano`:

- `start_streaming.service`
- `streaming_hid.service`
- `usb_mass_storage.service`
- `composite-gadget.service`

Paste the respective service file contents and save.

---

## 3. Setup Composite Gadget Script

```bash
mkdir ~/gadgets
cd ~/gadgets
sudo nano setup_composite_gadget.sh
```

Paste the gadget script, then:
```bash
sudo chmod +x setup_composite_gadget.sh
```

---

## 4. Setup Streaming_HID

```bash
mkdir ~/Streaming_HID
cd ~/Streaming_HID

# Copy your application files:
#   app.py
#   start_streaming.py
#   templates/index.html

git clone https://github.com/pikvm/ustreamer.git
sudo apt update
sudo apt install -y build-essential libevent-dev libjpeg-dev libbsd-dev libv4l-dev git pkg-config

cd ustreamer
make
cd ..

python3 -m venv venv
source venv/bin/activate
pip install -r requirements.txt

# Test locally
python3 app.py
```

---

## 5. Setup OS_Flashing

```bash
mkdir ~/OS_Flashing
cd ~/OS_Flashing

# Copy your application files:
#   app.py
#   templates/index.html

python3 -m venv venv
source venv/bin/activate
pip install -r requirements.txt

# Test locally
sudo python3 app.py
```

---

## 6. Enable & Manage Services

Reload daemon and enable services:
```bash
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
```

Check status:
```bash
sudo systemctl status usb_mass_storage.service
sudo systemctl status start_streaming.service
sudo systemctl status streaming_hid.service
sudo systemctl status composite-gadget.service
```

View logs:
```bash
journalctl -u usb_mass_storage.service -f
journalctl -u start_streaming.service -f
journalctl -u streaming_hid.service -f
journalctl -u composite-gadget.service -f
```

---

## 7. Debugging & Verification

### Check USB Capture Card
```bash
for dev in /dev/video*; do
  echo -e "\n=== $dev ==="
  v4l2-ctl --device="$dev" --all 2>/dev/null | grep -E 'Driver name|Card type|Bus info'
done
```

### Check HID Gadget Binding
```bash
cat /sys/kernel/config/usb_gadget/composite_gadget/UDC
# Expected: fe980000.usb

ls /sys/kernel/config/usb_gadget/composite_gadget/configs/c.1/
# Expected: hid.usb0  hid.usb1  mass_storage.usb0

ls -l /dev/hidg*
# Expected:
# crw------- 1 root root 236, 0 Sep 19 15:15 /dev/hidg0
# crw------- 1 root root 236, 1 Sep 19 15:15 /dev/hidg1
```

---

✅ Your Raspberry Pi is now ready with **Streaming + HID Keyboard/Mouse + Mass Storage** composite gadget setup!
