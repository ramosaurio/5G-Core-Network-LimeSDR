# 5G SA Network with srsRAN Project and LimeSDR Mini

## 🧩 Abstract

This repository provides a reproducible setup for deploying a standalone 5G (SA) network using a **customized version of the srsRAN Project** (for the gNB) in combination with **Open5GS** as the 5G Core. The gNB implementation has been patched to enable **SoapySDR compatibility via the UHD backend**, allowing seamless integration with the **LimeSDR Mini**.

This setup is ideal for educational, development, and research use cases focused on private 5G networks and programmable SIMs. It provides a clean and modular separation between the radio access network (gNB) and the core network (Open5GS), while supporting operation on modest hardware for experimentation in local environments.

The objective is to offer a practical 5G SA deployment that combines the flexibility of open-source tools with enhanced SDR support, making it suitable for both protocol-level analysis and real-world mobile connectivity testing.

## 📂 Structure

```plaintext
/
├── README.md               ← This documentation
└── submodules/             ← Forked Git submodules for reproducibility
    ├── srsRAN_Project/     ← Forked srsRAN 5G (specific commit)
    ├── SoapySDR/           ← Forked SoapySDR (specific commit)
    ├── uhd/                ← Forked uhd (master)
    ├── SoapyUHD/           ← Forked SoapyUHD (master)
    ├── LimeSuite/          ← Forked LimeSuite (specific commit)
    ├── docker_open5gs/     ← Forked docker_open5gs (master)
    └── LimeSuite/          ← Forked LimeSuite (specific commit)
```

## 🧩 Software and Versions Used

