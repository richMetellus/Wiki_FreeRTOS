.. include:: /global.rst 
   :start-after: _RTOSLinksStart:
   :end-before: _RTOSLinksEnd


#################################################
Intro to RTOS Part 2: FreeRTOS
#################################################

******************************
Agenda and Description
******************************

This is my notes on FreeRTOS series done by Shawn Hymmel from Digi-key and many 
others. I combined all the lessons I come across here as they suit my needs.

This is a continuation of :ref:`Intro to free RTOS <RTOSIntro>`.

**Required Hardware**

1. Development board: Not there are many board variants of the ESP32

  * Shawn from Digikey uses `Adafruit Feather HUZZAH32. <https://www.digikey.com/en/products/detail/adafruit-industries-llc/3405/7244967>`_

  * I am using Teyleten Robot ESP32S uino (ESP32 30P, 3PCS)

    * I got mine on `Amazon (3 Pcs) <https://www.amazon.com/dp/B08246MCL5?psc=1&ref=ppx_yo2ov_dt_b_product_details>`_

      * `a cheaper site to get it <https://s.click.aliexpress.com/e/_9xJakd>`_

    * Product barcode: B08246MCL5
    * Brand: Teyleten Robot
    * My board is the 30 Pins instead of the 38 Pins
    * ESP-WROOM-32
    * The comment FAQ section indicated it should be compatible with Arduino IDE


*******************************
What is FreeRTOS [1]_
*******************************


* FreeRTOS is a real-time kernel (or real-time scheduler) on top of which 
  embedded applications can be built to meet their hard real-time requirements; 

* small enough to run on a microcontroller - although its use is not limited to 
  microcontroller applications. [5]_

* It allows applications to be organized as a collection of independent threads 
  of execution; 

* Traditional real time schedulers, such as the scheduler used in FreeRTOS, 
  achieve determinism by allowing the user to assign a priority to each thread 
  of execution [5]_

* The kernel (scheduler) decides which thread should be executing by examining the priority 
  assigned to each thread by the application designer; 

* In the simplest case, the application designer could assign higher priorities 
  to threads that implement hard real-time requirements, and lower priorities 
  to threads that implement soft real-time requirements

    * **Soft real-time requirements** are those that state a time deadline, 
      but breaching the deadline would not render the system useless; 
    
    * **Hard real-time requirements** are those that state a time deadline, 
      and breaching the deadline would result in absolute failure of the system

* FreeRTOS is free and open-source. Amazon took control of the project in 2017.

******************
Setup The Host
******************

FreeRTOS Setup
=================

Here are the basics steps to get started with FreeRTOS.
This steps are just to explore what FreeRTOS.

1. Go to https://www.freertos.org/ and click on ``Download FreeRTOS``
2. At the time of this writing, I selected the FreeRTOS 202212.00 that 
   contains the following:
   
   * FreeRTOS Kernel
   * FreeRTOS-Plus libraries
   * AWS IoT libraries
   * and example projects

