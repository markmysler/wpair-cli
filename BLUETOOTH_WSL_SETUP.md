# Bluetooth Setup for WSL2 - Testing WPair

This guide shows how to pass Bluetooth hardware from Windows to WSL2 so you can test the WPair vulnerability scanner.

## Prerequisites

- Windows 10/11 with WSL2
- Bluetooth adapter (built-in or USB)
- Administrator access on Windows

## Method 1: USB/IP Passthrough (Recommended)

This is the official Microsoft-supported method for passing USB devices to WSL2.

### Step 1: Install usbipd-win on Windows

Open PowerShell as Administrator and run:

```powershell
winget install --interactive --exact dorssel.usbipd-win
```

Or download from: https://github.com/dorssel/usbipd-win/releases

### Step 2: Find Your Bluetooth Adapter

In PowerShell (Administrator), list all USB devices:

```powershell
usbipd list
```

Look for your Bluetooth adapter. Example output:
```
BUSID  VID:PID    DEVICE
1-2    8087:0026  Intel(R) Wireless Bluetooth(R)
2-3    0bda:8153  Realtek USB GbE Family Controller
```

Your Bluetooth adapter might be named:
- Intel Wireless Bluetooth
- Broadcom Bluetooth
- Realtek Bluetooth
- Generic Bluetooth Adapter

**Note the BUSID** (e.g., `1-2`)

### Step 3: Bind the Bluetooth Device

Still in PowerShell (Administrator):

```powershell
usbipd bind --busid 1-2
```

Replace `1-2` with your actual BUSID. This only needs to be done once.

### Step 4: Attach to WSL

Every time you want to use Bluetooth in WSL:

```powershell
usbipd attach --wsl --busid 1-2
```

**Keep PowerShell window open** while using Bluetooth in WSL.

### Step 5: Setup WSL Side

Open your WSL terminal and install required packages:

```bash
# Update package lists
sudo apt update

# Install USB/IP tools
sudo apt install linux-tools-generic hwdata usbutils

# Verify Bluetooth adapter is visible
lsusb | grep -i bluetooth
```

You should see your Bluetooth adapter listed.

### Step 6: Install BlueZ (Bluetooth Stack)

```bash
# Install Bluetooth tools and daemon
sudo apt install bluez bluez-tools bluetooth

# Start Bluetooth service
sudo systemctl start bluetooth

# Enable Bluetooth service (optional - auto-start)
sudo systemctl enable bluetooth

# Check status
systemctl status bluetooth
```

You should see "active (running)" in green.

### Step 7: Verify Bluetooth Works

```bash
# Check Bluetooth controller
hciconfig -a

# Should show something like:
# hci0:   Type: Primary  Bus: USB
#         BD Address: XX:XX:XX:XX:XX:XX  ACL MTU: 1021:8  SCO MTU: 64:1

# Power on Bluetooth (if needed)
sudo hciconfig hci0 up

# Scan for nearby Bluetooth devices (test)
sudo timeout 10s hcitool lescan
```

If you see devices listed, Bluetooth is working!

### Step 8: Test WPair

```bash
cd /home/mark/wpair-app/wpair-cli

# Scan for Fast Pair devices
sudo python3 -m wpair scan --timeout 10

# Test a specific device (replace with real address)
sudo python3 -m wpair test AA:BB:CC:DD:EE:FF
```

**Note**: You need `sudo` because Bluetooth operations require root privileges.

## Method 2: WSL1 (Alternative)

WSL1 has more direct hardware access but is slower overall.

### Convert WSL to Version 1

In Windows PowerShell:

```powershell
# Check current version
wsl -l -v

# Convert to WSL1
wsl --set-version Ubuntu 1

# Or set default to WSL1 for new distros
wsl --set-default-version 1
```

Then follow steps 5-8 from Method 1.

### Convert Back to WSL2

```powershell
wsl --set-version Ubuntu 2
```

## Troubleshooting

### "usbipd: command not found"

Make sure you installed usbipd-win and restarted PowerShell.

### "Device is already attached"

Detach first:
```powershell
usbipd detach --busid 1-2
```

Then re-attach.

### "No Bluetooth adapter found in WSL"

