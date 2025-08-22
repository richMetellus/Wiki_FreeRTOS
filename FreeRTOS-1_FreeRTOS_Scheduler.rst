.. include:: /global.rst 
   :start-after: _RTOSLinksStart:
   :end-before: _RTOSLinksEnd


####################################################
Intro to RTOS Part 3: FreeRTOS Task Scheduling [1]_
####################################################

******************************
Agenda and Description
******************************

In this section, I will make note on the lesson of FreeRTOS scheduler and
how tasks get schedule. I will rely on various source to get a complete picture.

FreeRTOS allows us to set priorities for tasks, which allows the scheduler to 
preempt (interrupt) lower priority tasks with higher priority tasks. The scheduler is a 
piece of software inside the operating system in charge of figuring out which 
task should run at each tick.

***************
Intro: Concept
***************

Usually with the help of a scheduler, writing a multi-threaded (or multi-task) 
program looks something like this in code:

.. image:: ../../_images/FreeRTOS-1_ConcurrentThread_withISR.jpeg
   :height: 310px
   :width: 350px

* From our perspective Each task appears to run concurrently in its own while loop (assuming we don't 
  end a thread after a single execution) running independently and at the same time. 
    
    * In microcontrollers, we can also set up independent interrupt service 
      routines (ISRs) that can preempt any of the tasks to execute some code. 
        
        * An ISR is used to handle things like hardware timer overflows, pin 
          state changes, or new communication on a bus.

A more useful representation of how a core give use the impression of concurrency
is this


.. image:: ../../_images/FreeRTOS-1_ConcurrentThread_OneCore_TimeSlicing.jpeg


Let's imagine we are looking at processor utilization over time which moves
from left to right

* Each row in the diagram above is priority
* In a single-core system, the CPU must divide up its time among the tasks 
  that we wrote into time slices so  that they can appear to run concurrently.
  **The scheduler in an operating system is charged with figuring out which task 
  to run each time slice.**

    * one of the hardware timers interrupts the processor at regular intervals 
      you'll commonly see a time slice of one millisecond in free

        * the timer calls this scheduler task at every interval which is one 
          millisecond in our case
    
    * In FreeRTOS, the default time slice is 1 ms, and a time slice is known as a 
      “tick.” A hardware timer is configured to create an interrupt every 1 ms. 
      The ISR for that timer runs the scheduler, which chooses the task to run next.
    
    * At each tick interrupt, the task with the highest priority is chosen to run. 
      If the highest priority tasks have the same priority, they are executed in a 
      round-robin fashion. If a task with a higher priority than the currently running 
      task becomes available (e.g. in the "Ready" state), then it will immediately run. 
      It does not wait for the next tick.
    * A hardware interrupt is always considered to have a higher priority than 
      any task running in software. As a result, a hardware ISR can interrupt 
      any task. Because of this, we recommend keeping any ISR code as short as 
      possible to reduce interruptions to the running tasks.
    
    * When we create tasks, we can assign them priorities. In fact, we can even 
      change priorities of tasks with vTaskPrioritySet() 
      (read more about that function `here <https://www.freertos.org/a00129.html>`_).

Animation Explaination:

At each tick interrupt, the task with the highest priority is chosen to run. 
If the highest priority tasks have the same priority, they are executed in a 
round-robin fashion. If a task with a higher priority than the currently running 
task becomes available (e.g. in the "Ready" state), then it will immediately run. 
It does not wait for the next tick.

1. let's say that ``TaskA`` has a very low priority and it's the only task 
   that needs to run so the scheduler chooses it and task a runs for the 
   rest of the time slice.
2. the tick timer then interrupts the task and calls the scheduler to run 
   again.
#. TaskA has not finished what it needs to do so the scheduler lets it 
   finish running as there are still no other higher priority tasks before 
   this interval is done.
#. TaskA calls the ``vTaskDelay`` function for two ticks and enters the 
   blocked state. (CPU idle)
#. the next time the scheduler is called counts as one of the ticks for 
   that delay however it still has one more tick to wait and the 
   scheduler has no other tasks in the ready state **so the operating system 
   just idles for one tick.**
#. At that point TaskA's delay is over and is ready to run again so 
   the scheduler lets it run at some point during this