3. `Getting Started <https://www.freertos.org/FreeRTOS-quick-start-guide.html>`_
   From the freeRTOS website, click on ``Kernel``, then jump
   to the ``Getting Started`` section. Follow the instructions.

    * there you will find resources how to get started, and how to create your
      own freeRTOS project if your board is not supported.
    
    * To include FreeRTOS in your project you need to include at minimum the 
      c and header files 

    1. Unzipped the freeRTOS folder.
       
       .. note::
          
          I ran into an error of ``path too long`` while extracting the zip file
          despite my windows system LongPath is enabled, so make sure to extract
          it to a short location drive such as "C:\LocalWorkspace"
       
       * Let's explore the extract folder:
         
         .. code-block:: bash
            :caption: The FreeRTOS Folder

            ricky-wsl@Rich-LenovExtX1:/mnt/c/LocalWorkspace/FreeRTOSv202212.00$ tree -pL 3
            [drwxrwxrwx]  .
            ├── [drwxrwxrwx]  FreeRTOS        # -----> This library is just a scheduler. This
            |                                 # contains the freeRTOS real-time Kernel source file
            │   ├── [drwxrwxrwx]  Demo        #. Contains the demo application projects
            │   ├── [drwxrwxrwx]  License
            │   ├── [-rwxrwxrwx]  README.md
            │   ├── [drwxrwxrwx]  Source      #. Contains the real time kernel source code
            │   │   ├── [-rwxrwxrwx]  CMakeLists.txt
            │   │   ├── [-rwxrwxrwx]  croutine.c  #. implements software co-routine functionality
            │   │   ├── [-rwxrwxrwx]  event_groups.c
            │   │   ├── [drwxrwxrwx]  include
            │   │   ├── [-rwxrwxrwx]  list.c
            │   │   ├── [-rwxrwxrwx]  manifest.yml
            │   │   ├── [drwxrwxrwx]  portable  #. Processor specific code.
            │   │   ├── [-rwxrwxrwx]  queue.c
            │   │   ├── [-rwxrwxrwx]  sbom.spdx
            │   │   ├── [-rwxrwxrwx]  stream_buffer.c
            │   │   ├── [-rwxrwxrwx]  tasks.c
            │   │   └── [-rwxrwxrwx]  timers.c  #. implements software timer functionality
            │   ├── [drwxrwxrwx]  Test
            │   └── [-rwxrwxrwx]  links_to_doc_pages_for_the_demo_projects.url
            ├── [-rwxrwxrwx]  FreeRTOS+TCP.url
            ├── [drwxrwxrwx]  FreeRTOS-Plus #--> this library is a few drivers and addon for the kernel
            |                               # like network driver TCP/UDP
            │   ├── [drwxrwxrwx]  Demo
            │   ├── [drwxrwxrwx]  Source
            │   ├── [drwxrwxrwx]  Test
            │   ├── [drwxrwxrwx]  ThirdParty
            │   ├── [drwxrwxrwx]  VisualStudio_StaticProjects
            │   └── [-rwxrwxrwx]  readme.txt
            ├── [-rwxrwxrwx]  GitHub-FreeRTOS-Home.url
            ├── [-rwxrwxrwx]  History.txt
            ├── [-rwxrwxrwx]  Quick_Start_Guide.url
            ├── [-rwxrwxrwx]  Upgrading-to-FreeRTOS.url
            ├── [-rwxrwxrwx]  lexicon.txt
            ├── [-rwxrwxrwx]  manifest.yml
            └── [drwxrwxrwx]  tools
                ├── [drwxrwxrwx]  aws_config_offline
                ├── [drwxrwxrwx]  aws_config_quick_start
                ├── [drwxrwxrwx]  cmock
                └── [-rwxrwxrwx]  uncrustify.cfg

         * The core RTOS code is contained in three files, which are 
            
            * FreeRTOS/Source/tasks.c, 
            * FreeRTOS/Source/queue.c and 
            * FreeRTOS/Source/list.c

The ESP32 does not run Vanilla FreeRTOS. Rather, it runs a modified version 
of FreeRTOS inside its ESP-IDF framework. See this reference [2]_ |VanillaFreeRTOSvsEsp-IDF|for more info.
Espressif modified FreeRTOS specifically for the ESP32 to support its SMP architecture.

* Read through this page |VanillaFreeRTOSvsEsp-IDF| to understand how the scheduler 
  can use the second core.

  * The digi-key series all demo example will only use one core for the tasks. 
    They should work in freeRTOS as well.


Install Serial Drivers for ESP32
=====================================

Before you can flash the ESP32, you will need to install some drivers.

The ESP32 module/chip has a built-in USB to UART controller on the chip so 
you don't need a USB to TTL controller to flash it but you'll need a driver 
so your host computer can recognize it.

**How to identify which driver to download**

1. You can find which chip your board is using by looking at its datasheet or
   by looking at the board itself.
   
   .. image:: /Operating_System/_images/FreeRTOS-0_esp32_38Pins.png
      :alt: credit Achim Pieters
      :width: 716px
      :height: 380px