1. Check if attached: `lsusb | grep -i bluetooth`
2. Re-attach from Windows: `usbipd attach --wsl --busid 1-2`
3. Restart Bluetooth: `sudo systemctl restart bluetooth`

### "Operation not permitted" when scanning

You need root privileges:
```bash
sudo python3 -m wpair scan --timeout 10
```

Or add your user to bluetooth group:
```bash
sudo usermod -aG bluetooth $USER
# Log out and back in
```

### BlueZ service won't start

```bash
# Check for errors
sudo journalctl -u bluetooth

# Manually start
sudo /usr/lib/bluetooth/bluetoothd

# Check if process is running
ps aux | grep bluetoothd
```

### Bluetooth works but WPair finds no devices

This is normal if:
- No Fast Pair devices are nearby
- Devices are already paired to another phone
- Devices are in case/off

Try putting a device (like Google Pixel Buds) in pairing mode.

## Permissions Setup (Optional)

To avoid using `sudo` every time:

```bash
# Add your user to bluetooth group
sudo usermod -aG bluetooth $USER

# Configure permissions for BlueZ
sudo tee /etc/dbus-1/system.d/bluetooth-permissions.conf <<EOF
<!DOCTYPE busconfig PUBLIC "-//freedesktop//DTD D-BUS Bus Configuration 1.0//EN"
 "http://www.freedesktop.org/standards/dbus/1.0/busconfig.dtd">
<busconfig>
  <policy group="bluetooth">
    <allow send_destination="org.bluez"/>
  </policy>
</busconfig>
EOF

# Restart D-Bus and Bluetooth
sudo systemctl restart dbus
sudo systemctl restart bluetooth

# Log out and back in for group membership to take effect
```

Then you can run:
```bash
python3 -m wpair scan --timeout 10  # No sudo needed
```

## Detaching Bluetooth (When Done)

To return Bluetooth control to Windows:

```powershell
# In PowerShell (Administrator)
usbipd detach --busid 1-2
```

Bluetooth will work in Windows again.

## Automatic Reattach Script (Windows)

Save this as `attach-bluetooth.ps1`:

```powershell
# Run as Administrator
$busid = "1-2"  # Change to your BUSID

Write-Host "Attaching Bluetooth ($busid) to WSL..."
usbipd attach --wsl --busid $busid

if ($?) {
    Write-Host "✓ Bluetooth attached successfully!" -ForegroundColor Green
    Write-Host "Run in WSL: sudo systemctl restart bluetooth" -ForegroundColor Yellow
} else {
    Write-Host "✗ Failed to attach Bluetooth" -ForegroundColor Red
}

# Keep window open
Write-Host "`nPress Ctrl+C to detach and exit..."
while ($true) {
    Start-Sleep -Seconds 1
}
```

Run from PowerShell (Administrator):
```powershell
.\attach-bluetooth.ps1
```

## Testing Workflow

1. **Windows**: Attach Bluetooth
   ```powershell
   usbipd attach --wsl --busid 1-2
   ```

2. **WSL**: Start Bluetooth service
   ```bash
   sudo systemctl restart bluetooth
   ```

3. **WSL**: Run WPair
   ```bash
   cd /home/mark/wpair-app/wpair-cli
   sudo python3 -m wpair scan --timeout 30
   ```

4. **WSL**: Test a device
   ```bash
   sudo python3 -m wpair test <MAC_ADDRESS>
   ```

5. **Windows**: Detach when done
   ```powershell
   usbipd detach --busid 1-2
   ```

## Security Notes

- WPair requires Bluetooth access (obviously)
- Only test devices you own
- Use `--confirm` flag for exploit command (as required)
- Read legal disclaimers in README.md

## Alternative: Native Linux

For best Bluetooth compatibility, consider:
- Dual-boot Linux
- Linux VM with USB passthrough
- Dedicated Linux machine
- Live USB boot (Ubuntu, Kali)

WSL2 Bluetooth support is improving but can be finicky.

## References

- [usbipd-win GitHub](https://github.com/dorssel/usbipd-win)
- [WSL USB Support Docs](https://learn.microsoft.com/en-us/windows/wsl/connect-usb)
- [BlueZ Documentation](http://www.bluez.org/)

---

**Good luck testing WPair!** If you have issues, check the troubleshooting section or open a GitHub issue.
