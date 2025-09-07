---
layout: post
title:  "Designing some firmware for the power board"
pubished: false
date:   2025-08-26 18:09:30 +0200
categories: Hardware
---

It isn't just enough to get the PCB functioning, good firmware has to be written for it to maximise the performance. This is where I will write the code and try to understand whatever chatGPT chucks out.

# Requirements

I will need to use FreeRTOS to manage my tasks, which are:
- Monitoring the current of the 3 different subsystems
- Monitoring the voltage of the battery
- Sending these values via serial
- Turning on or off the indicator lights and making it non-blocking

# The code
A real time operating system works from tasks, broadly speaking just quick tasks that need to be done at a set frequency. So set the frequency of the task and then let it go as long as it doesnt take too long. It is a bit more complicated than that but hear me out.

So as far as how the code is written it makes the most sense to start from the declarations to give a flavour of the structure of the system with main and task and then go from there to the minutia.

## Task.h
So there are a few things we declare in the task.h file that are looked at during compilation. This includes all the datatypes and task names we will be using. So in this case all the general bits are:
- shared structs
  - Datapackets, the types that we use to communicate between the tasks that many of them use
- task functions
  - the freeRTOS entry points and encapsulates one job with its own timing
- task handles
  - these are the taskhandle_t identifiers returned by xTaskCreate*. allows for better manipulation of the task as they can be queried, nofigied or suspended.
- task frequencies
  - how often the task runs, works with the timing of vTaskdelayUntil to make sure everything happens at a set frequency
- task enables
  - in this case it toggles the tasks on or off
- shared variables
  - globals that multiple tasks read or write to, we protect them in this case with a mutex