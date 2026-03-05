# AMD VEK280 Boot Issue: Black UART0 Terminal

**Author:** Julien Posso (ONERA)

**Date:** 2026-03

**Tags:** `AMD-Versal` `VEK280` `QuickStart` `Vitis-AI`

## 🎯 The Goal
Trying to execute the [Vitis-AI 3.5 quickstart guide](https://xilinx.github.io/Vitis-AI/3.5/html/docs/quickstart/vek280.html) on a new board.

## ⚙️ Environment
* **Hardware:** AMD Versal VEK280 (rev B3)
* **OS:** N/A
* **Toolchain:** Vitis-AI 3.5

## 💥 The Problem
The board does not boot from the micro SD card. The Versal UART0 terminal remains completely black (shows nothing).

!!! failure "The Root Cause"
    Vitis-AI 3.5 is deprecated and strictly incompatible with Versal production boards (like the VEK280 rev B3). However, as of early 2026, the 3.5 documentation is still the top result referenced on Google, leading to massive losses of time.

## 💡 The Workaround / Solution

!!! success "The Fix: Upgrade Vitis-AI"
    You must use **Vitis-AI 5.1 or above**. AMD only released the compatible toolchain for production boards in the first quarter of 2026.

1. Go to the [Vitis-AI 5.1+ Documentation](https://vitisai.docs.amd.com/en/latest/index.html).
2. If needed, use the icon at the bottom right of the AMD webpage to ensure you are on version 5.1 or higher.
3. Follow the Quick Start Guide (and check the System Requirements).

## 🔗 References
* 📖 [Vitis-AI 5.1 and above (Official Docs)](https://vitisai.docs.amd.com/en/latest/index.html)
* 💬 [Related AMD Support Forum post](https://adaptivesupport.amd.com/s/question/0D54U00008LPh76SAD/issues-booting-vek280?language=en_US)