2. Your chip typically will have one of these number printed on it. Download 
   the specific driver from the link listed based on your operating system.
   
   * `CH340 (Windows & Linux) <https://learn.sparkfun.com/tutorials/how-to-install-ch340-drivers/all>`_
   - `CH340 (macOS)  <https://github.com/adrianmihalko/ch340g-ch34g-ch34x-mac-os-x-driver>`_
   - `CP210x (Windows, macOS & Linux) <https://www.silabs.com/developers/usb-to-uart-bridge-vcp-drivers>`_
      
      * My esp32 board is using that one.

3. For the CP210x on windows, Do the following:
   
   * Extract the zip file, right click on silabser.inf and click on ``install``
     
     * This will prompt you to open

.. collapse:: error I got if I try to flash without installing the driver
   
   .. code-block:: bash
       
      ketch uses 235197 bytes (17%) of program storage space. Maximum is 1310720 bytes.
      Global variables use 21800 bytes (6%) of dynamic memory, leaving 305880 bytes for local variables. Maximum is 327680 bytes.
      esptool.py v4.2.1
      Serial port COM3
      Connecting......................................
      
      A fatal error occurred: Failed to connect to ESP32: No serial data received.
      For troubleshooting steps visit: https://docs.espressif.com/projects/esptool/en/latest/troubleshooting.html

Setup Arduino IDE for ESP32 [6]_
====================================

Here are the steps to set up arduino IDE for ESP32.

1. Go to the `arduino website <https://www.arduino.cc/en/software>`_
2. Download the non-web version, the standalone IDE version for your host OS
3. Install the IDE on your system
#. By default, Arduino IDE only support official arduino boards, but the IDE can
   be extended. Head over to `Arduino core for EPS32 github page <https://github.com/espressif/arduino-esp32>`_
#. Scroll down to the ReadME, installing and it will send you to this `site <https://docs.espressif.com/projects/arduino-esp32/en/latest/installing.html>`_
#. copy the stable release link. 
   
   * ``https://raw.githubusercontent.com/espressif/arduino-esp32/gh-pages/package_esp32_index.json``

#. Open up the arduino IDE, click on File --> Preference and paste the link
   on the ``Additional boards manager URLs`` bar and click on ``Ok``
   
   * you can add multiple link by using a comma

#. Install the board in the IDE by going to 
   
   #. ``Tools``--> ``Board``--> ``Board Manager`` This will refresh its index
   #. Search for ``esp32`` on the filter text input box, then install the
      ``esp32 by Espressif Systems``
   #. Pick the latest version, then click on ``INSTALL``
      
      * The arduino IDE will download all the necessary software.
      * The eps32 relies on settings in a FreeRTOS config file.

        * To view this file, head to wherever arduino keeps your board packages.
          In my case, on Windows it is under
          ``C:\Users\<uname>\AppData\Local\Arduino15\packages\esp32\hardware\esp32\<verx.y.z>\tools\sdk\esp32\include\freertos\include\esp_additions\freertos\FreeRTOSConfig.h``
        
        * This ``FreeRTOSConfig.h`` is a good way to see how FreeRTOS is configure for esp32.

          * There you can see there is at most 25 priorities level available.
            
            .. code-block:: c
               :lineno-start: 143
               
               /* This has impact on speed of search for highest priority */
               #define configMAX_PRIORITIES                            ( 25 )

*********************************
Write Your first application
*********************************

There are couple examples, I found to show how to blink the built-in LED of the
ESP32 board using FreeRTOS. Although I could have write my own but I am more
interested in following example and build an index than creating my own content.
Learn from the expert then use what you learn type of philosophy.

* The first example and simplest one is by this video |SimplyExplained_BlinkLEDEsp32nArduinoIDE|

  * this one doesn't call FreeRTOS function really.

* The 2nd is by Shawn Hymmel that blinks and LED using freeRTOS kernel to create
  2 tasks that blink the same LED at 2 different rate.


