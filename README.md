# Quick Guide: Installing Talos on Raspberry Pi 4 using macOS

This guide walks you through setting up **Talos Linux** on a **Raspberry Pi 4** from a **macOS host**, entirely headless (no monitor or keyboard required). Should work fine for other Raspberry Pi editions as well.

---

### Prerequisites
- macOS system with Homebrew installed  
- Raspberry Pi 4 (with Ethernet connection)  
- SD card (at least 8 GB)  
- USB card reader  
- Network with DHCP (so Pi gets an IP automatically)

---

### 1️⃣ Install `talosctl`

Talos provides a CLI for managing your nodes.

```bash
brew install talosctl
```

---

### 2️⃣ Update the EEPROM (Bootloader)

You’ll need to ensure your Pi’s EEPROM is updated to support **SD card boot**.

1. Open **Raspberry Pi Imager**
2. Choose the following:

   * **Device:** Raspberry Pi 4
   * **Operating System:** Misc Utility images → Bootloader (Pi 4 family) → **SD Card Boot**
   * **Storage:** Your SD card
3. Click **Write** and wait for it to complete.
4. Once done, remove and reinsert the SD card to confirm the EEPROM update completed successfully.

---

### 3️⃣ Download the Talos Image

Find the latest image for your architecture from the [Talos docs](https://docs.siderolabs.com/latest/).
As of writing:

```bash
curl -LO https://factory.talos.dev/image/ee21ef4a5ef808a9b7484cc0dda0f25075021691c8c09a276591eedb638ea1f9/v1.10.6/metal-arm64.raw.xz
xz -d metal-arm64.raw.xz
```

---

### 4️⃣ Write the Image to SD Card

Insert the SD card and identify it:

```bash
diskutil list | grep -i external
```

For example, it might show `/dev/disk5`.

Unmount the disk, then write the Talos image:

```bash
diskutil unmountDisk /dev/disk5
sudo dd if=metal-arm64.raw of=/dev/rdisk5 bs=4m conv=sync status=progress
diskutil eject /dev/disk5
```

> ⚠️ **Double-check the disk number** before running `dd` — writing to the wrong disk will erase your Mac’s data.

---

### 5️⃣ Connect Your Raspberry Pi

Insert the SD card into the Pi and plug it into Ethernet.
Wi-Fi is **not supported** during the initial Talos setup.

Power on the device and wait ~60 seconds for it to boot.

---

### 6️⃣ Bootstrap Talos

Once the Pi boots, get its IP address from your router or controller (e.g., Unifi).
In this example, assume it’s `192.168.1.227`.

Run the interactive setup:

```bash
talosctl apply-config -m interactive -n 192.168.1.227 --insecure
```

During the prompts:

* Choose **Control Plane node**
* Enable **Workload scheduling** if you only have a single Pi

Monitor boot logs:

```bash
talosctl dmesg -f -n 192.168.1.227
```

### 7️⃣ Access Kubernetes and the Talos Dashboard

Fetch the cluster’s kubeconfig:

```bash
talosctl kubeconfig -n 192.168.1.227 -f
```

Tail the dashboard for node status and health:

```bash
talosctl -n 192.168.1.227 dashboard
```

You can now use `kubectl` commands locally with your new cluster.

