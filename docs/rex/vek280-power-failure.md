# AMD VEK280 Power Failure

**Author:** Julien Posso (ONERA)

**Date:** 2026-03

**Tags:** `VEK280` `Hardware` `Boot` `Power-Sequencing` `Versal`

## 🎯 The Goal
Successfully boot the board using the physical power switch (SW13) (OFF -> ON) after a previous session.

## ⚙️ Environment
* **Hardware:** AMD Versal VEK280
* **OS:** N/A
* **Toolchain:** N/A

## 💥 The Problem
The board stays dead when toggling SW13 to ON. This occurs specifically after a successful first boot, mimicking a total power failure.

!!! failure "The Root Cause"
    Residual charge in capacitors and **parasitic power** from the USB/JTAG cable keep the PMIC in a locked state, preventing the power-on sequence from re-triggering.

## 💡 The Workaround / Solution
!!! success "The Fix: 30s Cold Boot"
    1. Toggle **SW13 to OFF**.
    2. **Disconnect USB** and **Unplug Power Cable**.
    3. **Wait 30 seconds** (essential for capacitor discharge).
    4. Reconnect cables and toggle **SW13 to ON**.

## 🔗 References

* [VEK280 Evaluation Board User Guide (UG1612)](https://docs.amd.com/r/en-US/ug1612-vek280-eval-bd)