1. Write a different Arduino Sketch using either one of these files:

   .. tabs:: 
      
      .. tab:: BlinkySimple
         
         .. literalinclude:: ../../_resources/FreeRTOS/FreeRTOS_Tutorials/ESP32+ArduinoIDE/01_BlankESP32BuiltinLED/BlankLedWithDigitalWrite/BlankLedWithDigitalWrite.ino
            :language: c
            :linenos:
   
   
      .. tab:: Blinky2DifrntRates
         
         .. literalinclude:: ../../_resources/FreeRTOS/FreeRTOS_Tutorials/ESP32+ArduinoIDE/01_BlankESP32BuiltinLED/BlankLedAtDifrntRateUsing2Tasks/BlankLedAtDifrntRateUsing2Tasks.ino
            :language: c
            :linenos:

2. Program you board using a board that is compatible with your ESP32:

   * Go to ``Tools`` --> ``Board:`` and select the board

     * There are many variants available for ESP32. The most common and compatible
       ones are:
       
       * ``DOIT ESP32 DEVKIT V1``: if you can't find your board in this list then 
          try picking this one and seeing if it works 
         
         * My board was not listed so this one worked for me.
   
   * Make sure the correct port is also selected.
     
     * Go to ``Tools`` -> ``Port`` and select the correct serial port.
   
   * Click on the upload arrow and Voila
     
     .. collapse:: successful programming using COM5 and Board DOIT ESP32 Dev KIT 1
        
        .. code-block:: bash
           
           Sketch uses 235197 bytes (17%) of program storage space. Maximum is 1310720 bytes.
           Global variables use 21800 bytes (6%) of dynamic memory, leaving 305880 bytes for local variables. Maximum is 327680 bytes.
           esptool.py v4.2.1
           Serial port COM5
           Connecting....
           Chip is ESP32-D0WD (revision 1)
           Features: WiFi, BT, Dual Core, 240MHz, VRef calibration in efuse, Coding Scheme None
           WARNING: Detected crystal freq 41.01MHz is quite different to normalized freq 40MHz. Unsupported crystal in use?
           Crystal is 40MHz
           MAC: 94:3c:c6:38:02:78
           Uploading stub...
           Running stub...
           Stub running...
           Changing baud rate to 921600
           Changed.
           Configuring flash size...
           Flash will be erased from 0x00001000 to 0x00005fff...
           Flash will be erased from 0x00008000 to 0x00008fff...
           Flash will be erased from 0x0000e000 to 0x0000ffff...
           Flash will be erased from 0x00010000 to 0x00049fff...
           Compressed 17440 bytes to 12108...
           Writing at 0x00001000... (100 %)
           Wrote 17440 bytes (12108 compressed) at 0x00001000 in 0.4 seconds (effective 395.3 kbit/s)...
           Hash of data verified.
           Compressed 3072 bytes to 146...
           Writing at 0x00008000... (100 %)
           Wrote 3072 bytes (146 compressed) at 0x00008000 in 0.0 seconds (effective 517.0 kbit/s)...
           Hash of data verified.
           Compressed 8192 bytes to 47...
           Writing at 0x0000e000... (100 %)
           Wrote 8192 bytes (47 compressed) at 0x0000e000 in 0.1 seconds (effective 773.9 kbit/s)...
           Hash of data verified.
           Compressed 235584 bytes to 129169...
           Writing at 0x00010000... (12 %)
           Writing at 0x0001e3d4... (25 %)
           Writing at 0x000243a9... (37 %)
           Writing at 0x0002963e... (50 %)
           Writing at 0x0002ec0f... (62 %)
           Writing at 0x00037146... (75 %)
           Writing at 0x0003f264... (87 %)
           Writing at 0x0004482d... (100 %)
           Wrote 235584 bytes (129169 compressed) at 0x00010000 in 2.3 seconds (effective 803.3 kbit/s)...
           Hash of data verified.
           
           Leaving...
           Hard resetting via RTS pin...

Test on ESP32
==================

If the code is installed successfully, you should see that the LED on the ESP 32
is blinking. On my board, it's the LED further right and is blinking blue.

Code Explaination
====================

In both examples, 

