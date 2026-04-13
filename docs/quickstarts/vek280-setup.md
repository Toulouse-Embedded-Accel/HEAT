# 🚀 Quickstart: AMD Versal VEK280

**Author:** Julien Posso (ONERA)  
**Date:** 2026-03  
**Tags:** `AMD-Versal` `VEK280` `Quickstart` `Getting Started` `Setup`

This guide supplements the official AMD documentation by providing a high-level system view necessary to avoid common pitfalls during initial setup.

## 🔗 Useful Resources
* 📺 [Official VEK280 Introduction Video](https://www.youtube.com/watch?v=N3ZAhXxo-4s)
* 📖 [Official VEK280 Board User Guide (UG1612)](https://docs.amd.com/r/en-US/ug1612-vek280-eval-bd)
* 🛠️ [Field Note: Known VEK280 Boot Issues](../notes/vek280-boot-issue.md)
* 🛠️ [Field note: VEK280 Power Failure](../notes/vek280-power-failure.md)

## 📋 Prerequisites

Before proceeding with the board setup, it is mandatory to have your development environment ready. Without this, you will not be able to communicate with the board via UART or JTAG.

* **Vitis & USB Drivers:** You must have the AMD Vitis Toolchain and the necessary USB/JTAG cable drivers installed. If you haven't done this yet, please follow our dedicated guide: [🚀 Setup: AMD Vitis Toolchain](vitis-toolchain-setup.md).

## 🧠 Understanding the Dual-System Architecture
Unlike Zynq boards, the Versal features **two distinct SoCs** running in parallel:

1.  **System Controller (Zynq UltraScale+ MPSoC):**
    * **Role:** Manages board peripherals (power, clocks, BEAM).
    * **Boot:** Boots from factory-preconfigured **eMMC**.
    * **Interface:** Has its own dedicated Ethernet port and UART.
2.  **Versal MPSoC (Main Target):**
    * **Role:** The primary processor for your research, AI engines, and custom logic.
    * **Boot:** Typically boots from **JTAG** or the **SD Card**. It also handles fan speed control.

---

## 🏁 Versal First Boot

!!! warning "Note on the 'In-Box' Quickstart"
    The QR code provided in the box (linking to [amd.com/vek280-start](http://www.amd.com/vek280-start)) points to a guide that contains a MRMAC Example Design. This design requires an **FMC card** which is not provided with the standard VEK280 Evaluation Kit.

### 🛠️ SD Card Preparation

1.  **Download** the official VEK280 SD pre-built Image (`.wic`) which contains the board platform: tested with [v2025.1 SD Image](https://www.xilinx.com/member/forms/download/xef.html?filename=xilinx-v2025.1_vek280_sdimage.zip); other versions available [here](https://xilinx-wiki.atlassian.net/wiki/spaces/A/pages/18842316).
2.  **Unzip** the downloaded archive.
3.  **Flash** using **[Balena Etcher](https://etcher.balena.io/)** to write the WIC image to your micro SD card.

**Tips for using Balena Etcher on Linux:**

1. Download the x64 (64-bit) (zip) version.
2. Extract it and navigate to the `balenaEtcher-linux-x64` folder.
3. Launch the application from a terminal with: `./balena-etcher --no-sandbox`
4. Superuser authentication will be required to flash the card.

### 🔌 Hardware Setup
![Standard VEK280 setup](https://vitisai.docs.amd.com/en/latest/_images/target_board_updated.png)

Follow the standard setup, noting these specific connections:

* **Ethernet:** Connect to the port located at the **top right** of the board (System Controller Ethernet) instead of the bottom-left shown in the image above (Versal Ethernet),facing a switch or your computer.
* **USB:** Connecting the board's USB port to your computer will reveal **3 COM ports (Windows) / 4 interfaces (Linux)**:
    * **Interface 0**: JTAG (not shown on Windows COM ports).
    * **COM N / Interface 1:** Versal UART0 – User Linux (SD Boot).
    * **COM N+1 / Interface 2:** Versal UART1 via PL – Unused for now.
    * **COM N+2 / Interface 3:** System Controller UART – Management Linux (eMMC Boot).

!!! tip "🐧 Linux Specifics"
    To avoid port shifting (e.g., `/dev/ttyUSB1` becoming `ttyUSB5`), use persistent paths. You can list them with `ls -l /dev/serial/by-path/`.
    Open your terminals with:
    * **Versal:** `tio /dev/serial/by-path/*-usb-0:*:1.1-port0`
    * **System Controller:** `tio /dev/serial/by-path/*-usb-0:*:1.3-port0`

### 💻 Power On and Login

1.  **Terminal:** Open two serial console instances (e.g., Putty, Minicom or Tio) on **Versal UART** and **System Controller UART**.
    * *Settings:* 115200 baud, 8N1.
2.  **Power:** Toggle the **SW13** power switch (located near the power connector) to ON.
3.  **Observation:**
    * **Dual Boot:** You should see two Linux systems booting in parallel on your terminals.
    * **Status:** Power Good LEDs should turn green.
    * **Prompt:** You should reach a login prompt on both Versal and System Controller UART (on System Controller UART, **press Enter** after the terminal displays the *BEAM Tool Web Address* to reach the login prompt).

**Credentials:**

* **System Controller:** Choose your preferred credentials.
* **Versal:** Use the default Petalinux credentials:
    * **Username:** `petalinux`
    * **Password:** Choose your password at the first prompt.

---

## 🔍 Diagnostic Tool: BEAM

The **BEAM** (Board Evaluation and Management) tool allows you to verify board health. Access it via the System Controller web interface. Refer to the [Xilinx Wiki](https://xilinx-wiki.atlassian.net/wiki/spaces/A/pages/2273738753/System+Controller+Updates#Launching-the-GUI) for the official guide. Below are HEAT-specific tips.

### 🌐 Manual IP Configuration (Static IP)
If the System Controller fails to get an IP via DHCP, you must set a static IPv4 address manually via the UART.

1.  On the **System Controller terminal**, create the network file:
    ```bash
    sudo vi /etc/systemd/network/20-wired.network
    ```
2.  Insert the following content (**Adapt the IP/Gateway** to your local network):
    ```ini
    [Match]
    Name=eth0

    [Network]
    Address=192.168.1.173/24
    Gateway=192.168.1.1
    ConfigureWithoutCarrier=yes
    ```
    !!! tip "Quick VI Tutorial"
        1. Press **"i"** to enter insert mode.
        2. Edit your file.
        3. Press **Escape** to return to command mode.
        4. Type **":wq"** and press Enter to save and exit.
3.  Restart the System Controller:
    ```bash
    sudo reboot
    ```
Once rebooted, the BEAM URL should be accessible from a browser: 
> **🔗 BEAM Web Address:** `http://<your_ip>:50002`

### ✅ Running the Tests
In the BEAM web interface, under **"Test the Board"**:

* **Board Settings:** Manage GPIOs, clocks, FMC, and read real-time voltage/power consumption (e.g., `VCC_INT`).
* **Board Interface Test:** Essential for new boards. It tests LEDs, switches, etc.

!!! warning "Important information"
    Instructions (e.g., *"Press SWx"*) are displayed on the **Versal UART**. If you are not watching that terminal, some tests will appear to hang or fail.

---

## 🚀 Next Steps
Once your board is running, learn how to deploy AI models:

* [Official Vitis-AI Documentation](https://vitisai.docs.amd.com/en/latest/index.html)
