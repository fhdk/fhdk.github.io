---
title: 'Manjaro Folding@Home'
date: '2025-09-22 08:22'
taxonomy:
    category:
        - docs
permissions:
    inherit: true
    authors:
        - linux-aarhus
---


## System

| AI |    |
|--- |---|
| engine | ollama |
| llm | gpt-oss:120b |
| prompt | How can I configure Manjaro Linux for use with Folding@Home using AMD W7500 |

| OS |  |
|--- | ---|
| os | Manjaro Linux |
| branch | unstable |
| kernel | Linux 6.16 |

| Hardware   | |
|--- | ---|
| GPU | Radeon 7900XTX |
| CPU | Ryzen 5945WX |
| RAM | 64G |

## DISCLAIMER - AI GENERATED CONTENT

Verify by crossreferencing the current Manjaro repo and [Folding@home - archlinux wiki](https://wiki.archlinux.org/title/Folding@home)

Below is a **step‑by‑step, copy‑and‑paste ready guide** for turning a fresh (or already‑running) Manjaro Linux installation into a **Folding@home (FAH) GPU workstation** that uses an **AMD Radeon W7500**.  
It covers everything you need to know:

* 1️⃣  Preparing the system – updates, kernel, user, firewall  
* 2️⃣  Installing the correct AMD OpenCL stack for the W7500 (open‑source + optional proprietary)  
* 3️⃣  Verifying OpenCL works (`clinfo`)  
* 4️⃣  Installing the FAH client (official package or from source)  
* 5️⃣  Configuring the FAH client for GPU‑only work, setting a custom team, passkey, power‑limit, etc.  
* 6️⃣  Managing the service (systemd) and monitoring progress (web UI, `FAHControl`, `FAHLog`)  
* 7️⃣  Tweaking performance / power‑saving options specific to the W7500  
* 8️⃣  Common pitfalls & troubleshooting checklist  

---

## 1️⃣  System preparation

> **Why?**  
> Manjaro ships with a rolling‑release kernel and a fairly recent Mesa stack, but we still want to make sure the kernel, firmware and user permissions are ready for AMD’s OpenCL drivers.

```bash
# 1. Refresh keyrings and update everything
sudo pacman -Syyu --noconfirm

# 2. Install the generic development tools (git, base-devel, etc.)
sudo pacman -S --needed base-devel git

# 3. Install a modern LTS kernel (optional, but recommended for better driver support)
#    If you already run the default kernel (e.g., 6.7.x) you can skip this.
sudo mhwd-kernel -i linux61  # or linux62, linux63 – whichever is current LTS in Manjaro repos

# 4. Reboot into the new kernel (or keep your current one if it works)
reboot
```

### Create a dedicated “folding” user (optional but clean)

```bash
sudo useradd -m -G video,wheel -s /bin/bash folding
# set a password (you’ll need it later)
sudo passwd folding
```

> **Tip:** Adding the user to the `video` group gives it permission to access the GPU device nodes (`/dev/dri/*`). If you run FAH under your regular user, just make sure that user is in `video`.

### Firewall (if you enable one)

FAH uses **port 80/443** for control, **port 7396** for the web UI and **port 36330** for the optional “slot” server. The default `ufw` rules are enough:

```bash
# If you have ufw enabled
sudo ufw allow 7396/tcp
sudo ufw allow 36330/tcp
sudo ufw reload
```

---

## 2️⃣  Install the AMD OpenCL stack for the **Radeon W7500**

The W7500 is a **Polaris‑based** GPU (GCN 4). It is fully supported by the **open‑source `amdgpu` driver + Mesa OpenCL** (`ROCm` is not required). However, you can also install the **AMDGPU‑PRO OpenCL** runtime for a modest performance boost.

### 2‑a. Open‑source stack (recommended)

```bash
# Core driver + firmware (Manjaro already ships this, but reinstall to be safe)
sudo pacman -S --needed xf86-video-amdgpu mesa opencl-mesa libclc

# Install the OpenCL ICD loader (already part of mesa-opencl, but keep it explicit)
sudo pacman -S --needed ocl-icd

# Verify the ICD file exists
cat /etc/OpenCL/vendors/mesa.icd
# → should output: libRusticlOpenCL.so.1
```

### 2‑b. (Optional) AMDGPU‑PRO OpenCL runtime
**UNVERIFIED** CHECK upstream on amd.com for the update drivers
If you want the proprietary `amdgpu-pro` OpenCL libraries (they can be ~5‑10 % faster on some kernels), do the following **once**. It will coexist with the open‑source stack – the ICD loader will pick the “pro” library if present.

```bash
# Install the helper script that pulls the latest Pro package from AMD
# (Works on Manjaro because we use the official .tar.xz release)
mkdir -p ~/amdpro && cd ~/amdpro
wget -O amdgpu-pro.tar.xz \
  https://drivers.amd.com/drivers/linux/amdgpu-pro-22.50-1290604-ubuntu-22.04.tar.xz
tar -xf amdgpu-pro.tar.xz
cd amdgpu-pro-22.50-1290604-ubuntu-22.04

# Install ONLY the OpenCL portion – we keep the open‑source driver
sudo ./amdgpu-install -y --opencl=legacy,pal --headless

# The installer puts an extra ICD entry:
cat /etc/OpenCL/vendors/amdgpu-pro.icd
# → should point to libamdocl64.so (or libamdocl64.so.1)
```

> **Important:** After installing the proprietary OpenCL, you must **re‑boot** so that the kernel module picks up the new firmware.

### 2‑c. Verify OpenCL works
**VERIFIED** USING W7100
```bash
# Install a tiny helper to query OpenCL devices
sudo pacman -S --needed clinfo

# Run it – you should see at least one “AMD Radeon W7500” device
clinfo | grep -i "AMD\|W7500"
```

Typical output (open‑source stack):

```
 $ clinfo | grep -i "AMD\|7100"
  Device Name                                     AMD Radeon Pro WX 7100 Graphics (radeonsi, polaris10, ACO, DRM 3.61, 6.12.44-3-MANJARO)
  Device Vendor                                   AMD
    Device Name                                   AMD Radeon Pro WX 7100 Graphics (radeonsi, polaris10, ACO, DRM 3.61, 6.12.44-3-MANJARO)
    Device Name                                   AMD Radeon Pro WX 7100 Graphics (radeonsi, polaris10, ACO, DRM 3.61, 6.12.44-3-MANJARO)
    Device Name                                   AMD Radeon Pro WX 7100 Graphics (radeonsi, polaris10, ACO, DRM 3.61, 6.12.44-3-MANJARO)
...
```

If you see **“Number of platforms = 0”**, double‑check that:

* the `amdgpu` kernel driver is loaded (`lsmod | grep amdgpu`)  
* the user is in the `video` group  
* `/dev/dri/renderD*` nodes exist and are readable/writable (`ls -l /dev/dri/`)

---

## 3️⃣  Install the Folding@home client

Build the FAH client (`foldingathome`) from AUR. AUR packages may cease to function on system updates. If that happens you need to rebuild your FAH client.

```bash
pamac build foldingathome
```

**Alternative – compile the very latest client** (useful if you want a bleeding‑edge patch)

```bash
cd ~
git clone https://github.com/FoldingAtHome/fah-client.git
cd fah-client
./configure && make -j$(nproc) && sudo make install
```


## 4️⃣  Configure the FAH client for GPU‑only work

### 4‑a. Generate a **basic configuration file**

The first time the client runs, it will create `~/.config/FoldingAtHome/FAHClient.cfg`.  
We will edit it manually to lock the client to the GPU slot.

```bash
# If you installed as a dedicated user, become that user
sudo -iu folding

# Run the client once to generate a default config (it will pause and ask for team/passkey)
fah-client &
sleep 5
kill %1   # stop the temporary instance
```

Now edit the config:

```bash
sudo micro ~/.config/FoldingAtHome/FAHClient.cfg
```

Paste the following (replace **TEAM**, **PASSKEY**, **POWER** as you wish):

```ini
#-------------------------------------------------
# Folding@home client configuration – GPU only
#-------------------------------------------------

# 1. Identity
user = folding          # any name you like
team = 0                # 0 = default team; replace with your own team number (e.g., 12345)
passkey =                # optional – put your personal passkey here if you have one

# 2. Slots – we only want a GPU slot
slot.0.type = GPU
slot.0.device = 0       # first GPU (0‑based). If you have multiple GPUs, you can add slot.1, slot.2, etc.
slot.0.power = 75       # % of max power the client may draw (default 100). 75% is a good compromise.
slot.0.enabled = true

# 3. Turn off CPU slots (optional but saves CPU cycles)
slot.1.type = CPU
slot.1.enabled = false

# 4. Network & security
web-allow = 7396        # port for the built‑in web UI (http://localhost:7396)
allow = 0/0/0           # allow all IPs (you can tighten this later)

# 5. Misc
gpu-allow = 0/0/0       # allow all GPUs – you can restrict by vendor if you ever add an NVIDIA card
pause-on-battery = false   # If you run a laptop – set true to avoid running on battery
```

**Save (`Ctrl+s`) and exit (`Ctrl+q`).**

### 4‑b. Optional – set a **custom “passkey”** (recommended for tracking)

If you have a personal Folding@home account, generate a **passkey** on the website (https://foldingathome.org/passkey) and paste it into the `passkey =` line above. This links the work units to your account and gives you credit on the leaderboard.

### 4‑c. Set **resource limits** for the W7500

The Radeon W7500 can draw **≈ 150 W** at full load. To keep the system cool and the power bill modest, you can:

* **Throttle via the client** (`slot.0.power = 70` → ~105 W).  
* **Set a kernel power profile** with `amdgpu` sysfs:

```bash
# As root
echo "low"   > /sys/class/drm/card0/device/power_dpm_force_performance_level
# or for a fine‑grained PWM limit:
echo 80      > /sys/class/drm/card0/device/pp_dpm_sclk
```

The **client’s `slot.0.power`** is the easiest way because it is respected automatically for each work unit.

---

## 5️⃣  Enable & start the service

When you build the client, a systemd unit called `fahclient.service` was created.

```bash
# Enable it for the dedicated folding user (or your own user)
sudo systemctl enable fahclient.service
sudo systemctl start fahclient.service

# Check status
sudo systemctl status fahclient.service
```

You should see something like:

```
● fahclient.service - Folding@home client
   Loaded: loaded (/usr/lib/systemd/system/fahclient.service; enabled; vendor preset: disabled)
   Active: active (running) since Fri 2025-09-20 …
 Main PID: 1234 (FAHClient)
    Tasks: 4 (limit: 4915)
   Memory: 150.0M
   CGroup: /system.slice/fahclient.service
           └─1234 /usr/bin/FAHClient -v
```

If you compiled the client yourself, you can create a simple service file:

```bash
sudo nano /etc/systemd/system/fahclient.service
```

```ini
[Unit]
Description=Folding@home client
After=network-online.target
Wants=network-online.target

[Service]
Type=simple
User=folding            # or your own user
ExecStart=/usr/local/bin/FAHClient -v
Restart=on-failure
RestartSec=10

[Install]
WantedBy=multi-user.target
```

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now fahclient.service
```

### Verify that the GPU slot is active

```bash
# The client logs to ~/.config/FoldingAtHome/FAHClient.log
tail -f ~/.config/FoldingAtHome/FAHClient.log
```

You should see lines like:

```
[INFO] slot 0 (GPU) is now working on core 2025-05-04 (Molecular Dynamics) on AMD Radeon W7500
[INFO] slot 0: using 75% power limit
```

If you see `slot 0: no OpenCL platform found`, double‑check the OpenCL setup (section 2‑c).

---

## 6️⃣  Monitoring & controlling your folding node

| Tool | How to use | What you see |
|------|------------|--------------|
| **Web UI** (built‑in) | Open `http://localhost:7396` in a browser (or `http://<IP>:7396` from another machine) | Real‑time work unit progress, GPU temperature, power usage, credit earned |
| **FAHControl** (GUI) | `sudo pacman -S fahcontrol` then launch `FAHControl` | Same as web UI, but with a desktop app; can change slots on‑the‑fly |
| **FAHLog** (CLI) | `FAHLog -t` (tail) | Live log output (same as `tail -f` on the log file) |
| **clinfo** | `clinfo` | Confirms OpenCL platform/device details – handy for debugging driver issues |
| **systemd** | `systemctl status fahclient` / `journalctl -u fahclient -f` | Service health, restarts, crash logs |

**Tip:** If you run the node headless (no monitor), you can tunnel the web UI over SSH:

```bash
ssh -L 7396:localhost:7396 folding@your-node-ip
# then point your local browser to http://localhost:7396
```

---

## 7️⃣  Performance‑tuning specific to the Radeon W7500

| Setting | Command | Effect | Recommended value |
|---------|---------|--------|-------------------|
| **Client power limit** | `slot.0.power = 70` (in config) | Caps GPU power, reduces heat & fan noise | 70 % for 24/7 home use; 85 % if you need more credit |
| **OpenCL work‑size** | `slot.0.gpu-allow = 0/0/0` (default) – you can add `slot.0.gpu-allow = 0/0/256` to limit work‑group size if you see instability | Some kernels mis‑behave on older Polaris chips at large work‑groups | Not needed for most users; leave default |
| **Kernel performance level** | `echo high > /sys/class/drm/card0/device/power_dpm_force_performance_level` (root) | Forces the GPU to stay in “high” performance state, slightly better throughput but more heat | Use **high** if you notice the GPU dropping to low power frequently (check `dmesg | grep amdgpu`) |
| **Fan control** | Install `amdgpu-fan` from AUR (`yay -S amdgpu-fan`) and set a custom curve (e.g., 60 % at 50 °C) | Keeps temps under control without constant full‑speed fan | 40‑60 % fan speed is usually fine for 70 % power limit |
| **CPU affinity** (if you keep a tiny CPU slot) | `taskset -c 0-3 FAHClient` in a wrapper script | Keeps the client from stealing cores you might need for other tasks | Not required when CPU slot disabled |

### Example: Adding a fan curve (optional)

```bash
# Install the helper
yay -S amdgpu-fan

# Create a simple profile (e.g., 30% at 40°C, 60% at 55°C, 100% at 70°C)
sudo tee /etc/amdgpu-fan.conf > /dev/null <<'EOF'
[fan0]
temp=40,30
temp=55,60
temp=70,100
EOF

# Enable the daemon
sudo systemctl enable --now amdgpu-fan.service
```

---

## 8️⃣  Common pitfalls & quick‑fix checklist

| Symptom | Likely cause | Fix |
|---------|--------------|-----|
| **`FAHClient` exits with “no OpenCL platform found”** | - User not in `video` group <br> - `amdgpu` driver not loaded <br> - Wrong ICD file order (Pro overrides open‑source) | `sudo usermod -aG video $USER && newgrp video` <br> Verify `lsmod | grep amdgpu` <br> Ensure `/etc/OpenCL/vendors/mesa.icd` exists and `amdgpu-pro.icd` (if installed) points to a real lib |
| **GPU shows 0 % utilization in `clinfo` / FAH UI** | - Power limit set to 0 (or `slot.0.power = 0`) <br> - GPU in “low” performance mode due to thermal throttling | Set `slot.0.power` to a non‑zero value (≥ 50) <br> Run `cat /sys/class/drm/card0/device/pp_dpm_sclk` to see if clocks are stuck; if so, `echo high > /sys/.../power_dpm_force_performance_level` |
| **FAH client crashes with “Segmentation fault (core dumped)”** | - Mismatch between Mesa OpenCL version and AMDGPU‑PRO libs <br> - Out‑of‑date kernel (older kernels have bugs with Polaris) | Remove the proprietary ICD (`sudo rm /etc/OpenCL/vendors/amdgpu-pro.icd`) and stick to the open‑source stack, OR update to the latest kernel (`mhwd-kernel -li` → install newest) |
| **System hangs when folding starts** | - Insufficient power supply (W7500 draws ~150 W) <br> - Overheating (GPU fan stuck) | Verify PSU rating (> 400 W) <br> Check temps with `sensors` (install `lm_sensors`) <br> Clean dust, ensure fan spins |
| **FAH shows “No credit earned” despite work** | - You forgot to add your **passkey** or **team** <br> - You are using the default team 0 (which still gives credit, but you may be confused) | Add your personal passkey in `FAHClient.cfg` <br> Or join a team and set `team = <your‑team‑id>` |
| **Web UI unreachable from another machine** | - Firewall blocking 7396 <br> - `allow` setting restricts IPs | `sudo ufw allow 7396/tcp` <br> Set `allow = 0/0/0` in the config or specify `allow = 192.168.1.0/24/0` |

---

## 📦  Full “one‑shot” script (optional)

If you want to automate everything (except your personal team/passkey), copy‑paste the following script **as root** (or use `sudo -i` first). It will:

1. Update the system  
2. Install the open‑source AMDGPU + OpenCL stack  
3. Install FAH client  
4. Create a `folding` user (or use the current user)  
5. Write a ready‑to‑run config with a 70 % power limit  

```bash
#!/usr/bin/env bash
set -euo pipefail

# -------------------------------------------------
# 1️⃣  System update
# -------------------------------------------------
pacman -Syyu --noconfirm

# -------------------------------------------------
# 2️⃣  Install required packages
# -------------------------------------------------
pacman -S --needed base-devel git xf86-video-amdgpu mesa mesa-opencl libclc ocl-icd clinfo folding-at-home

# -------------------------------------------------
# 3️⃣  (Optional) create dedicated user
# -------------------------------------------------
if ! id -u folding &>/dev/null; then
    useradd -m -G video,wheel -s /bin/bash folding
    echo "Created user 'folding' (set a password when prompted)"
    passwd folding
fi

# -------------------------------------------------
# 4️⃣  Verify OpenCL
# -------------------------------------------------
if ! clinfo | grep -iq "AMD Radeon W7500"; then
    echo "⚠️  OpenCL detection failed. Please reboot and ensure the amdgpu driver loads."
    exit 1
fi

# -------------------------------------------------
# 5️⃣  Write config (for the chosen user)
# -------------------------------------------------
TARGET_USER=${SUDO_USER:-$(whoami)}
HOME_DIR=$(eval echo "~$TARGET_USER")
CONFIG_DIR="$HOME_DIR/.config/FoldingAtHome"
mkdir -p "$CONFIG_DIR"

cat > "$CONFIG_DIR/FAHClient.cfg" <<'EOF'
# Folding@home client config – GPU only (W7500)
user = folding
team = 0          # replace with your team number if you have one
passkey =        # optional – paste your personal passkey here

slot.0.type = GPU
slot.0.device = 0
slot.0.power = 70
slot.0.enabled = true

slot.1.type = CPU
slot.1.enabled = false

web-allow = 7396
allow = 0/0/0
gpu-allow = 0/0/0
pause-on-battery = false
EOF

chown -R "$TARGET_USER":"$TARGET_USER" "$CONFIG_DIR"

# -------------------------------------------------
# 6️⃣  Enable & start the service
# -------------------------------------------------
systemctl enable --now fahclient.service

echo "✅  Setup complete! Check the log at $CONFIG_DIR/FAHClient.log"
echo "🌐  Open http://$(hostname -I | awk '{print $1}'):7396 to watch your folding progress."
```

Run it with:

```bash
chmod +x setup_fah_w7500.sh
sudo ./setup_fah_w7500.sh
```

After the reboot (or after the script finishes), **open the web UI** and you should see the GPU slot actively folding.

---

## 🎉  You’re done!

Your Manjaro box with a Radeon W7500 is now **contributing to the world‑wide Folding@home project**.  

* **Check the web UI** regularly to see work‑unit progress and credit.  
* **Keep the system updated** (`sudo pacman -Syu`) – newer kernels and Mesa releases often bring modest OpenCL speed gains.  
* **If you ever add another GPU** (e.g., an NVIDIA RTX), just add another `slot.N` block in the same config and the client will automatically use both.

Happy folding, and thank you for donating your GPU cycles to science! 🚀🧬