* the code use the pin definition ``LED_BUILTIN``, Line 1 and Line 23 respectively.
  
  * my teyten board or Shawn's hymmel did not need the pin to be reconfigured. 
    The default seems to work.
  * Simply Explained dude was using the Lolin32 Lite board. The pinout shows
    that the LED Builtin is attached to GPIO22, hence he redefined his.

  * I search for ``LED_BUI`` from the top level directory ``C:\Users\ricky\AppData\Local\Arduino15\packages\esp32``

    * In this file, ``hardware\esp32\2.0.6\variants\doitESP32devkitV1\pins_arduino.h``,
      I see that LED_BUILTIN is pin # 2.

In Shawn's code [3]_,

* In line 12-16 he restrict the code to use one core only.

  * Normally, you would not want to use this,
    so you can unleash the full dual-core power of your ESP32,
    but, for learning RTOS concepts, having a second core
    can actually make things more difficult.

* Line 26-43: Creation of tasks

  * A task is simply a function that gets called during setup,
    or from another task. However, we have to tell the scheduler 
    about it so that it can prioritize and time-slice it appropriately.

    * For FreeRTOS, the function should return nothing,and accept one void pointer 
      as a parameter. This will allow you to pass in arguments if needed. However, 
      it can sometimes be tricky if your setup or calling task goes out of scope,
      which might remove the memory allocated for the argument.
    
    * For Task, SH (Shawn Hymmel)

  * and call freeRTOS api ``vTaskDelay`` instead of arduino define Delay. This is
    a non-blocking wait and works in vanilla FreeRTOS.

    * It tells the scheduler to run other tasks until the specified delay time is 
      up, and then come back to continue running this task.
    
    * Almost all RTOSs you come across are based on a tick timer. A **tick timer** 
      is simply one of the microcontroller's hardware timers allocated to interrupt 
      the processor at a specific interval.
      
      * **The interrupt period is known as a tick**, and the scheduler has an 
        opportunity to run each tick to figure out which task needs to run for 
        that tick.
    
    * By default, FreeRTOS sets the tick period to one millisecond and defines 
      ``portTICK_PERIOD_MS`` to one. ``vTaskDelay`` **expects the number of 
      ticks to delay, not the number of milliseconds.** So, we need to divide 
      our desired milliseconds by the tick period for the argument

      .. collapse:: understand portTICK_PERIOD_MS definition
         
         * a search for portTICK_PERIOD_MS reveals
           
           .. code-block:: bash
              
              hardware\esp32\2.0.6\tools\sdk\esp32\include\freertos\port\xtensa\include\freertos\portmacro.h:
                107  #define portSTACK_GROWTH                ( -1 )
                108: #define portTICK_PERIOD_MS              ( ( TickType_t ) 1000 / configTICK_RATE_HZ )
                109  #define portBYTE_ALIGNMENT              4
              
              hardware\esp32\2.0.6\tools\sdk\esp32c3\include\freertos\port\riscv\include\freertos\portmacro.h:
                102  #define portSTACK_GROWTH                (-1)
                103: #define portTICK_PERIOD_MS              ((TickType_t) (1000 / configTICK_RATE_HZ))
                104  #define portBYTE_ALIGNMENT              16
              
              hardware\esp32\2.0.6\tools\sdk\esp32s2\include\freertos\port\xtensa\include\freertos\portmacro.h:
                107  #define portSTACK_GROWTH                ( -1 )
                108: #define portTICK_PERIOD_MS              ( ( TickType_t ) 1000 / configTICK_RATE_HZ )
                109  #define portBYTE_ALIGNMENT              4
              
              hardware\esp32\2.0.6\tools\sdk\esp32s3\include\freertos\port\xtensa\include\freertos\portmacro.h:
                107  #define portSTACK_GROWTH                ( -1 )
                108: #define portTICK_PERIOD_MS              ( ( TickType_t ) 1000 / configTICK_RATE_HZ )
                109  #define portBYTE_ALIGNMENT

         * A search for clock rate reveals

           .. code-block:: bash
              
              hardware\esp32\2.0.6\tools\sdk\esp32\include\freertos\include\esp_additions\freertos\FreeRTOSConfig.h:
                140  #define configRECORD_STACK_HIGH_ADDRESS                 1
                141: #define configTICK_RATE_HZ                              ( CONFIG_FREERTOS_HZ )
                142  
            
            * CONFIG_FREERTOS_HZ
              
              .. code-block:: bash
                 
                 hardware\esp32\2.0.6\tools\sdk\esp32\sdkconfig:
                   1104  CONFIG_FREERTOS_SYSTICK_USES_CCOUNT=y
                   1105: CONFIG_FREERTOS_HZ=1000
                   1106  # CONFIG_FREERTOS_ASSERT_ON_UNTESTED_FUNCTION is not set
                                              

