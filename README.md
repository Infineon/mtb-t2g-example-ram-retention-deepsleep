<img src="./images/IFX_LOGO_600.gif" align="right" width="150"/>

# RAM Retention in DeepSleep Power Mode

**This code example demonstrates how to set RAM into Retention mode before entering DeepSleep power mode.**  

## Device

The device used in this code example (CE) is:

- [TRAVEO™ T2G CYT4DN Series](https://www.infineon.com/cms/en/product/microcontroller/32-bit-traveo-t2g-arm-cortex-microcontroller/32-bit-traveo-t2g-arm-cortex-for-cluster/traveo-t2g-cyt4dn/)

## Board

The board used for testing is:

- TRAVEO&trade; T2G Cluster 6M Lite Kit ([KIT_T2G_C-2D-6M_LITE](https://www.infineon.com/cms/en/product/evaluation-boards/kit_t2g_c-2d-6m_lite/))

## Scope of work
TRAVEO™ T2G MCUs provides the Static RAM (SRAM) Retention function in DeepSleep mode. This code example writes known data into SRAM Controller 1 and sets the SRAM into Retention mode before entering DeepSleep mode. After wakeup from DeepSleep mode, data written to SRAM Controller 1 is read back and compared to the data written before entering DeepSleep mode to check if the data is retained.

## Introduction  

**SRAM Interface**  

- Optional memory size: 640 KB
- Advanced eXtensible Interface (AXI) bus interfaces:
    - In the fast clock domain for the CM7 CPUs
- Advanced High-performance Bus (AHB)-Lite bus interface:
    - In the slow clock domain for all bus masters (CM0+ CPU, Crypto, P-DMA, M-DMA, debug interface, and optional external bus master). The slow bus infrastructure combines the bus masters in the slow clock domain.
- Programmable wait states.
- Error Correction Code (ECC) function
    - Single-bit error correction and double-bit error detection (SECDED)
    - ECC error injection
- RAM retention function
- RAM power-up delay control
    - Setting the power stabilization wait after switching on the SRAM power domain.

**Device Power Modes**  

The TVII-C device can operate in different power modes that are intended to minimize the average power consumption in an application. The power modes supported by TVII-C in the order of decreasing power consumption are:
<table border="1" style="border-collapse: collapse">
<thead><tr>
<th>Power Mode</th><th>Description</th><th>Entry Condition</th><th>Wakeup Source</th><th>Wakeup Action</th></tr></thead>
<tbody>
<tr><td><code>Active</code></td><td>Primary mode of operation; all peripherals are available (programmable).</td><td>Wakeup from Sleep/DeepSleep modes, Hibernate reset, or any other reset.</td><td>Not applicable</td><td>Not applicable</td></tr>
<tr><td><code>Low-Power Active Profile</code></td><td>A low-power profile of Active mode; most peripherals are available with limited capabilities</td><td>Register write from Active mode and wakeup from LPSLEEP/DeepSleep 
modes.</td><td>Not applicable</td><td>Not applicable</td></tr>
<tr><td><code>Sleep</code></td><td>CPU is in Sleep mode; all other peripherals are available.</td><td>Register write from Active mode or wakeup from DeepSleep through debugger</td><td>Any interrupt to CPU</td><td>Interrupt</td></tr>
<tr><td><code>Low-Power Sleep Profile</code></td><td>A low-power profile of Sleep mode; CPU is in Sleep mode; most peripherals are available with limited capabilities.</td><td>Register write from LPACTIVE mode.</td><td>Any interrupt to CPU</td><td>Interrupt</td></tr>
<tr><td><code>DeepSleep</code></td><td>All high-frequency clocks and peripherals are turned off. Low-frequency clock (ILO) and low-power analog and digital peripherals are available for operation and as wakeup sources. SRAM can be retained (configurable).</td><td>Register write from Active or LPACTIVE modes.</td><td>GPIO interrupt, event generators, SCB, watchdog timer, and RTC alarms and debugger</td><td> Interrupt or Debug</td></tr>
<tr><td><code>Hibernate</code></td><td>GPIO states are frozen; all high-frequency clocks and peripherals are switched off. Low-frequency clocks (32 kHz), WCO, or LPECO can function. Device resets on wakeup event.</td><td>Register write from Active or LPACTIVE modes.</td><td>WAKEUP pins, RTC alarm, and watchdog timer</td><td> Hibernate Reset</td> </tr>
</tbody>
</table>

More details can be found in:
- TRAVEO&trade; T2G CYT4DN
  - [Technical Reference Manual (TRM)](https://www.infineon.com/dgdl/?fileId=8ac78c8c8691902101869f03007d2d87)
  - [Registers TRM](https://www.infineon.com/dgdl/?fileId=8ac78c8c8691902101869ef098052d79)
  - [Data Sheet](https://www.infineon.com/dgdl/?fileId=8ac78c8c869190210186f0cceff43fd0)

## Hardware setup

This CE has been developed for:
- TRAVEO&trade; T2G Cluster 6M Lite Kit ([KIT_T2G_C-2D-6M_LITE](https://www.infineon.com/cms/en/product/evaluation-boards/kit_t2g_c-2d-6m_lite/))<BR>

**Figure 1. KIT_T2G_C-2D-6M_LITE (Top View)**

<img src="./images/kit_t2g_c-2d-6m_lite.png" width="800" /><BR>
No changes are required from the board's default settings.

## Implementation

This multi-core application has the main function running on the CM7_0 core.
 - *Power state machine*<BR><img src="./images/state_machine.PNG"/><BR>

**Setting up RTC as wakeup source**  

This example uses RTC (Real Time Clock) as a wakeup source from DeepSleep mode. The RTC is configured and controlled by HAL (Hardware Abstraction Layer) functions.
- After initializing the RTC by <a href="https://infineon.github.io/mtb-pdl-cat1/pdl_api_reference_manual/html/group__group__rtc__general__functions.html#ga5e627b3c4c82ee379c04dcb7e754f7d8"><i>Cy_RTC_SelectClockSource()</i></a>, <a href="https://infineon.github.io/mtb-pdl-cat1/pdl_api_reference_manual/html/group__group__rtc__general__functions.html#ga4c6aca12f876f94707818e7a5dcdbf82"><i>Cy_RTC_Init()</i></a> sets the current time and date
- <a href="https://infineon.github.io/mtb-pdl-cat1/pdl_api_reference_manual/html/group__group__rtc__interrupt__functions.html#gaf2ab28f85a042dc1c68b35e0c809eade"><i>Cy_RTC_SetInterruptMask()</i></a> enables RTC events that generate RTC interrupts at the specified timing
- In *SetRtcAlarmDateTime()* function, <a href="https://infineon.github.io/mtb-pdl-cat1/pdl_api_reference_manual/html/group__group__rtc__alarm__functions.html#ga92807dc709c679c70c07d44cf0331a94"><i>Cy_RTC_SetAlarmDateAndTime()</i></a> sets the alarm event occurrence after 10 seconds

**SRAM configuration and transition to DeepSleep mode**  

- After in system initialization is completed, known data is written to SRAM_CONTROLLER1 memory location (0x28040000 to 0x2807FFFF). In main function, Retention mode is enabled before entering DeepSleep mode, and after wakeup from DeepSleep mode via RTC Interrupt (every 10 second interrupt is triggered), SRAM is put in enabled mode. These operations can be done by setting *CPUSS_RAM1_PWR_CTL_PWR_MODE* field.
- The transition to DeepSleep mode is made by a call to <a href="https://infineon.github.io/mtb-pdl-cat1/pdl_api_reference_manual/html/group__group__syspm__functions__power.html#ga5150c28fe4d2626720c1fbf74b3111ca"><i>Cy_SysPm_CpuEnterDeepSleep()</i></a>, a HAL function, which manages whether the current MCU state meets the conditions for transition to DeepSleep. In this example, the TX FIFO of the UART used for string output to the terminal should be empty, so <a href="https://infineon.github.io/mtb-pdl-cat1/pdl_api_reference_manual/html/group__group__syslib__functions.html#gaad1c32546fdb0e3c6fa8b46fb95843b5"><i>Cy_SysLib_Delay()</i></a> is used to wait until it becomes empty.
- After wakeup, the data written in SRAM_CONTROLLER1 memory location is read back to check if data is retained or not. Result is then printed on the terminal and the loop starts again.

**Miscellaneous settings**  

- **STDIN / STDOUT setting**

  - Calling <a href="https://infineon.github.io/retarget-io/html/group__group__board__libs.html#ga21265301bf6e9239845227c2aead9293"><i>cy_retarget_io_init()</i></a> function to use UART as STDIN / STDOUT
    - Initialize the port defined as *CYBSP_DEBUG_UART_TX* as UART TX, defined as *CYBSP_DEBUG_UART_RX* as UART RX (these pins are connected to KitProg3 COM port)
    - The serial port parameters become to 8N1 and 115200 baud
  - In *PrintDebugString()* function, the current time obtained by <a href="https://infineon.github.io/mtb-pdl-cat1/pdl_api_reference_manual/html/group__group__rtc__general__functions.html#gace30abad5d40fc768d7c4fc14e84e847"><i>Cy_RTC_GetDateAndTime()</i></a> is output to the terminal along with a debug string

**Cores other than CM7_0**

Since all cores must transition to DeepSleep for the entire system becomes DeepSleep mode, cores other than CM7_0 are provided as projects to explicitly transition to DeepSleep.

## Run and Test

For this CE, a terminal emulator is required for displaying outputs. Install a terminal emulator if you do not have one. Instructions in this document use [Tera Term](https://teratermproject.github.io/index-en.html).

After code compilation, perform the following steps to flashing the device:

1. Connect the board to your PC using the provided USB cable through the KitProg3 USB connector.
2. Open a terminal program and select the KitProg3 COM port. Set the serial port parameters to 8N1 and 115200 baud.
3. Program the board using one of the following:
    - Select the CE project in the Project Explorer.
    - In the **Quick Panel**, scroll down, and click **[Project Name] Program (KitProg3_MiniProg4)**.
4. After programming, the CE starts automatically. Confirm that the messages are displayed on the UART terminal.

    - *Terminal output on program startup*<BR><img src="./images/terminal.PNG" width="640" />

5. You can debug the example to step through the code. In the IDE, use the **[Project Name] Debug (KitProg3_MiniProg4)** configuration in the **Quick Panel**. For details, see the "Program and debug" section in the [Eclipse IDE for ModusToolbox™ software user guide](https://www.infineon.com/dgdl/?fileId=8ac78c8c8929aa4d0189bd07dd6113f9).

**Note:** **(Only while debugging)** On the CM7 CPU, some code in *main()* may execute before the debugger halts at the beginning of *main()*. This means that some code executes twice: once before the debugger stops execution, and again after the debugger resets the program counter to the beginning of *main()*. See [KBA231071](https://community.infineon.com/t5/Knowledge-Base-Articles/PSoC-6-MCU-Code-in-main-executes-before-the-debugger-halts-at-the-first-line-of/ta-p/253856) to learn about this and for the workaround.

## References  

Relevant Application notes are:

- [AN235305](https://www.infineon.com/dgdl/?fileId=8ac78c8c8b6555fe018c1fddd8a72801) - Getting started with TRAVEO&trade; T2G family MCUs in ModusToolbox&trade;
- [AN220222](https://www.infineon.com/dgdl/?fileId=5546d462749a7c2d01749b357cbc0ce6) - Low Power Mode Procedure in TRAVEO&trade; T2G Family
- [AN220152](https://www.infineon.com/dgdl/?fileId=5546d462749a7c2d01749b35f7020cfa) - How to Retain RAM Data in Reset Procedure and Low-Power Mode Transition in TRAVEO&trade; T2G Family
- [AN220190](https://www.infineon.com/dgdl/?fileId=8ac78c8c7cdc391c017d0d3a87336756) - How to use RTC in TRAVEO&trade; T2G family

ModusToolbox™ is available online:
- <https://www.infineon.com/modustoolbox>

Associated TRAVEO™ T2G MCUs can be found on:
- <https://www.infineon.com/cms/en/product/microcontroller/32-bit-traveo-t2g-arm-cortex-microcontroller/>

More code examples can be found on the GIT repository:
- [TRAVEO™ T2G Code examples](https://github.com/orgs/Infineon/repositories?q=mtb-t2g-&type=all&language=&sort=)

For additional trainings, visit our webpage:  
- [TRAVEO™ T2G trainings](https://www.infineon.com/cms/en/product/microcontroller/32-bit-traveo-t2g-arm-cortex-microcontroller/32-bit-traveo-t2g-arm-cortex-for-cluster/traveo-t2g-cyt4dn/#!trainings)

For questions and support, use the TRAVEO™ T2G Forum:  
- <https://community.infineon.com/t5/TRAVEO-T2G/bd-p/TraveoII>  
