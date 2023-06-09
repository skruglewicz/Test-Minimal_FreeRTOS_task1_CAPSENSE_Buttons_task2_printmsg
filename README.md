# FreeRTOS Task Scheduler

The PSoC 62S4 Pioneer Kit, which is a low-cost hardware platform designed for designing and hardware debugging applications using the PSoC 62 MCU.
This is my design for a FreeRTOS Task Scheduler implementation to be used in my final design. It uses the example project CAPSENSE_Buttons_and_Slider_FreeRTOS as a base. I then added my design functionality to test FreeRTOS concepts that will be used in my design.

Functional Spec:

This minimal firmware test will:

Using FreeRTOS to execute the tasks required by this application
The following tasks are used:
CAPSENSETm task: Initializes the CAPSENSETm hardware block, processes the touch input, and sends a command to the LED task to update the LED status.
Modified to:
uses the logic from the CAPSENSE to send a command to the new print task
only processes the button touch input.
Use the logic that  sends a command to the LED task to update the LED status to send a message to the PrintMSG TASK
LED task: Not USED 
Initializes the TCPWM in PWM mode for driving the LED, and updates the status of the LED based on the received command.
PrintMSG TASK: NEW TASK to generate a message
uses the logic from the LED task to print messages to the terminal.
 Receives messages from other TASK


A FreeRTOS-based timer is used for making the CAPSENSETm scan periodic; a queue is used for communication between the CAPSENSETm task and other task. FreeRTOSConfig.h contains the FreeRTOS settings and configuration.
Task priority levels
capsense=MAX-1
print=MAX-2
Technical  Spec:

library(s) used. 

MAIN.c

#include "cybsp.h"
#include "FreeRTOS.h"
#include "task.h"
#include "queue.h"
#include "capsense_task.h"
#include "printmsg_task.h"

capsense_task.c

#include "capsense_task.h"
#include "cybsp.h"
#include "cyhal.h"
#include "cycfg.h"
#include "cycfg_capsense.h"
#include "FreeRTOS.h"
#include "task.h"
#include "queue.h"
#include "timers.h"
#include "printmsg_task.h"
#include "string.h"

printmsg_task.com

#include "printmsg_task.h"
#include "cybsp.h"
#include "cyhal.h"
#include "FreeRTOS.h"
#include "task.h"
#include "queue.h"
#include "cycfg.h"
#include "cy_retarget_io.h"



API Functions Used  (HAL or PDL)

main.c
cybsp_init()
  __enable_irq()
QueueCreate()
xTaskCreate()()
vTaskStartScheduler()
capsense_task.c
xTimerCreate()
xTimerStart()
xQueueReceive()
Cy_CapSense_IsBusy()
Cy_CapSense_ScanAllWidgets()
Cy_CapSense_ProcessAllWidgets()
Cy_CapSense_IsSensorActive()
xQueueSendToBack()
Cy_CapSense_Init()
Cy_CapSense_Enable()
Cy_CapSense_Wakeup()
Cy_CapSense_InterruptHandler()
printmsg_task.com
cy_retarget_io_init()
 xQueueReceive()
