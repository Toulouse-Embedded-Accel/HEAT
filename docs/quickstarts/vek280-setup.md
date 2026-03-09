# 🚀 Quickstart: AMD Versal VEK280

**Author:** Julien Posso (ONERA)  
**Date:** 2026-03  
**Tags:** `VEK280 Rev. B3` `Versal` `Setup` `Hardware`

This guide supplements the official AMD documentation by providing a high-level system view necessary to avoid common pitfalls during initial setup.

## 🔗 Useful Resources
* 📺 [Official VEK280 Unboxing & Setup Video](https://www.youtube.com/watch?v=N3ZAhXxo-4s)
* 🛠️ [Field Note: Known VEK280 Boot Issues](../notes/vek280-boot-issue.md)
* 🛠️ [Field note: VEK280 Power Failure](../notes/vek280-power-failure.md)

## 🧠 Understanding the Dual-System Architecture
Unlike Zynq boards, the Versal features **two distinct SoCs** running in parallel:

1.  **System Controller (Zynq UltraScale+ MPSoC):**
    * **Role:** Manages board peripherals (power, clocks, BEAM).
    * **Boot:** Boots from factory-preconfigured **eMMC**.
    * **Interface:** Has its own dedicated Ethernet port and UART.
2.  **Versal MPSoC (Main Target):**
    * **Role:** The primary processor for your research, AI engines, and custom logic.
    * **Boot:** Typically boots from the **SD Card**. It also handles fan speed control.

---

## 🏁 Versal First Boot

!!! warning "Note on the 'In-Box' Quickstart"
    The QR code provided in the box (linking to [amd.com/vek280-start](http://www.amd.com/vek280-start)) points to a guide that requires an **MRMAC daughter card**. This guide only partially relies on it.

### 🛠️ SD Card Preparation

1.  **Download** the official VEK280 SD Image (`.wic`) which contains the board platform (e.g., [v2025.1 SD Image](https://www.xilinx.com/member/forms/download/xef.html?filename=xilinx-v2025.1_vek280_sdimage.zip)).
2.  **Unzip** the downloaded archive.
3.  **Flash** using **Balena Etcher** to write the WIC image to your micro SD card.

### 🔌 Hardware Setup
![Standard VEK280 setup](https://vitisai.docs.amd.com/en/latest/_images/target_board_updated.png)

Follow the standard setup, noting these specific connections:
* **Ethernet:** Connect to the port located at the **top right** of the board (facing a switch or your computer).
* **USB:** Connecting the board's USB port to your computer will reveal **3 COM ports**:
    * **Port N (Interface 0):** Versal UART0 – User Linux (SD Boot).
    * **Port N+1 (Interface 1):** Versal UART1 via PL – Unused for now.
    * **Port N+2 (Interface 2):** System Controller UART – Management Linux (eMMC Boot).

### 💻 Power On and Login

1.  **Terminal:** Open two serial console instances (e.g., Putty or Minicom) on **Port N** and **Port N+2**.
    * *Settings:* 115200 baud, 8N1.
2.  **Power:** Toggle the **SW13** power switch (located near the power connector) to ON.
3.  **Observation:**
    * **Dual Boot:** You should see two Linux systems booting in parallel on your terminals.
    * **Status:** Power Good LEDs should turn green.
    * **Prompt:** You should reach a login prompt on both COM N and COM N+2.

**Credentials:**
* **System Controller (COM N+2):** Choose your preferred credentials.
* **Versal UART (COM N):** Use the default Petalinux credentials:
    * **Username:** `petalinux`
    * **Password:** `petalinux` (you may be prompted to change it on first login).

## 🔍 Diagnostic Tool: BEAM

The **BEAM** (Board Evaluation and Management) tool allows you to verify board health. Access it via the System Controller web interface. Refer to the [Xilinx Wiki](https://xilinx-wiki.atlassian.net/wiki/spaces/A/pages/2273738753/System+Controller+Updates#Launching-the-GUI) for the official guide. Below are HEAT-specific tips.

### 🌐 Manual IP Configuration (Static IP)
If the System Controller fails to get an IP via DHCP, you must set a static IPv4 address manually via the UART.

1.  On the **System Controller terminal (COM N+2)**, create the network file:
    ```bash
    sudo vi /etc/systemd/network/20-wired.network
    ```
2.  Insert the following content (**Adapt the IP/Gateway** to your local network):
    ```ini
    [Match]
    Name=eth0

    [Network]
    Address=192.168.225.173/16
    Gateway=192.168.90.100
    ConfigureWithoutCarrier=yes
    ```
3.  Restart the System Controller:
    ```bash
    sudo reboot
    ```
Once rebooted, the BEAM URL should be accessible: 
> **🔗 BEAM Web Address:** `http://<your_ip>:50002`

### ✅ Running the Tests
In the BEAM web interface, under **"Test the Board"**:

* **Board Settings:** Manage GPIOs, clocks, FMC, and read real-time voltage/power consumption (e.g., `VCC_INT`).
* **Board Interface Test:** Essential for new boards. It tests LEDs, switches, etc.
* **⚠️ Important (Buttons Test):** Instructions (e.g., *"Press SWx"*) are displayed on the **Versal UART (COM N)**. If you are not watching that terminal, the test will appear to hang or fail.

!!! tip "Expert Advice"
    Always keep the **Versal UART (Port COM N)** window visible while using BEAM to avoid missing critical interactive prompts.