* Line 45- ... setup

  * Set the led pin to be an output.
  * Create the taskCreate functions:

    * use ``xTaskCreatePinnedToCore`` to tell the scheduler that we want to run 
      our task in only one of the cores.
      
      .. warning::
         This function does not exist in Vanilla Free RTOS, and you'd want to use 
         ``xTaskCreate`` instead.

         .. note:: 
            Note that xTaskCreate does still work in ESP-IDF. However, by doing so, 
            you tell the scheduler that it's free to run the task in whatever 
            core it wants.

      * For the parameters, we need to tell it 
        
        #. what function we want to call for our task,
        #. and give the task a name as a string.
        #. We then set the stack size in bytes.
          
           * Note that this would be number of words in Vanilla FreeRTOS.
           * According to the config file we looked at earlier, the smallest stack 
             size we can set here is 768 bytes, which is the minimum required to 
             run an empty task, and whatever overhead the scheduler needs.
           
           * Since this is a basic task, let's increase that to one kilobyte.

        #. You can use the next parameter to pass in a pointer to some memory 
           as an argument to the task function, if you'd like.
        
        #. The task priority. The higher the number, the higher the priority.
           
           * The config max priorities setting can be found in the config file,
             and it's set to 25 by default.
             That means we can assign a priority between zero and 24,
           
           * If you would like to manage this task from other tasks, or the main 
             execution loop, you can set a pointer or handle to this task. That way, 
             you can check on its status, watch its memory usage, or end it if necessary.
        
        #. for this particular function, we need to define the CPU core we want 
           the task to run in

           .. note::
              Note that this parameter is not presentin the xTaskCreate function 
              for Vanilla FreeRTOS.

* Line 74-...

  * In the Arduino framework for ESP32, the setup and loop functions exist inside 
    their own task, separate from the main program entry point. Because of this, 
    a new task is spawned as soon as we call xTaskCreate, or xTaskCreatePinnedToCore.
  
  * In other systems, you would need to call the vTaskStartScheduler function 
    to tell the scheduler to start running. Only then would new tasks begin executing.
    However, that function has already been called for us prior to the setup function, 
    so we don't need to worry about it. We will leave the ``loop()`` function empty.


.. warning::
   
   Note that the setup and loop functions run as their own task with priority one 
   in core one of the ESP32. As a result, this task may override other tasks you create,
   if you make them a lower priority.

************************************
References and Related Projects
************************************


.. [1] |lirmm_FreeRTOSIntro|
.. [2] |VanillaFreeRTOSvsEsp-IDF|
.. [3] |RTOS_DigiKeyPlaylist_IntroToRTOSPart2|

* |FreeRTOSBookAndReferenceGuide| 

* |ESP32-ArduinoAndFreeRTOS|

.. [5] “What is an RTOS?” article: https://www.freertos.org/about-RTOS.html 

.. [6] |SimplyExplained_SetupArduineIDEforESP32|

**Related Projects**

* `Getting Started with STM32 - Introduction to STM32CubeIDE <https://www.digikey.com/en/maker/projects/getting-started-with-stm32-introduction-to-stm32cubeide/6a6c60a670c447abb90fd0fd78008697>`_
* `Getting Started with STM32 - I2C Example <https://www.digikey.com/en/maker/projects/getting-started-with-stm32-i2c-example/ba8c2bfef2024654b5dd10012425fa23>`_
* `Getting Started with STM32 - Introduction to FreeRTOS <https://www.digikey.com/en/maker/projects/getting-started-with-stm32-introduction-to-freertos/ad275395687e4d85935351e16ec575b1>`_
