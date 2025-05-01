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

| Software / Library         | Version Used / Recommended         | Purpose                                                         | Source / Notes                                                              |
|---------------------------|------------------------------------|-----------------------------------------------------------------|-----------------------------------------------------------------------------|
| **srsRAN Project**        | `release_24_03`                    | 5G stack: gNB, 5GC (AMF, SMF, UPF), CU/DU split                  | [GitHub](https://github.com/srsran/srsRAN_Project)                         |
| **LimeSuite**             | `v22.09.1`                         | SDR driver and utilities for LimeSDR hardware                   | [GitHub](https://github.com/myriadrf/LimeSuite)                             |
| **SoapySDR**              | `soapy-sdr-0.8.1`                  | SDR abstraction layer                                           | [GitHub](https://github.com/pothosware/SoapySDR)                            |
| **Python**                | `3.9+`                             | Required for scripts and SIM provisioning tools                 | Bundled with Ubuntu 20.04+                                                  |
| **CMake**                 | `3.16+`                            | Build system for C++ components                                 | `sudo apt install cmake`                                                    |
| **GCC / G++**             | `>=9.x`                            | C/C++ compiler                                                  | Default in Ubuntu 20.04 / Debian 11                                         |
| **libboost-dev**          | `>=1.71`                           | Dependency for various components                               | `sudo apt install libboost-all-dev`                                         |
| **libconfig++-dev**       | `1.5.x` or compatible              | Configuration parser                                            | `sudo apt install libconfig++-dev`                                          |

## 💻 Minimal Hardware Requirements

- **CPU**: Quad-core Intel or better (e.g., Intel NUC / Ryzen)
- **RAM**: 8GB minimum (tested with 4GB with limitations)
- **Disk**: 16GB SSD recommended
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

## ⚙️ Build and install all components

```bash
# Define the local install path
export SRSRAN_INSTALL=$HOME/ran-5g
mkdir -p "$SRSRAN_INSTALL"
export LD_LIBRARY_PATH=${SRSRAN_INSTALL}/lib
export PATH=${SRSRAN_INSTALL}/bin:$PATH
```

### 1. Build and install SoapySDR
📖 [Reference: SoapySDR Wiki](https://github.com/pothosware/SoapySDR/wiki)

```bash
cd submodules/SoapySDR
mkdir build && cd build
cmake -DCMAKE_PREFIX_PATH=$SRSRAN_INSTALL -DCMAKE_INSTALL_PREFIX=$SRSRAN_INSTALL ..
make -j$(nproc)
make install
```

### 2. Build and install UHD
📖 [Reference: UHD Wiki](https://github.com/EttusResearch/uhd/wiki)

> ⚠️ You **must** build UHD from the `host/` directory.

```bash
  cd submodules/uhd/host
mkdir build && cd build
cmake -DCMAKE_PREFIX_PATH=$SRSRAN_INSTALL -DCMAKE_INSTALL_PREFIX=$SRSRAN_INSTALL ..
make -j$(nproc)
make install
```

### 3. Build and install SoapyUHD
📖 [Reference: SoapyUHD Wiki](https://github.com/pothosware/SoapyUHD/wiki)

```bash
  cd submodules/SoapyUHD
mkdir build && cd build
cmake -DCMAKE_PREFIX_PATH=$SRSRAN_INSTALL -DCMAKE_INSTALL_PREFIX=$SRSRAN_INSTALL ..
make -j$(nproc)
make install
```

### 4. Build and install LimeSuite
📖 [Reference: SoapySDR Wiki](https://github.com/pothosware/SoapySDR/wiki)

```bash
  cd submodules/LimeSuite
mkdir build && cd build
cmake -DCMAKE_PREFIX_PATH=$SRSRAN_INSTALL -DCMAKE_INSTALL_PREFIX=$SRSRAN_INSTALL ..
make -j$(nproc)
make install
```

### 5. Build and install srsRAN_Project
📖 [Reference: srsRAN Installation Guide](https://docs.srsran.com/projects/project/en/latest/user_manuals/source/installation.html)

> 📌 Make sure you're on tag `release_limesdr_24_10_1`. This is already set in the submodule but it's good practice to verify:

```bash
  cd submodules/srsRAN_Project
git checkout release_limesdr_24_10_1
mkdir build && cd build
cmake -DCMAKE_PREFIX_PATH=$SRSRAN_INSTALL -DCMAKE_INSTALL_PREFIX=$SRSRAN_INSTALL ..
make -j$(nproc)
make install
```

---

## ✅ Verifying the installation

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

---
🔗 Reference:  
[https://github.com/herlesupreeth/docker_open5gs/discussions/248](https://github.com/herlesupreeth/docker_open5gs/discussions/248)  
*(Credit to @herlesupreeth for documenting this behavior)*
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
---

## 👨‍💻 Author

- **Name**: Rafael Moreno
- **Email**: rmoreno.morcam@gmail.com
- **Project**: Final Degree Project (TFG) - Computer Engineering

---