#. At some point while TaskA is running in the middle of the tick, TaskB and
   TaskC are in the ready state, **A higher priority task in FreeRTOS will 
   IMMEDIATELY preempt other tasks and run if it is made ready.  It does not 
   wait for the next tick to run** (R. diagram pic is not accurate)
    
    .. note:: 
       There are different scheduling algorithms and the algorithm can be configured 
       by specifying: 
       
       * ``configUSE PREEMPTION``
       * ``configUSE TIME SLICING``
       
       The options have to be configured into ``FreeRTOSConfig.h``

       See the intro to FreeRTOS slide by Aberto Bosio.


    * since TaskB and TaskC are priority one which is higher than TaskA's priority 
      zero, they get scheduled to run.
    
    * B and C are equal priority and so the scheduler executes each task in turn in a
      round-robin fashion b and c will continue taking turns like this so long 
      as they need to run. 
        
        * This is known as :term:`pre-emptive scheduling` because cpu time is 
          taken away from one task (TaskA) to run other higher priority tasks.
    
    * keep in mind that this is all done in software, we have not talked about
      hardware interrupts yet unless you do something in code to specifically 
      disable hardware interrupts.
    
    * a hardware interrupt will always have a higher priority over other software 
      running.
        
        * the only exception to this is that a hardware interrupt may or  may 
          not preempt another hardware interrupt service routine.
            
            * that's called `nested interrupts` and depends a lot on your 
              particular hardware and configuration it doesn't have much to do 
              with the rtos.

    .. important::
       Shwan recommend having only one interrupt service routine run at a time 
       and trying to keep them as short as possible.
       whenever an ISR is done running execution will return to whichever task
       was running. 

#. TaskB runs get an HW interrupt, ISR take priority and return to taskB
#. once tasks B and C are complete or go into the blocked or suspended state 
   the scheduler will then allow the lower priority TaskA to run.



in a multi-core system a scheduler may choose to put some tasks on another core.
For example, if we let the esp-idf scheduler decide how to allocate tasks between 
the cores it may allow TaskB and TaskC to run at exactly the same time on separate 
cores. 

* For simplicity the lesson here use only one core.
    
**************
Task States
**************

the free rtos scheduler maintains a record of what state each task is in.

* When a task is created it automatically enters the ``ready`` state

    .. image:: ../../_images/FreeRTOS-1_TaskState_Create-Ready.png
    
    * the task is telling the scheduler that it's ready to run at any time.
        
* the scheduler can choose to run that task only if there are no other higher
  priority tasks waiting to run.

    * if a task is not run because another task of equal or higher priority was 
      chosen then it remains in the ready state.
    * If a task is chosen by the scheduler to run it will enter the ``Running`` 
      state and remain in that state while it is using the processor.

        .. image:: ../../_images/FreeRTOS-1_TaskState_Create-Ready-Running.png
    
    * if the processor has only one core then there can only be one task in the
      running state at any given time.


* the scheduler can move tasks between the ready and running states as needed at 
  each tick.

* While running a task may call an api function that moves it to the 
  blocked state.

  .. image:: ../../_images/FreeRTOS-1_TaskState_Create-Ready-Running-Blocked.png

  * these api functions can be things like ``vTaskdelay`` or waiting for a
    queue or semaphore.
  
  * Tasks in the blocked state do not run on the processor and
    cannot be selected to enter the running state.
  * A call to vTaskDelay will put your task to sleep (blocked from getting any CPU) 
    for the number of FreeRTOS ticks specified.  You can't use it for precise timing, 
    but it's fine for a task they needs to wake up now and then to do something.  
    Like updating LEDs, checking a battery level, etc. [2]_

  * The task will wait until the unblocking event has occurred like the delay 
    timer expiring or a semaphore being released the task will enter the ready 
    state and wait to be scheduled for processor time
  
    * Here, the task is waiting for some other event to occur, such as the timer 
      on the vTaskDelay() to expire. The task may also be waiting for some resource, 
      like a semaphore, to be released by another task. 
  
  * **Tasks in the Blocked state allow other tasks to run instead.**

* FreeRTOS has a ``vtaskSuspend()`` api function that allows you to put a task 
  into the suspended state.
  
  .. image:: ../../_images/FreeRTOS-1_TaskState_Create-Ready-Running-Blocked-Suspended.png

  * you can call this function from within the task itself or from another task. 
  
  * **In the suspended state**, a task cannot be selected to run just like the blocked state,
     however **only an explicit call to the** ``vTaskResume()`` **function will allow a 
     task to go to the ready state.**

     * this is a good way to essentially put tasks to sleep if you don't want to 
       rely on a timer like vTaskDelay 

  * **Any task may put any task (including itself) into the Suspended mode.**
  * **A task may only be returned to the Ready state by an explicit call to**
    ``vTaskResume()`` **by another task.**

************************
Context Switching
************************

when switching from one task to the next the scheduler has the job of remembering 
exactly where the task left off and all of its working variables including values 
in RAM and in the cpu registers. 

* All of this information where the task was in the program instructions and 
  working memory is known as **a context** and the process of saving it all and 
  restoring another task's context is called **context switching**.