printf(()

MAIN.c

Set task priority
capsense=high-1
print=high-2
set stack sizes for task
capsense=256
print=minimal_stack_size
set Queue lengths of message queues used
Initialize the device and board peripherals
Enable global interrupts
Create the queues.
print_command_data_q
capsense_command_q
Create the user tasks. calls to xTaskCreate()
 "CapSense Task"
" PrintMsg Task
Start the RTOS scheduler. This function should never return 
-vTaskStartScheduler();
end of main


CAPSENSE TASK 

Initialize timer for periodic CapSense scan 
Initialize CapSense block 
Start the timer */
LOOP
Block until a CapSense command has been received over queue
received command?
CAPSENSE_SCAN  then call ScanAllWidgets
When the scan is finished it fires the Scan Callback function (see below)
CAPSENSE_PROCESS  then call ProcessAllWidgets
call process_touch
Determine which CAPESENSE button was pushed?
Construct a message with the button that was pressed.
Send the message to the PRINTMSG QUeue to be read by the PrintMSG task
When the timer goes OFF then the TIMER Callback: 
sends a CAPSENSE_SCAN command to the CAPSENSE Command Queue.
If a scan ends the scan callback is called
this sends a CAPSENSE_PROCESS command to the CAPSENSE Command Queue


PRINTMSG TASK

Initialize the debug uart
Print  opening message
LOOP
Block until a command has been received over queue
If a command is received 
check the command
if PRINT_MESSAGE is the command print the message of the command to the terminal.
CODE

I started with the example project CAPSENSE_Buttons_and_Slider_FreeRTOS and copied the project to Test-Minimal_FreeRTOS_task1_CAPSENSE_Buttons_task2_printmsg
Code a new print task printmgs.c  and printmgs.h
then I modified the capsense source file to only use the buttons. and send a print message to a newly developed  print task. 
I had to send the print comand from the CAPSense Task to the Print Tas
WRAPUP

The takeaways from this test are:

FreeRTOS task
Creating a new FreeRTOS schedular task
Task to Task communication using queues.


What follows is the README for the Example firmware 

# PSoC&trade; 6 MCU: CAPSENSE&trade; buttons and slider (FreeRTOS)

This code example features a 5-segment linear slider and two CAPSENSE&trade; buttons. Button 0 turns the LED ON, Button 1 turns the LED OFF, and the slider controls the brightness of the LED. The code example also demonstrates interfacing with the CAPSENSE&trade; Tuner using the I2C interface.

[Provide feedback on this code example.](https://cypress.co1.qualtrics.com/jfe/form/SV_1NTns53sK2yiljn?Q_EED=eyJVbmlxdWUgRG9jIElkIjoiQ0UyMjcxMjciLCJTcGVjIE51bWJlciI6IjAwMi0yNzEyNyIsIkRvYyBUaXRsZSI6IlBTb0MmdHJhZGU7IDYgTUNVOiBDQVBTRU5TRSZ0cmFkZTsgYnV0dG9ucyBhbmQgc2xpZGVyIChGcmVlUlRPUykiLCJyaWQiOiJzZGFrIiwiRG9jIHZlcnNpb24iOiI0LjAuMCIsIkRvYyBMYW5ndWFnZSI6IkVuZ2xpc2giLCJEb2MgRGl2aXNpb24iOiJNQ0QiLCJEb2MgQlUiOiJJQ1ciLCJEb2MgRmFtaWx5IjoiUFNPQyJ9)


## Requirements

- [ModusToolbox&trade; software](https://www.infineon.com/modustoolbox) v3.0 or later (tested with v3.0)
- PSoC&trade; 6 Board support package (BSP) minimum required version: 4.0.0
- Programming language: C
- Associated parts: All [PSoC&trade; 6 MCU](https://www.cypress.com/PSoC6) parts, [AIROC&trade; CYW20735 Bluetooth&reg; & Bluetooth&reg; LE system on chip](https://www.infineon.com/cms/en/product/wireless-connectivity/airoc-bluetooth-le-bluetooth-multiprotocol/), [AIROC™ CYW20819 Bluetooth&reg; & Bluetooth&reg; LE system on chip](https://www.infineon.com/cms/en/product/wireless-connectivity/airoc-bluetooth-le-bluetooth-multiprotocol/airoc-bluetooth-le-bluetooth/cyw20819/?redirId=204689), [AIROC&trade; CYW43012 Wi-Fi & Bluetooth&reg; combo chip](https://www.infineon.com/cms/en/product/wireless-connectivity/airoc-wi-fi-plus-bluetooth-combos/cyw43012/), [AIROC&trade; CYW4343W Wi-Fi & Bluetooth&reg; combo chip](https://www.infineon.com/cms/en/product/wireless-connectivity/airoc-wi-fi-plus-bluetooth-combos/cyw4343w/), [AIROC&trade; CYW4373 Wi-Fi & Bluetooth&reg; combo chip](https://www.infineon.com/cms/en/product/wireless-connectivity/airoc-wi-fi-plus-bluetooth-combos/cyw4373/), [AIROC&trade; CYW43439 Wi-Fi & Bluetooth&reg; combo chip](https://www.infineon.com/cms/en/product/wireless-connectivity/airoc-wi-fi-plus-bluetooth-combos/cyw43439/)


## Supported toolchains (make variable 'TOOLCHAIN')

- GNU Arm&reg; embedded compiler v10.3.1 (`GCC_ARM`) - Default value of `TOOLCHAIN`
- Arm&reg; compiler v6.16 (`ARM`)
- IAR C/C++ compiler v9.30.1 (`IAR`)


## Supported kits (make variable 'TARGET')

- [PSoC&trade; 6 Wi-Fi Bluetooth&reg; prototyping kit](https://www.infineon.com/CY8CPROTO-062-4343W) (`CY8CPROTO-062-4343W`) – Default value of `TARGET`
- [PSoC&trade; 6 Wi-Fi Bluetooth&reg; pioneer kit](https://www.infineon.com/CY8CKIT-062-WIFI-BT) (`CY8CKIT-062-WIFI-BT`)
- [PSoC&trade; 6 Bluetooth&reg; LE pioneer kit](https://www.infineon.com/CY8CKIT-062-BLE) (`CY8CKIT-062-BLE`)
- [PSoC&trade; 62S2 Wi-Fi Bluetooth&reg; pioneer kit](https://www.infineon.com/CY8CKIT-062S2-43012) (`CY8CKIT-062S2-43012`)
- [PSoC&trade; 62S1 Wi-Fi Bluetooth&reg; pioneer kit](https://www.infineon.com/CYW9P62S1-43438EVB-01) (`CYW9P62S1-43438EVB-01`)
- [PSoC&trade; 62S1 Wi-Fi Bluetooth&reg; pioneer kit](https://www.infineon.com/CYW9P62S1-43012EVB-01) (`CYW9P62S1-43012EVB-01`)
- [PSoC&trade; 62S3 Wi-Fi Bluetooth&reg; prototyping kit](https://www.infineon.com/CY8CPROTO-062S3-4343W) (`CY8CPROTO-062S3-4343W`)
- [PSoC&trade; 64 "Secure Boot" Wi-Fi Bluetooth&reg; pioneer kit](https://www.infineon.com/CY8CKIT-064B0S2-4343W) (`CY8CKIT-064B0S2-4343W`)
- [PSoC&trade; 62S4 pioneer kit](https://www.infineon.com/CY8CKIT-062S4) (`CY8CKIT-062S4`)
- [PSoC&trade; 62S2 evaluation kit](https://www.infineon.com/CY8CEVAL-062S2) (`CY8CEVAL-062S2`, `CY8CEVAL-062S2-LAI-4373M2`, `CY8CEVAL-062S2-MUR-43439M2`)

## Hardware setup

This example uses the board's default configuration. See the kit user guide to ensure that the board is configured correctly.

**Note:** The PSoC&trade; 6 Bluetooth&reg; LE pioneer kit (CY8CKIT-062-BLE) and the PSoC&trade; 6 Wi-Fi Bluetooth&reg; pioneer kit (CY8CKIT-062-WIFI-BT) ship with KitProg2 installed. The ModusToolbox&trade; software requires KitProg3. Before using this code example, make sure that the board is upgraded to KitProg3. The tool and instructions are available in the [Firmware Loader](https://github.com/Infineon/Firmware-loader) GitHub repository. If you do not upgrade, you will see an error like "unable to find CMSIS-DAP device" or "KitProg firmware is out of date".


## Software setup

This example requires no additional software or tools.


## Using the code example

Create the project and open it using one of the following:

<details><summary><b>In Eclipse IDE for ModusToolbox&trade; software</b></summary>

1. Click the **New Application** link in the **Quick Panel** (or, use **File** > **New** > **ModusToolbox&trade; Application**). This launches the [Project Creator](https://www.infineon.com/ModusToolboxProjectCreator) tool.

2. Pick a kit supported by the code example from the list shown in the **Project Creator - Choose Board Support Package (BSP)** dialog.

   When you select a supported kit, the example is reconfigured automatically to work with the kit. To work with a different supported kit later, use the [Library Manager](https://www.infineon.com/ModusToolboxLibraryManager) to choose the BSP for the supported kit. You can use the Library Manager to select or update the BSP and firmware libraries used in this application. To access the Library Manager, click the link from the **Quick Panel**.

   You can also just start the application creation process again and select a different kit.

   If you want to use the application for a kit not listed here, you may need to update the source files. If the kit does not have the required resources, the application may not work.

3. In the **Project Creator - Select Application** dialog, choose the example by enabling the checkbox.

4. (Optional) Change the suggested **New Application Name**.

5. The **Application(s) Root Path** defaults to the Eclipse workspace which is usually the desired location for the application. If you want to store the application in a different location, you can change the *Application(s) Root Path* value. Applications that share libraries should be in the same root path.

6. Click **Create** to complete the application creation process.

For more details, see the [Eclipse IDE for ModusToolbox&trade; software user guide](https://www.infineon.com/MTBEclipseIDEUserGuide) (locally available at *{ModusToolbox&trade; software install directory}/docs_{version}/mt_ide_user_guide.pdf*).

</details>

<details><summary><b>In command-line interface (CLI)</b></summary>

ModusToolbox&trade; software provides the Project Creator as both a GUI tool and the command line tool, "project-creator-cli". The CLI tool can be used to create applications from a CLI terminal or from within batch files or shell scripts. This tool is available in the *{ModusToolbox&trade; software install directory}/tools_{version}/project-creator/* directory.

Use a CLI terminal to invoke the "project-creator-cli" tool. On Windows, use the command line "modus-shell" program provided in the ModusToolbox&trade; software installation instead of a standard Windows command-line application. This shell provides access to all ModusToolbox&trade; software tools. You can access it by typing `modus-shell` in the search box in the Windows menu. In Linux and macOS, you can use any terminal application.

The "project-creator-cli" tool has the following arguments:

Argument | Description | Required/optional
---------|-------------|-----------
`--board-id` | Defined in the `<id>` field of the [BSP](https://github.com/Infineon?q=bsp-manifest&type=&language=&sort=) manifest | Required
`--app-id`   | Defined in the `<id>` field of the [CE](https://github.com/Infineon?q=ce-manifest&type=&language=&sort=) manifest | Required
`--target-dir`| Specify the directory in which the application is to be created if you prefer not to use the default current working directory | Optional
`--user-app-name`| Specify the name of the application if you prefer to have a name other than the example's default name | Optional

<br />

The following example clones the "[mtb-example-buttons-slider-freertos](https://github.com/Infineon/mtb-example-psoc6-capsense-buttons-slider-freertos)" application with the desired name "ButtonSliderFreeRtos" configured for the *CY8CPROTO-062-4343W* BSP into the specified working directory, *C:/mtb_projects*:

   ```
   project-creator-cli --board-id CY8CKIT-062-WIFI-BT --app-id mtb-example-psoc6-capsense-buttons-slider-freertos --user-app-name ButtonSliderFreeRtos --target-dir "C:/mtb_projects"
   ```

**Note:** The project-creator-cli tool uses the `git clone` and `make getlibs` commands to fetch the repository and import the required libraries. For details, see the "Project creator tools" section of the [ModusToolbox&trade; software user guide](https://www.infineon.com/ModusToolboxUserGuide) (locally available at *{ModusToolbox&trade; software install directory}/docs_{version}/mtb_user_guide.pdf*).

</details>

<details><summary><b>In third-party IDEs</b></summary>

Use one of the following options:

- **Use the standalone [Project Creator](https://www.infineon.com/ModusToolboxProjectCreator) tool:**

   1. Launch Project Creator from the Windows Start menu or from *{ModusToolbox&trade; software install directory}/tools_{version}/project-creator/project-creator.exe*.

   2. In the initial **Choose Board Support Package** screen, select the BSP, and click **Next**.

   3. In the **Select Application** screen, select the appropriate IDE from the **Target IDE** drop-down menu.

   4. Click **Create** and follow the instructions printed in the bottom pane to import or open the exported project in the respective IDE.

<br />

- **Use command-line interface (CLI):**

   1. Follow the instructions from the **In command-line interface (CLI)** section to create the application.

   2. Export the application to a supported IDE using the `make <ide>` command.

   3. Follow the instructions displayed in the terminal to create or import the application as an IDE project.

For a list of supported IDEs and more details, see the "Exporting to IDEs" section of the [ModusToolbox&trade; software user guide](https://www.infineon.com/ModusToolboxUserGuide) (locally available at *{ModusToolbox&trade; software install directory}/docs_{version}/mtb_user_guide.pdf*).

</details>


## Operation


If using a PSoC&trade; 64 "Secure" MCU kit (like CY8CKIT-064B0S2-4343W), the PSoC&trade; 64 device must be provisioned with keys and policies before being programmed. Follow the instructions in the ["Secure Boot" SDK user guide](https://www.infineon.com/dgdlac/Infineon-PSoC_64_Secure_MCU_Secure_Boot_SDK_User_Guide-Software-v07_00-EN.pdf?fileId=8ac78c8c7d0d8da4017d0f8c361a7666&utm_source=cypress&utm_medium=referral&utm_campaign=202110_globe_en_all_integration-software) to provision the device. If the kit is already provisioned, copy-paste the keys and policy folder to the application folder.

1. Connect the board to your PC using the provided USB cable through the KitProg3 USB connector.

2. Program the board using one of the following:

   <details><summary><b>Using Eclipse IDE for ModusToolbox&trade; software</b></summary>

      1. Select the application project in the Project Explorer.

      2. In the **Quick Panel**, scroll down, and click **\<Application Name> Program (KitProg3_MiniProg4)**.
   </details>

   <details><summary><b>Using CLI</b></summary>

     From the terminal, execute the `make program` command to build and program the application using the default toolchain to the default target. The default toolchain and target are specified in the application's Makefile but you can override those values manually:
      ```
      make program TARGET=<BSP> TOOLCHAIN=<toolchain>
      ```

      Example:
      ```
      make program TARGET=CY8CPROTO-062-4343W TOOLCHAIN=GCC_ARM
      ```
   </details>

3. After programming, the application starts automatically. Confirm that the user LED is glowing.

4. To test the application, touch CAPSENSE&trade; button 1 (BTN1) to turn the LED OFF, touch CAPSENSE&trade; Button 0 (BTN0) to turn the LED ON, and touch the slider in different positions to change the brightness.

### Monitor data using CAPSENSE&trade; tuner

You can also monitor the CAPSENSE&trade; data using the CAPSENSE&trade; Tuner application as follows:

1. Open CAPSENSE&trade; tuner from the IDE Quick Panel.

   You can also run the CAPSENSE&trade; Tuner application standalone from *{ModusToolbox&trade; install directory}/ModusToolbox/tools_{version}/capsense-configurator/capsense-tuner*. In this case, after opening the application, select **File** > **Open** and open the *design.cycapsense* file for the respective kit, which is present in the *bsp/TARGET_<BSP-NAME>/config* folder.

2. Ensure that the kit is in KitProg3 mode. See [Firmware-loader](https://github.com/Infineon/Firmware-loader) to learn how to update the firmware and switch to KitProg3 mode.

3. In the Tuner application, click **Tuner Communication Setup** or select **Tools** > **Tuner Communication Setup**. In the window that appears, select the I2C checkbox under KitProg3 and configure as follows:
   ```
   I2C Address: 8
   Sub-address: 2-Bytes
   Speed (kHz): 400
   ```

4. Click **Connect** or select **Communication** > **Connect**.

5. Click **Start** or select **Communication** > **Start**.

Under the **Widget View** tab, you can see the corresponding widgets highlighted in blue color when you touch the button or slider. You can also view the sensor data in the **Graph View** tab. For example, to view the sensor data for Button 0, select **Button0_Rx0** under **Button0**.

**Figure 1** shows the CAPSENSE&trade; Tuner displaying the status of CAPSENSE&trade; touch on Button 0 and LinearSlider 0.

**Figure 1. CAPSENSE&trade; Tuner showing touch data**

![](images/tuner_display.png)

CAPSENSE&trade; Tuner can also be used for CAPSENSE&trade; parameter tuning and measuring signal-to-noise ratio (SNR). See the [ModusToolbox&trade; CAPSENSE&trade; Tuner Guide](https://www.cypress.com/ModusToolboxCapSenseTuner) (**Help** > **View Help**) and [AN85951 – PSoC&trade; 4 and PSoC&trade; 6 MCU CAPSENSE&trade; Design Guide](https://www.cypress.com/an85951) for more details on selecting the tuning parameters.

## Debugging

You can debug the example to step through the code. In the IDE, use the **\<Application Name> Debug (KitProg3_MiniProg4)** configuration in the **Quick Panel**. For details, see the "Program and debug" section in the [Eclipse IDE for ModusToolbox&trade; user guide](https://www.infineon.com/MTBEclipseIDEUserGuide).

**Note:** **(Only while debugging)** On the CM4 CPU, some code in `main()` may execute before the debugger halts at the beginning of `main()`. This means that some code executes twice – once before the debugger stops execution, and again after the debugger resets the program counter to the beginning of `main()`. See [KBA231071](https://community.infineon.com/docs/DOC-21143) to learn about this and for the workaround.


## Design and implementation

In this project, PSoC&trade; 6 MCU scans a self-capacitance (CSD) based 5-element CAPSENSE&trade; slider, and two mutual capacitance (CSX) CAPSENSE&trade; buttons for user input. The project uses the [CAPSENSE&trade; Middleware](https://github.com/Infineon/capsense) (see [ModusToolbox&trade; user guide](http://www.infineon.com/ModusToolboxUserGuide) for details on selecting a middleware).

See [AN85951 – PSoC&trade; 4 and PSoC&trade; 6 MCU CAPSENSE&trade; design guide](https://www.infineon.com/AN85951) for details on CAPSENSE&trade; features and usage.

In this application, the state of the user LED is controlled based on user inputs provided using the CAPSENSE&trade; buttons and slider. A PWM HAL resource is configured for controlling the brightness of the LED. Touching Button 0 turns the LED ON, and Button 1 turns the LED OFF. The brightness of the LED is set based on the touch position on the CAPSENSE&trade; slider.

The [ModusToolbox&trade; CAPSENSE&trade; configurator tool guide](https://www.infineon.com/ModusToolboxCapSenseConfig) provides step-by-step instructions on how to configure CAPSENSE&trade; in the application. The CAPSENSE&trade; Configurator and Tuner tools can be launched from the CSD personality in the Device Configurator tool.

The application uses an [EZI2C HAL](https://infineon.github.io/mtb-hal-cat1/html/group__group__hal__ezi2c.html) interface for communicating with the CAPSENSE&trade; tuner.

The firmware uses FreeRTOS to execute the tasks required by this application. The following tasks are created:

1. **CAPSENSE&trade; task:** Initializes the CAPSENSE&trade; hardware block, processes the touch input, and sends a command to the LED task to update the LED status.

2. **LED task:** Initializes the TCPWM in PWM mode for driving the LED, and updates the status of the LED based on the received command.

A FreeRTOS-based timer is used for making the CAPSENSE&trade; scan periodic; a queue is used for communication between the CAPSENSE&trade; task and LED task. *FreeRTOSConfig.h* contains the FreeRTOS settings and configuration.

## Operation at Custom Power Supply Voltages

The application is configured to work with the default operating voltage of the kit.

**Table 1. Operating voltages supported by the kits**

 Kit                   | Supported operating voltages | Default operating voltage
 :-------------------- | ---------------------------- | -------------
 CY8CPROTO-062-4343W   | 3.3 V/1.8 V                | 3.3 V
 CY8CKIT-062-BLE       | 3.3 V/1.8 V                | 3.3 V
 CY8CKIT-062-WIFI-BT   | 3.3 V/1.8 V                | 3.3 V
 CY8CKIT-062S2-43012   | 3.3 V/1.8 V                | 3.3 V
 CYW9P62S1-43438EVB-01 | 3.3-V Only                   | 3.3 V
 CYW9P62S1-43012EVB-01 | 1.8-V Only                   | 1.8 V
 CY8CPROTO-062S3-4343W | 3.3 V/1.8 V                | 3.3 V
 CY8CKIT-064B0S2-4343W | 3.3 V/1.8 V                | 3.3 V
 CY8CKIT-062S4         | 3.3 V/1.8 V                | 3.3 V
 CY8CEVAL-062S2        | 3.3 V/1.8 V                | 3.3 V



For kits that support multiple operating voltages, the default BSP configuration is provided by the *design.modus* file should be customized. Follow the instructions to use the example at a custom power supply, such as 1.8 V:

1. Create a directory at the root of the application to hold any custom BSP configuration files. *\<application_folder>/templates*.

2. Create a subdirectory for each target that needs to be modified to work at a custom power supply. *\<application_folder>/templates/TARGET_<BSP-NAME>/config*.

3. Copy the *design.modus* file and other configuration files (from the path *bsp/TARGET_<BSP-NAME>/config*) and paste them into the new directory for the target.

4. Launch the [Device Configurator](https://www.infineon.com/ModusToolboxDeviceConfig) tool using the Quick Panel link in the IDE.
   This opens the *design.modus* file from the newly created *templates/TARGET_<BSP-NAME>/config* folder; and you are now free to customize the configuration as required.

5. Update the *Operating Conditions* parameters in Power Settings with the desired voltage and select **File** > **Save**.

   **Figure 2. Power settings to work with 1.8 V**

   ![](images/design_modus_custom_power_settings.png)

6. Change the jumper/switch setting as follows:

   **Table 2. Jumper/switch position for 1.8-V operation**

   Kit                   | Jumper/switch position
   :-------------------- | ----------------------
   CY8CPROTO-062-4343W   | J3 (1-2) 
   CY8CKIT-062-BLE       | SW5 (1-2)
   CY8CKIT-062-WIFI-BT   | SW5 (1-2)
   CY8CKIT-062S2-43012   | J14 (1-2)
   CY8CPROTO-062S3-4343W | J3 (1-2) 
   CY8CKIT-064B0S2-4343W | J14 (1-2)
   CY8CKIT-062S4         | J12 (1-2)
   CY8CEVAL-062S2        | J18 (1-2)

7. Re-build and program the application to evaluate the application at the new power setting.

### Resources and settings

**Table 3. Application resources**

 Resource  |  Alias/object     |    Purpose
 :-------- | :-------------    | :------------
 GPIO (HAL)    | CYBSP_USER_LED     | User LED to show visual output
 PWM (HAL)     | pwm_led       | PWM HAL object used to vary LED brightness
 EZI2C (HAL)   | sEzI2C        |  Slave EZI2C object used to tune CAPSENSE&trade;
<br>

## Related resources


Resources  | Links
-----------|----------------------------------
Application notes  | [AN228571](https://www.infineon.com/AN228571) – Getting started with PSoC&trade; 6 MCU on ModusToolbox&trade; software <br>  [AN215656](https://www.infineon.com/AN215656) – PSoC&trade; 6 MCU: Dual-CPU system design <br>  [AN234334](https://www.cypress.com/AN234334) – Getting started with XMC&trade; 7000 MCU on ModusToolbox&trade;
Code examples  | [Using ModusToolbox&trade; software](https://github.com/Infineon/Code-Examples-for-ModusToolbox-Software) on GitHub <br> [Using PSoC&trade; Creator](https://www.infineon.com/cms/en/design-support/software/code-examples/psoc-3-4-5-code-examples-for-psoc-creator)
Device documentation | [PSoC&trade; 6 MCU datasheets](https://documentation.infineon.com/html/psoc6/bnm1651211483724.html) <br> [PSoC&trade; 6 technical reference manuals](https://documentation.infineon.com/html/psoc6/zrs1651212645947.html)
Development kits | Select your kits from the [evaluation board finder](https://www.infineon.com/cms/en/design-support/finder-selection-tools/product-finder/evaluation-board)
Libraries on GitHub  | [mtb-pdl-cat1](https://github.com/Infineon/mtb-pdl-cat1) – PSoC&trade; 6 peripheral driver library (PDL)  <br> [mtb-hal-cat1](https://github.com/Infineon/mtb-hal-cat1) – Hardware abstraction layer (HAL) library <br> [retarget-io](https://github.com/Infineon/retarget-io) – Utility library to retarget STDIO messages to a UART port
Middleware on GitHub  | [capsense](https://github.com/Infineon/capsense) – CAPSENSE&trade; library and documents <br> [psoc6-middleware](https://github.com/Infineon/modustoolbox-software#psoc-6-middleware-libraries) – Links to all PSoC&trade; 6 MCU middleware
Tools  | [Eclipse IDE for ModusToolbox&trade; software](https://www.infineon.com/cms/en/design-support/tools/sdk/modustoolbox-software/) – ModusToolbox&trade; software is a collection of easy-to-use software and tools enabling rapid development with Infineon MCUs, covering applications from embedded sense and control to wireless and cloud-connected systems using AIROC&trade; Wi-Fi and Bluetooth® connectivity devices.

<br />

## Other resources

Infineon provides a wealth of data at www.infineon.com to help you select the right device, and quickly and effectively integrate it into your design.

For PSoC&trade; 6 MCU devices, see [How to design with PSoC&trade; 6 MCU – KBA223067](https://community.infineon.com/t5/Knowledge-Base-Articles/How-to-Design-with-PSoC-6-MCU-KBA223067/ta-p/248857) in the Infineon community.

## Document history

Document Title: *CE227127* - *PSoC&trade; 6 MCU: CAPSENSE&trade; Buttons and Slider (FreeRTOS)*

 Version | Description of change
 ------- | ---------------------
 1.0.0   | New code example
 1.1.0     | Fixed miscapitalization of "FreeRTOS.h" include
 1.2.0     | Added Support for CY8CPROTO-062S3-4343W Kit <br> Updated FreeRTOS Heap Allocation Scheme <br> Updated to use EzI2C HAL for CAPSENSE&trade; Tuner interface <br> Updated to support ModusToolbox&trade; software v2.1
 2.0.0     | Major update to support ModusToolbox&trade; software v2.2, added support for new kits<br> This version is not backward compatible with ModusToolbox&trade; software v2.1
 2.1.0     | Updated to support FreeRTOS v10.3.1
 3.0.0     | Updated to support ModusToolbox&trade; software v2.4 <br> Added support for new kits <br> Updated the BSPs to v3.X
 4.0.0   | Major update to support ModusToolbox&trade; v3.0 and BSPs v4.X. This version is not backward compatible with previous versions of ModusToolbox&trade;

<br>

---------------------------------------------------------

© Cypress Semiconductor Corporation, 2020-2022. This document is the property of Cypress Semiconductor Corporation, an Infineon Technologies company, and its affiliates ("Cypress").  This document, including any software or firmware included or referenced in this document ("Software"), is owned by Cypress under the intellectual property laws and treaties of the United States and other countries worldwide.  Cypress reserves all rights under such laws and treaties and does not, except as specifically stated in this paragraph, grant any license under its patents, copyrights, trademarks, or other intellectual property rights.  If the Software is not accompanied by a license agreement and you do not otherwise have a written agreement with Cypress governing the use of the Software, then Cypress hereby grants you a personal, non-exclusive, nontransferable license (without the right to sublicense) (1) under its copyright rights in the Software (a) for Software provided in source code form, to modify and reproduce the Software solely for use with Cypress hardware products, only internally within your organization, and (b) to distribute the Software in binary code form externally to end users (either directly or indirectly through resellers and distributors), solely for use on Cypress hardware product units, and (2) under those claims of Cypress’s patents that are infringed by the Software (as provided by Cypress, unmodified) to make, use, distribute, and import the Software solely for use with Cypress hardware products.  Any other use, reproduction, modification, translation, or compilation of the Software is prohibited.
<br />
TO THE EXTENT PERMITTED BY APPLICABLE LAW, CYPRESS MAKES NO WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, WITH REGARD TO THIS DOCUMENT OR ANY SOFTWARE OR ACCOMPANYING HARDWARE, INCLUDING, BUT NOT LIMITED TO, THE IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE.  No computing device can be absolutely secure.  Therefore, despite security measures implemented in Cypress hardware or software products, Cypress shall have no liability arising out of any security breach, such as unauthorized access to or use of a Cypress product. CYPRESS DOES NOT REPRESENT, WARRANT, OR GUARANTEE THAT CYPRESS PRODUCTS, OR SYSTEMS CREATED USING CYPRESS PRODUCTS, WILL BE FREE FROM CORRUPTION, ATTACK, VIRUSES, INTERFERENCE, HACKING, DATA LOSS OR THEFT, OR OTHER SECURITY INTRUSION (collectively, "Security Breach").  Cypress disclaims any liability relating to any Security Breach, and you shall and hereby do release Cypress from any claim, damage, or other liability arising from any Security Breach.  In addition, the products described in these materials may contain design defects or errors known as errata which may cause the product to deviate from published specifications. To the extent permitted by applicable law, Cypress reserves the right to make changes to this document without further notice. Cypress does not assume any liability arising out of the application or use of any product or circuit described in this document. Any information provided in this document, including any sample design information or programming code, is provided only for reference purposes.  It is the responsibility of the user of this document to properly design, program, and test the functionality and safety of any application made of this information and any resulting product.  "High-Risk Device" means any device or system whose failure could cause personal injury, death, or property damage.  Examples of High-Risk Devices are weapons, nuclear installations, surgical implants, and other medical devices.  "Critical Component" means any component of a High-Risk Device whose failure to perform can be reasonably expected to cause, directly or indirectly, the failure of the High-Risk Device, or to affect its safety or effectiveness.  Cypress is not liable, in whole or in part, and you shall and hereby do release Cypress from any claim, damage, or other liability arising from any use of a Cypress product as a Critical Component in a High-Risk Device. You shall indemnify and hold Cypress, including its affiliates, and its directors, officers, employees, agents, distributors, and assigns harmless from and against all claims, costs, damages, and expenses, arising out of any claim, including claims for product liability, personal injury or death, or property damage arising from any use of a Cypress product as a Critical Component in a High-Risk Device. Cypress products are not intended or authorized for use as a Critical Component in any High-Risk Device except to the limited extent that (i) Cypress’s published data sheet for the product explicitly states Cypress has qualified the product for use in a specific High-Risk Device, or (ii) Cypress has given you advance written authorization to use the product as a Critical Component in the specific High-Risk Device and you have signed a separate indemnification agreement.
<br />
Cypress, the Cypress logo, and combinations thereof, WICED, ModusToolbox, PSoC, CapSense, EZ-USB, F-RAM, and Traveo are trademarks or registered trademarks of Cypress or a subsidiary of Cypress in the United States or in other countries. For a more complete list of Cypress trademarks, visit cypress.com. Other names and brands may be claimed as property of their respective owners.
