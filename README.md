# IoT Starter Raspberry Pi Lumi

#### Home Intelligence with Raspberry Pi

### IoT.Starter.Pi.Thing for a Universal Remote Control

## Introduction

This series, targeted to Raspberry Pi with Linux,  started with [IoT.Starter.Pi.Core](https://www.codeproject.com/Articles/1220930/IoT-Starter-Raspberry-Pi-Core) using API First Design strategy  to develop an ASP.NET Core Web Server automatically generated by Swagger Hub.

The second part introduced  [IoT.Starter.Pi.Thing](https://www.codeproject.com/Articles/1224347/IoT-Starter-Raspberry-Pi-Thing), an  embryo for Home Intelligence using Raspberry Pi with Linux. 
Designed as a starter kit for IoT initiatives, it provides a solid and structured platform to speed up product development. The  [starter kit concept](https://www.codeproject.com/Articles/1122233/IoT-Starter-Core-for-Netduino-Plus) encourages the teams to start working immediately on the product development.

The third part [IoT.Starter.Pi.Lirc](https://www.codeproject.com/Articles/1226559/IoT-Starter-Raspberry-Pi-Lirc) is dedicated to IoT projects that require infrared devices. A test environment is created by a `Console` powered by Lirc, the Linux Infrared Remote Control.  The `Thing` is now able to benefit from what we learned from these tests.

The objective now is to extend the `Thing` web services to allow IR remotes and  their respective codes to be considered by the API. Powered by Lirc, the `IoT.Starter.Pi.Thing` can used on IoT initiatives that require infrared support.

## API first

Following the API first strategy, a new [API version 1.0.2](https://app.swaggerhub.com/apis/motta/home/1.0.2) is created and summarized at picture below. The `GET` operations should identify remotes and their respective IR codes. The `POST` operation reflects the intention to create routes to fire IR blasts for remotes already installed at RPI host, using Lirc software.     

![](https://i.imgur.com/Mb5TWpO.png)

### Generate home-web