| Software / Library         | Version Used / Recommended   | Purpose                                                         | Source / Notes                                         |
|---------------------------|------------------------------|-----------------------------------------------------------------|--------------------------------------------------------|
| **srsRAN Project**        | `release_limesdr_24_10_1`    | 5G stack: gNB, 5GC (AMF, SMF, UPF), CU/DU split                 | [GitHub](https://github.com/ramosaurio/srsRAN_Project) |
| **LimeSuite**             | `v22.09.1`                   | SDR driver and utilities for LimeSDR hardware                   | [GitHub](https://github.com/myriadrf/LimeSuite)        |
| **SoapySDR**              | `soapy-sdr-0.8.1`            | SDR abstraction layer                                           | [GitHub](https://github.com/pothosware/SoapySDR)       |
| **UHD**                   | `master`                     | Driver and tools for USRP SDR devices                           | [GitHub](https://github.com/EttusResearch/uhd)         |
| **SoapyUHD**              | `master`                     | SoapySDR plugin for UHD-supported devices                       | [GitHub](https://github.com/pothosware/SoapyUHD)       |
| **docker_open5gs**        | `master`                     | Dockerized 5GC (Open5GS core) with Kamailio support             | [GitHub](https://github.com/herlesupreeth/docker_open5gs.git) |
| **Python**                | `3.9+`                       | Required for scripts and SIM provisioning tools                 | Bundled with Ubuntu 20.04+                             |
| **CMake**                 | `3.16+`                      | Build system for C++ components                                 | `sudo apt install cmake`                               |
| **GCC / G++**             | `>=9.x`                      | C/C++ compiler                                                  | Default in Ubuntu 20.04 / Debian 11                    |
| **libboost-dev**          | `>=1.71`                     | Dependency for various components                               | `sudo apt install libboost-all-dev`                    |
| **libconfig++-dev**       | `1.5.x` or compatible        | Configuration parser                                            | `sudo apt install libconfig++-dev`                     |

## 💻 Minimal Hardware Requirements

- **CPU**: Quad-core Intel Core I7 5ª Gen or better 
- **RAM**: 8GB minimum 
- **Disk**: 100GB SSD recommended
- **SDR**: LimeSDR Mini (tested)
- **OS**: Ubuntu 20.04 (others may work with adaptation)
# 📡 RAN Setup: srsRAN 5G + LimeSDR + UHD + SoapySDR

This section describes how to set up the RAN (Radio Access Network) for a 5G SA deployment using `srsRAN_Project`, `LimeSDR`, and the UHD/SoapySDR ecosystem.

> 💡 This setup assumes you're working from a forked project with the following directory structure under `submodules/`, with each submodule fixed to a known working commit or tag for reproducibility.

## 📁 Project structure

```
submodules/
├── srsRAN_Project/     ← Forked srsRAN 5G (tag: `release_limesdr_24_10_1`)
├── SoapySDR/           ← Forked SoapySDR
├── uhd/                ← Forked UHD (build from `host/`)
├── SoapyUHD/           ← Forked SoapyUHD
├── LimeSuite/          ← Forked LimeSuite
├── docker_open5gs/     ← Core network with Docker
```

All components must be built and installed under a local prefix to keep the environment isolated.

---

## ⚙️ Build and install RAN components

```bash
# Define the local install path
git clone --recurse-submodules git@github.com:ramosaurio/5G-Core-Network-LimeSDR.git
export SRSRAN_INSTALL=$HOME/ran-5g
mkdir -p "$SRSRAN_INSTALL"
export LD_LIBRARY_PATH=${SRSRAN_INSTALL}/lib
export PATH=${SRSRAN_INSTALL}/bin:$PATH
```

### 1. Build and install SoapySDR
📖 [Reference: SoapySDR Wiki](https://github.com/pothosware/SoapySDR/wiki)

```bash
sudo apt install g++ cmake libpython3-dev python3-numpy swig
cd submodules/SoapySDR
mkdir build && cd build
cmake -DCMAKE_PREFIX_PATH=$SRSRAN_INSTALL -DCMAKE_INSTALL_PREFIX=$SRSRAN_INSTALL ..
make -j$(nproc)
make install
```

### 2. Build and install LimeSuite
📖 [Reference: SoapySDR Wiki](https://github.com/pothosware/SoapySDR/wiki)

```bash
sudo apt-get install libi2c-dev libusb-1.0-0-dev
cd submodules/LimeSuite
mkdir builddir && cd builddir
cmake -DCMAKE_PREFIX_PATH=$SRSRAN_INSTALL -DCMAKE_INSTALL_PREFIX=$SRSRAN_INSTALL ..
make -j$(nproc)
make install
cd ../udev-rules
sudo bash install.sh
```


### 3. Build and install UHD
📖 [Reference: UHD Wiki](https://github.com/EttusResearch/uhd/wiki)

> ⚠️ You **must** build UHD from the `host/` directory.

```bash
sudo apt install libboost-all-dev python3-setuptools
cd submodules/uhd/host
mkdir build && cd build
cmake -DCMAKE_PREFIX_PATH=$SRSRAN_INSTALL -DCMAKE_INSTALL_PREFIX=$SRSRAN_INSTALL ..
make -j$(nproc)
make install
```

### 4. Build and install SoapyUHD
📖 [Reference: SoapyUHD Wiki](https://github.com/pothosware/SoapyUHD/wiki)

```bash
cd submodules/SoapyUHD
mkdir build && cd build
cmake -DCMAKE_PREFIX_PATH=$SRSRAN_INSTALL -DCMAKE_INSTALL_PREFIX=$SRSRAN_INSTALL ..
make -j$(nproc)
make install
```


### 5. Build and install srsRAN_Project
📖 [Reference: srsRAN Installation Guide](https://docs.srsran.com/projects/project/en/latest/user_manuals/source/installation.html)

> 📌 Make sure you're on tag `release_limesdr_24_10_1`. This is already set in the submodule but it's good practice to verify:

```bash
sudo apt install libmbedtls-dev libfftw3-dev libgtest-dev libyaml-cpp-dev libsctp-dev lksctp-tools
cd submodules/srsRAN_Project
mkdir build && cd build
cmake -DCMAKE_PREFIX_PATH=$SRSRAN_INSTALL \
      -DCMAKE_INSTALL_PREFIX=$SRSRAN_INSTALL \
      -DCMAKE_CXX_FLAGS="-Wno-error=unused-result -Wno-error=maybe-uninitialized -Wno-error=stringop-overflow" \
      ..
make -j$(nproc)
make install
```

---

## ✅ Verifying the RAN installation

### 🔎 Check SoapySDR detection

Make sure the LimeSDR is connected and run:

```bash
$SRSRAN_INSTALL/bin/SoapySDRUtil --info
```

Expected output:

```
######################################################
##     Soapy SDR -- the SDR abstraction library     ##
######################################################
Lib Version: v0.8.1-g1cf5a539
API Version: v0.8.0
ABI Version: v0.8
Install root: /home/user/srsRAN
Search path:  /home/user/srsRAN/lib/SoapySDR/modules0.8 (missing)
No modules found!
Available factories... No factories found!
Available converters...
 -  CF32 -> [CF32, CS16, CS8, CU16, CU8]
 -  CS16 -> [CF32, CS16, CS8, CU16, CU8]
 -  CS32 -> [CS32]
 -   CS8 -> [CF32, CS16, CS8, CU16, CU8]
 -  CU16 -> [CF32, CS16, CS8]
 -   CU8 -> [CF32, CS16, CS8]
 -   F32 -> [F32, S16, S8, U16, U8]
 -   S16 -> [F32, S16, S8, U16, U8]
 -   S32 -> [S32]
 -    S8 -> [F32, S16, S8, U16, U8]
 -   U16 -> [F32, S16, S8]
 -    U8 -> [F32, S16, S8]
LimeSuite
```


then SoapySDR could not load the LimeSDR backend. Double-check your LimeSuite build and install paths.

---

### 🔎 Check UHD device detection

Run:

```bash
$SRSRAN_INSTALL/bin/uhd_find_devices
```

Expected output:

```
[INFO] [UHD] linux; ...
--------------------------------------------------
-- UHD Device 0
--------------------------------------------------
Device Address:
    serial: 1DA160BEF543E4
    driver: lime
    label: LimeSDR Mini [USB 2.0] 1DA160BEF543E4
    media: USB 2.0
    module: FT601
    name: LimeSDR Mini
    type: soapy
```

❗If the device is not detected or `type: soapy` is missing, the UHD backend is not correctly linked to Soapy. Verify your `SoapyUHD` installation.

---

## ⚙️ Build and install CORE Network components
> ⚠️ **Important note:** All components will be deployed on a **single host**. Although a multi-host architecture would be possible, it is not being used in this setup because there is no external clock source available. The system will rely on the **internal clock of the SDR**, which limits synchronization accuracy across multiple machines. Therefore, a monolithic deployment is chosen for this test environment.

### 1. Install Docker
📖 [Reference: Docker Docs](https://docs.docker.com/engine/install/ubuntu/)

```bash
sudo apt install apt-transport-https ca-certificates curl software-properties-common
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu focal stable"
sudo apt update
apt-cache policy docker-ce
sudo apt install docker-ce
sudo usermod -aG docker ${USER}
```

### 2. Pull Images
📖 [Reference: Forked Open5gs Docker Documentation](https://github.com/ramosaurio/docker_open5gs)

Pull base images: 
```bash
docker pull ghcr.io/herlesupreeth/docker_open5gs:master
docker tag ghcr.io/herlesupreeth/docker_open5gs:master docker_open5gs

docker pull ghcr.io/herlesupreeth/docker_grafana:master
docker tag ghcr.io/herlesupreeth/docker_grafana:master docker_grafana

docker pull ghcr.io/herlesupreeth/docker_metrics:master
docker tag ghcr.io/herlesupreeth/docker_metrics:master docker_metrics
```

Pull IMSI components:
```bash
docker pull ghcr.io/herlesupreeth/docker_osmohlr:master
docker tag ghcr.io/herlesupreeth/docker_osmohlr:master docker_osmohlr

docker pull ghcr.io/herlesupreeth/docker_osmomsc:master
docker tag ghcr.io/herlesupreeth/docker_osmomsc:master docker_osmomsc

docker pull ghcr.io/herlesupreeth/docker_pyhss:master
docker tag ghcr.io/herlesupreeth/docker_pyhss:master docker_pyhss

docker pull ghcr.io/herlesupreeth/docker_kamailio:master
docker tag ghcr.io/herlesupreeth/docker_kamailio:master docker_kamailio

docker pull ghcr.io/herlesupreeth/docker_mysql:master
docker tag ghcr.io/herlesupreeth/docker_mysql:master docker_mysql
```

### 2. Deploy Core
📖 [Reference: Forked Open5gs Docker Documentation](https://github.com/ramosaurio/docker_open5gs)

Pull base images:
```bash
docker compose -f sa-vonr-deploy.yaml up
```

## ✅ Verifying the Open5Gs deployment

### 🔎 Check Open5Gs deployemtn

To verify that Open5GS has been installed correctly, open your web browser and go to:

http://{HOST-IP}:9999/

Use the following login credentials:

Username: admin
Password: 1423

If the web interface loads and you can log in successfully, the installation has been completed properly.

---

## 🛠️ Troubleshooting

### 1. LimeSDR Mini is not natively supported by UHD

The 5G gNB implementation in `srsRAN_Project` currently supports **UHD** as its only SDR backend. However, the **LimeSDR Mini** is not directly compatible with UHD drivers, which causes runtime errors such as:

To bridge this incompatibility, this setup uses **[SoapyUHD](https://github.com/pothosware/SoapyUHD)**, a middleware that enables SDR devices supported by **SoapySDR** (like LimeSDR) to appear as UHD-compatible devices. This allows the `srsran_gnb` binary to interface with the LimeSDR Mini through UHD, even though UHD does not support it natively.

### 2. Patch required in srsRAN gNB source code

Even when using SoapyUHD, the original srsRAN gNB expects to configure `time_source` and `clock_source`, which are not supported or necessary in this context, since SoapySDR abstracts those settings.

```text
Invalid time source. Supported: ...
Invalid clock source. Supported: ...
```

To fix this, the source code of `srsRAN_Project` was patched to **remove the references to `time_source` and `clock_source`**, ensuring compatibility with LimeSDR via SoapyUHD.

You can find the exact patched version used in this setup here:

🔗 **Custom srsRAN fork with LimeSDR compatibility**:  
[https://github.com/ramosaurio/srsRAN_Project/releases/tag/release_limesdr_24_10_1](https://github.com/ramosaurio/srsRAN_Project/releases/tag/release_limesdr_24_10_1)

---

### 3. SDR not detected

Make sure your LimeSDR Mini is connected and recognized:

```bash
LimeUtil --find
```

Also verify that `udev` rules have been installed:

```bash
cd submodules/LimeSuite/udev-rules && sudo bash install.sh
```

---

### 4. iPhone  blocks SIM from attaching to 5G SA network

During testing, it was observed that **iPhones (e.g. iPhone 13)** may silently reject attaching to a standalone 5G network even if the PLMN, SIM and gNB are correctly configured. This is due to **Apple blacklisting certain ICCIDs** (SIM card identifiers) when they are used with unsupported or unknown PLMN values.

Once a SIM is rejected under a non-whitelisted PLMN, its ICCID may be flagged internally by iOS, and subsequent attempts to connect—even with correct configurations—will fail.

#### ✅ Solution: Temporary use of a "Private Mobile Network PLMN"

To bypass this behavior and “whitelist” the SIM inside the iPhone, follow these steps:

1. **Program the SIM** with a known *private mobile network* PLMN, e.g., `999 70`.
2. **Start the 5G Core** using the a private PLMN (`999 70`) on Open5GS.
3. **Start the gNB** using the a private PLMN (`999 70`) on the patched `srsRAN_Project`.
4. Insert the SIM into the iPhone and **wait for it to attach for some kind of attachment or see the network** to the 5G SA network. (if not just wait a minut once the iphone finishs to scan the networks and try to see it manually, if the network not shows cannot continue)
5. Once attached:
    - Shut down the network (gNB and 5GC).
    - Enable Airplane Mode on the iPhone.
    - Remove the SIM.
6. Re-program the SIM with your **desired PLMN**, e.g., `001 01`.
7. Reinsert the SIM and start the network with the new PLMN.
8. The iPhone will now **allow attaching with the new PLMN**, since the ICCID has already been internally validated.

📌 *This workaround effectively “unblacklists” the SIM’s ICCID for 5G SA use.*

🔗 Reference:  
[https://github.com/herlesupreeth/docker_open5gs/discussions/248](https://github.com/herlesupreeth/docker_open5gs/discussions/248)  
*(Credit to @herlesupreeth for documenting this behavior)*

### 5. iPhone does not see or attach to the 5G SA network (SUPI concealment required)

Even when all technical parameters are correctly configured (PLMN, IMSI, APN, network broadcasting), **iPhones** (e.g., iPhone 13) may not display or attach to standalone 5G networks. This is due to Apple's strict **compliance requirements for private 5G**, notably the use of **SUPI concealment**, mandatory **NAS encryption**, and proper **user-plane security**.

These are security and privacy features enforced by iOS, as described in Apple’s official deployment documentation.

#### ✅ Solution: Enable full iPhone support for 5G SA networks

To meet Apple's requirements, perform the following steps:

1. **Program the SIM card** with mandatory 5G files using the `pySim-shell` tool:

   ```bash
   ./pySim-shell.py -p 0 --script scripts/deactivate-5g.script
   ./pySim-shell.py -p 0 --script scripts/iphone-private-5g.script
   ```

   > 📌 Make sure to **edit the script** and set the correct `ADM` key for your SIM card.

2. **Enable NAS encryption** in Open5GS (`amf.yaml` file):

   ```yaml
   security:
       integrity_order : [ NIA2, NIA1, NIA0 ]
       ciphering_order : [ NEA2, NEA1, NEA0 ]
   ```

3. **Enable user-plane encryption** in the gNB (srsRAN Project):

   ```bash
   cu_cp security --nea_pref_list=nea2,nea1,nea3,nea0
   ```

4. Use a **private mobile network PLMN**, e.g., `999 70`.

These steps ensure that:

- SUPI concealment is respected
- NAS integrity and ciphering are correctly negotiated
- iPhone considers the network "valid" for 5G SA attach

After these changes, the iPhone should be able to **detect and attach** to your private 5G network.


🔗 Reference:  
[https://github.com/herlesupreeth/docker_open5gs/discussions/248](https://github.com/herlesupreeth/docker_open5gs/discussions/248)  
*(Credit to @herlesupreeth for documenting this behavior)*

### 6. Instability in multi-host setups without an external clock source

During testing, it was observed that **multi-host deployments** of the 5G SA network components can lead to **significant instability**. This is primarily due to the **lack of an external clock source**, which is critical for maintaining synchronization between the SDR and distributed network components (e.g., gNB and 5GC on different machines).

In the absence of precise clock synchronization (typically achieved via GPSDOs or dedicated timing hardware), packet loss, attach failures, and unexpected behavior may occur—especially in real-time radio interfaces.

#### ✅ Solution: Use a monolithic, single-host architecture

To ensure system stability and consistent behavior in this testbed:

- All components (5GC, gNB, SIM provisioning tools, etc.) are deployed on a **single host**.
- The SDR relies on its **internal clock** for timing, which is adequate for controlled local environments.
- This choice is also driven by **hardware limitations and cost considerations**, as external clocking solutions (GPSDO, IEEE 1588 PTP) may not be feasible for low-cost or home lab setups.

📌 *This approach increases reliability for development and functional testing, at the cost of scalability.*

### 7. `TRANSFER ERROR` with LimeSDR Mini due to unstable USB voltage

During testing with the **LimeSDR Mini**, repeated `[FATAL] [UHDSoapyDevice] TRANSFER ERROR` messages were observed shortly after starting the `gnb` service. These errors are caused by **instabilities in USB communication**, which can stem from insufficient or fluctuating voltage on the USB port.

This issue is especially common when:

- Using **USB front-panel ports**, laptop ports, or low-quality USB hubs.
- The LimeSDR Mini is operated with **high TX gain** or **high sample rates**, increasing its power demand.
- There are **multiple high-power USB devices** sharing the same internal USB controller.

Below is an example of the log output when this error occurs:

```
 gnb -c gnb_rf_limesdr_01.yml 

--== srsRAN gNB (commit 7e10e42fd3) ==--


The PRACH detector will not meet the performance requirements with the configuration {Format B4, ZCZ 0, SCS 30kHz, Rx ports 1}.
Lower PHY in quad executor mode.
Available radio types: uhd.
[INFO] [UHD] linux; GNU C++ version 9.4.0; Boost_107100; UHD_4.8.0.HEAD-0-50967d13
[INFO] [LOGGING] Fastpath logging disabled at runtime.
Making USRP object with args 'type=soapy,num_recv_frames=64,num_send_frames=64'
[INFO] [UHDSoapyDevice] Make connection: 'LimeSDR Mini [USB 3.0] 1DA160BEF543E4'
[INFO] [UHDSoapyDevice] Reference clock 40.00 MHz
[INFO] [UHDSoapyDevice] Device name: LimeSDR-Mini_v2
[INFO] [UHDSoapyDevice] Reference: 40 MHz
[INFO] [UHDSoapyDevice] LMS7002M register cache: Disabled
[INFO] [UHDSoapyDevice] Filter calibrated. Filter order-4th, filter bandwidth set to 23.04 MHz.Real pole 1st order filter set to 2.5 MHz. Preemphasis filter not active
[INFO] [UHDSoapyDevice] TX LPF configured
[INFO] [UHDSoapyDevice] RX LPF configured
Cell pci=1, bw=20 MHz, 1T1R, dl_arfcn=632628 (n78), dl_freq=3489.42 MHz, dl_ssb_arfcn=632256, ul_freq=3489.42 MHz

N2: Connection to AMF on 127.0.0.1:38412 completed
[INFO] [UHDSoapyDevice] Tx calibration finished
[INFO] [UHDSoapyDevice] Rx calibration finished
==== gNB started ===
Type <h> to view help
[FATAL] [UHDSoapyDevice] TRANSFER ERROR
[FATAL] [UHDSoapyDevice] TRANSFER ERROR
[FATAL] [UHDSoapyDevice] TRANSFER ERROR
[FATAL] [UHDSoapyDevice] TRANSFER ERROR
[FATAL] [UHDSoapyDevice] TRANSFER ERROR
...
```

#### ✅ Solution: Use a high-quality USB 3.0 port with stable power

To resolve this issue:

- Connect the LimeSDR Mini to a **rear-panel USB 3.0 port** directly on the motherboard (preferred).
- Alternatively, use a **powered USB 3.0 hub** that provides stable 5V with sufficient current (≥1A).
- Avoid passive USB hubs or extension cables, as they can introduce voltage drops.
- Once the LimeSDR Mini was connected to a USB port with **better voltage stability**, the `TRANSFER ERROR` messages stopped, and the `gnb` process remained stable.

📌 *Ensuring clean and stable power delivery to the SDR is crucial for reliable operation at high bandwidths and TX levels.*


Sistema en tiempo real:

tail -f /tmp/gnb.log
2025-05-08T12:14:42.015028 [GNB     ] [I] Built in Release mode using commit 7e10e42fd3 on branch HEAD
2025-05-08T12:14:44.790681 [PHY     ] [W] [   10.19] Real-time failure in low-phy: PRACH request late for sector 0, slot 10.19 and start symbol 0.
2025-05-08T12:14:44.800652 [PHY     ] [W] [   11.19] Real-time failure in low-phy: PRACH request late for sector 0, slot 11.19 and start symbol 0.
2025-05-08T12:14:44.810722 [PHY     ] [W] [   12.19] Real-time failure in low-phy: PRACH request late for sector 0, slot 12.19 and start symbol 0.
2025-05-08T12:14:44.820707 [PHY     ] [W] [   13.19] Real-time failure in low-phy: PRACH request late for sector 0, slot 13.19 and start symbol 0.
2025-05-08T12:14:44.831697 [PHY     ] [W] [   14.19] Real-time failure in low-phy: PRACH request late for sector 0, slot 14.19 and start symbol 0.
2025-05-08T12:14:44.841727 [PHY     ] [W] [   15.19] Real-time failure in low-phy: PRACH request late for sector 0, slot 15.19 and start symbol 0.
2025-05-08T12:14:44.851649 [PHY     ] [W] [   16.19] Real-time failure in low-phy: PRACH request late for sector 0, slot 16.19 and start symbol 0.
2025-05-08T12:14:44.860731 [PHY     ] [W] [   17.19] Real-time failure in low-phy: PRACH request late for sector 0, slot 17.19 and start symbol 0.
2025-05-08T12:14:44.870582 [PHY     ] [W] [   18.19] Real-time failure in low-phy: PRACH request late for sector 0, slot 18.19 and start symbol 0.
2025-05-08T12:14:44.880701 [PHY     ] [W] [   19.19] Real-time failure in low-phy: PRACH request late for sector 0, slot 19.19 and start symbol 0.
2025-05-08T12:14:44.891690 [PHY     ] [W] [   20.19] Real-time failure in low-phy: PRACH request late for sector 0, slot 20.19 and start symbol 0.
2025-05-08T12:14:44.900584 [PHY     ] [W] [   21.19] Real-time failure in low-phy: PRACH request late for sector 0, slot 21.19 and start symbol 0.
2025-05-08T12:14:44.911645 [PHY     ] [W] [   22.19] Real-time failure in low-phy: PRACH request late for sector 0, slot 22.19 and start symbol 0.


sudo apt install -y linux-tools-common linux-tools-generic
sudo cpupower frequency-set -g performance

1. ✅ Fijar el modo "performance" en todos los núcleos

sudo apt install -y linux-tools-common linux-tools-generic
sudo cpupower frequency-set -g performance

Para que se aplique siempre:

sudo systemctl enable cpupower.service

2. ✅ Desactivar servicios que interrumpen (GUI, snapd, etc.)

Desde consola (TTY o SSH), detén servicios innecesarios:

sudo systemctl stop gdm3     # o lightdm, sddm, según tu entorno
sudo systemctl stop snapd

3. ✅ Aislar núcleos para tiempo real (kernel boot param)

Edita el archivo de configuración del GRUB:

sudo nano /etc/default/grub

Cambia esta línea:

GRUB_CMDLINE_LINUX_DEFAULT="quiet splash"

Por esta (ajustando núcleos según tu CPU):

GRUB_CMDLINE_LINUX_DEFAULT="quiet splash isolcpus=2,3 nohz_full=2,3 rcu_nocbs=2,3"

Luego:

sudo update-grub
sudo reboot

4. ✅ Verifica que los núcleos estén aislados

Después del reinicio:

cat /proc/cmdline

Y:

htop

Confirma que los núcleos 2 y 3 no tengan procesos del sistema.
5. ✅ Ejecución en tiempo real del gNB

sudo taskset -c 2,3 chrt -f 99 ./gnb -c gnb_rf_limesdr_01.yml

6. ✅ Conectar el LimeSDR Mini en puerto USB 3.0 directo a placa

   Evita hubs y extensores.

   Usa cable corto, blindado, de buena calidad.

   Usa lsusb -t para confirmar que está en un root hub USB 3.0.

7. ✅ Verifica que Soapy y LimeSuite estén actualizados

sudo apt update
sudo apt install soapysdr-tools limesuite liblimesuite-dev
LimeUtil --update

8. ✅ Desactiva ahorro de energía USB (por si Ubuntu intenta dormir el bus)

sudo nano /etc/default/grub

Añade usbcore.autosuspend=-1:

GRUB_CMDLINE_LINUX_DEFAULT="quiet splash usbcore.autosuspend=-1 ..."

Luego:

sudo update-grub
sudo reboot

9. ✅ (Opcional) Usa kernel de baja latencia

sudo apt install linux-lowlatency

Reinicia y selecciona el kernel lowlatency desde GRUB si no es por defecto.
🧪 Resultado esperado

Con todo esto deberías poder ejecutar el gnb así:

sudo taskset -c 2,3 chrt -f 99 ./gnb -c gnb_rf_limesdr_01.yml

Y lograr 0 errores PRACH con esta configuración:

srate: 30.72e6
channel_bandwidth_MHz: 30
tx_gain: 45


---

## 📱 Tested User Equipment (UE)



## 🧪 Tested Configurations



## 📚 References and Credits


- **Open5GS Project**:
    - [https://github.com/open5gs/open5gs](https://github.com/open5gs/open5gs)  
      Open-source 5G Core used for AMF, SMF, UPF and UDM roles in this standalone 5G deployment.

- **iPhone 5G SA Compatibility Scripts** by @herlesupreeth:
    - [https://github.com/herlesupreeth/docker_open5gs/blob/master/sim/iphone-private-5g.script](https://github.com/herlesupreeth/docker_open5gs/blob/master/sim/iphone-private-5g.script)  
      Script for programming 5G USIM files required for iPhone compatibility with private SA networks.

- **Apple Platform Deployment – Cellular Requirements**:
    - [https://support.apple.com/en-ie/guide/deployment/depac6747317/web](https://support.apple.com/en-ie/guide/deployment/depac6747317/web)  
      Official Apple documentation outlining cellular compatibility requirements for iOS devices.

- **iPhone + Open5GS 5G SA compatibility discussion**:
    - [https://github.com/herlesupreeth/docker_open5gs/discussions/248](https://github.com/herlesupreeth/docker_open5gs/discussions/248)  
      Community thread that documents SIM provisioning, PLMN constraints, and iOS-specific limitations in private 5G SA networks. Authored and moderated by @herlesupreeth.

- **Open5GS Docker Deployment by @herlesupreeth**:
    - [https://github.com/herlesupreeth/docker_open5gs](https://github.com/herlesupreeth/docker_open5gs)  
      Docker-based deployment of Open5GS 5G Core, including preconfigured UDM, AMF, SMF, UPF and tools for SIM provisioning, iPhone compatibility, and SUPI concealment. Extensively used in this project for fast prototyping and integration with custom gNB builds.

- **UHD – USRP Hardware Driver**:
    - [https://github.com/EttusResearch/uhd](https://github.com/EttusResearch/uhd)  
      Official UHD repository for Ettus SDRs. Provides `uhd_find_devices` and other utilities used to detect SDRs like the LimeSDR via Soapy.

- **SoapySDR Project**:
    - [https://github.com/pothosware/SoapySDR](https://github.com/pothosware/SoapySDR)  
      Vendor-neutral SDR abstraction library used to interface the LimeSDR Mini with UHD and other SDR frameworks.

- **LimeSuite (LimeSDR Drivers and Tools)**:
    - [https://github.com/myriadrf/LimeSuite](https://github.com/myriadrf/LimeSuite)  
      Driver suite and GUI tools for LimeSDR hardware. Used in this setup to ensure compatibility and device detection.

- **Integration of Soapy with UHD**:
    - [https://github.com/pothosware/SoapyUHD](https://github.com/pothosware/SoapyUHD)  
      SoapySDR plugin that bridges UHD applications with Soapy-supported SDRs like LimeSDR. Enables `uhd_find_devices` to recognize LimeSDR Mini.

- **USB 3.0 Performance Note for LimeSDR Mini**:
    - [https://wiki.myriadrf.org/LimeSDR-Mini_Quick_Test](https://wiki.myriadrf.org/LimeSDR-Mini_Quick_Test)  
      Official guide noting the importance of connecting LimeSDR Mini to a USB 3.0 port to avoid throughput bottlenecks in SDR applications.
- **Docker Docks Installation**:
    - [https://docs.docker.com/engine/install/ubuntu/](https://docs.docker.com/engine/install/ubuntu/)  
      Official guide for Ubuntu installation

---

## 👨‍💻 Author

- **Name**: Rafael Moreno
- **Email**: rmoreno.morcam@gmail.com
- **Project**: Final Degree Project (TFG) - Computer Engineering

---