# 🚀 Quickstart: AMD Versal VCK190

**Author:** Julien Posso (ONERA)  
**Date:** 2026-06
**Tags:** `AMD-Versal` `VCK190` `Quickstart` `Getting Started` `Setup`

This guide supplements the official AMD documentation by providing a high-level system view necessary to avoid common pitfalls during initial setup.

## 🔗 Useful Resources
* 📖 [Official VCK190 Board User Guide (UG1366)](https://docs.amd.com/r/en-US/ug1366-vck190-eval-bd)
* 🌐 [VCK190 Quickstart Guide](http://www.amd.com/vck190-start)

## 📋 Prerequisites

Before proceeding with the board setup, it is mandatory to have your development environment ready. Without this, you will not be able to communicate with the board via UART or JTAG.

* **Vitis & USB Drivers:** You must have the AMD Vitis Toolchain and the necessary USB/JTAG cable drivers installed. If you haven't done this yet, please follow our dedicated guide: [🚀 Setup: AMD Vitis Toolchain](vitis-toolchain-setup.md).

## 🧠 Understanding the Dual-System Architecture
Unlike Zynq boards, the Versal features **two distinct SoCs** running in parallel:

1.  **System Controller (Zynq UltraScale+ MPSoC):**
    * **Role:** Manages board peripherals (power, clocks, BEAM).
    * **Boot:** Boots from SD card.
    * **Interface:** Has its own dedicated Ethernet port and UART.
2.  **Versal MPSoC (Main Target):**
    * **Role:** The primary processor for your research, AI engines, and custom logic.
    * **Boot:** Typically boots from **JTAG** or the **SD Card**.

---

## 🏁 Versal First Boot

!!! warning "Note on the 'In-Box' Quickstart"
    The QR code provided in the box (linking to [amd.com/vck190-start](http://www.amd.com/vck190-start)) points to a guide that contains third party reference designs. These designs require an **FMC card** which is not provided with the standard VCK190 Evaluation Kit.

### 🛠️ SD Card Preparation

1.  **Download** the official VCK190 SD pre-built Image (`.wic`): tested with [SC Update 7.1](https://account.amd.com/en/forms/downloads/xef.html?filename=system-controller-image-full-cmdline-vck-sc-zynqmp.rootfs-20250524151314_update7.1.wic.xz); other versions may be available [here](https://xilinx-wiki.atlassian.net/wiki/spaces/A/pages/2273738753/Evaluation+Board+-+System+Controller+SC#5.1-Board-to-Version-Mapping).
2.  **Unzip** the downloaded archive.
3.  **Flash** using **[Balena Etcher](https://etcher.balena.io/)** to write the WIC image to your micro SD card.

**Tips for using Balena Etcher on Linux:**

1. Download the x64 (64-bit) (zip) version.
2. Extract it and navigate to the `balenaEtcher-linux-x64` folder.
3. Launch the application from a terminal with: `./balena-etcher --no-sandbox`.
4. Superuser authentication will be required to flash the card.

### 🔌 Hardware Setup
![Standard VCK190 setup](https://docs.amd.com/api/khub/maps/AsLG7VfT7S1HdAAfQi~I4A/resources/MA1071kG0gpybeSTzurDBQ-AsLG7VfT7S1HdAAfQi~I4A/content?Ft-Calling-App=ft%2Fturnkey-portal&Ft-Calling-App-Version=5.3.20&filename=rqj1654108314579.image)

| Switch Block | Target | Mode | Configuration |
| :--- | :--- | :--- | :--- |
| **SW1** | Versal Main | **QSPI Boot** (Usine) | `ON, OFF, OFF, OFF` |
| **SW11** | System Controller | **SD Boot** | `ON, OFF, OFF, OFF` |

Follow the standard setup above (only the system controller SD card is needed).

Connecting the board's USB port to your computer will reveal **3 COM ports (Windows) / 4 interfaces (Linux)**:

* **Interface 0**: JTAG (not shown on Windows COM ports).
* **COM N / Interface 1:** Versal UART0 – User Linux (Versal SD Boot).
* **COM N+1 / Interface 2:** Versal UART1 via PL – Unused for now.
* **COM N+2 / Interface 3:** System Controller UART – Management Linux (System Controlelr SD Boot).

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

The **BEAM** (Board Evaluation and Management) tool allows you to verify board health. Access it via the System Controller web interface. Refer to the [Xilinx UG1573](https://docs.amd.com/r/en-US/VCK190/VMK180-Board-Evaluation-and-Management-BEAM-Tool-User-Guide-UG1573/Introduction) for the official guide. Below are HEAT-specific tips.

### 🌐 Network Configuration
1.  On the **System Controller** terminal, launch the network configuration script:
    ```bash
    sudo /usr/bin/system_config.sh
    ```
2. Select the _Configure Network_ option (a static IP is highly recommended for lab environments) and follow the onscreen prompts.
3. Reboot the controller to apply changes: `sudo reboot`.

Once rebooted, the BEAM URL should be accessible from a browser: 
> **🔗 BEAM Web Address:** `http://<your_ip>:50002`

### ✅ Running the Tests
In the BEAM web interface, under **"Test the Board"**:

* **Board Settings:** Manage GPIOs, clocks, FMC, and read real-time voltage/power consumption (e.g., `VCC_INT`).
* **Board Interface Test:** Essential for new boards. It tests LEDs, switches, DRAM, etc.

!!! warning "Important Note on BIT Testing"
    Keep your Versal terminal console (Interface 1) open while running these tests: textual instructions and live progress status for each test step are piped directly to it.

_Note: Automated tests for the QSPI Flash and the X-EBM EEPROM require the boot extension module X-EBM-01 ([QSPI Boot Module](https://www.amd.com/fr/products/adaptive-socs-and-fpgas/evaluation-boards/vck190.html#tabs-fce99290a2-item-1436d838cb-tab)) provided in the box to be physically plugged into the board._

The X-EBM EEPROM and the QSPI automated tests need the X-EBM-01 ([QSPI Boot Module)](https://www.amd.com/fr/products/adaptive-socs-and-fpgas/evaluation-boards/vck190.html#tabs-fce99290a2-item-1436d838cb-tab)) to be plugged in.

If any test fail, follow the _Workaround_ section in our [Versal Power Failure Field Note](https://www.google.com/search?q=../notes/vek280-power-failure.md).

---

## 🚀 Next Steps
Once your board is running, learn how to deploy AI models:

* [Starting Wiki](https://xilinx-wiki.atlassian.net/wiki/spaces/A/pages/748617729/Versal+AI+Core+Series+VCK190+Evaluation+Kit#Running-a-design)
* [Vitis-AI Repository](https://github.com/xilinx/vitis-ai)