.. seealso:: 
   To get more idea of FreeRTOS do the the context switch, see this FreeRTOS
   website `context switching basic <https://www.freertos.org/implementation/a00006.html>`_
   and `context switch detailed steps <https://www.freertos.org/implementation/a00019.html>`_

.. note::
  note that the stack allocated for the task is used to store much of this context 
  which is why we need to allocate a minimum amount when creating a task.

****************************
Context Switch In Action
****************************

Code and code explanation
===========================

The code will show task preemption.

* it will only use one core
* some bogus string will get printed from one task
* we will use some handles for both taks to control their states from a 3rd task
* 1st task 

  * counts the number of character in the string, print them out to the serial
    terminal one by one (one character at a time)

    * if we just used serial.print for the whole string you'd find that this task 
      would just copy the whole string to the arduino's serial buffer which would prevent 
      us from seeing any interruption by the second task
  
  * When it's done, put the task in the blocked stated using vTaskDelay for 1 sec

    *  the rtos will take care of returning this task to the ready state 
       whenever this delay time is up.

* 2nd task:

  * simply print out a single asteriks character every 100 millisecoonds.
  * higher priority task so it will preempt 1st task

* in setup()

  * serial terminal use a slow serial baud rate so we can see all all of this in real time.

    * too fast you'll likely get the whole string in the first time printed in 
      a single tick. The trick is to add a delay for about a second for the esp32
      to give the serial a chance to connect otherwise you might meiss some of the
      output
  
    * print out some information about the curent task.

      * this will demonstrate that setup and loop are indeed running in their own
        task. we then start the two other task.

  * we start to 2 task, first task with priority 1 and assigned the handle task 1.
  * 2nd task has higher priority of 2 and is assigned handle task 2 

* In the loop():

  * because we're running a third task we can use it to control the other tasks.
  * So let's suspend and resume task2, the one printing out the asterisks every two
    seconds after doing this a few times.
  
  * we'll use ``vTaskDelete`` to completely remove the first task

    .. important::
      Advice:
      Shawn H highly recommend checking to make sure the task handle is not null 
      and then immediately setting the handle to null after deleting it.

      .. warning::
         
         if you call ``vTaskDelete`` on a non-existent task you're gonna have a 
         bad time as in you'll probably overwrite some memory or cause the 
         processor to crash and reset

.. tabs:: 
   
   .. tab:: TasksPreemptionDemo
      
      This is the code by S.H

      .. literalinclude:: ../../_resources/FreeRTOS/ShawnH_IntroToRTOS/03-task-scheduling-and-management/esp32-freertos-03-demo-prioritization/esp32-freertos-03-demo-prioritization.ino
         :language: c

Test
========

1. Upload the code to the esp32
#. Set serial baudrate to 300
#. Reset the microcontroller to see the output early on

On the serial terminal,

* you should see that the setup and loop function do indeed run in their own
  task with priority one on core 1.

* the sentence should be printed at a regular interval and you should begin to 
  see an asterisk being printed from time to time even in the middle of the sentence
  written by task 1 as we starts, stops, suspend, resume that task. Task 2 should
  preempt task 1 time to time

* Whent the first task is deleted, just Task2 (asterisk) will continue to run (print).

****************************
Code Challenge
****************************

Create a program that uses 

* two tasks to control the blinking rate of an led.
  
  * one task should listen for input on the serial terminal.
    
    * when you enter a number the delay time on the blinking led should be 
      updated to that time. 
      
      * this allows us to create a simple user interface that runs 
        independently of the hardware being controlled.


.. tabs:: 
   
   .. tab:: ChallengeSolutionbyS.H
      
      This is the code by S.H

      .. literalinclude:: ../../_resources/FreeRTOS/ShawnH_IntroToRTOS/03-task-scheduling-and-management/esp32-freertos-03-solution-led/esp32-freertos-03-solution-led.ino
         :language: c


*************
Gloassary
*************


.. glossary::

    pre-emptive scheduling
        A type of scheduling where a task/thread of higher priority will interrupt
        (pre-empt) a lower-priority task that is currently running.

        Pre emptive scheduling algorithms will **immediately** 'pre empt' the 
        Running state task if a task that has a priority higher than the 
        Running state task enters the Ready state. Being pre-empted means 
        being involuntarily (without explicitly yielding or blocking) moved 
        out of the Running state and into the Ready state to allow a 
        different task to enter the Running state;


********************
References
********************

.. [1] Digikey and Youtube by Shawn Hymel
.. [2] https://www.microchip.com/forums/m1185542.aspx#:~:text=A%20call%20to%20vTaskDelay%20will,9%2F17%2F2021


