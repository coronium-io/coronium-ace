# ~ Coronium ACE RPi ~

<a href="https://twitter.com/share" class="twitter-share-button" data-via="develephant" data-size="large" data-hashtags="coroniumace">Tweet</a>
<script>!function(d,s,id){var js,fjs=d.getElementsByTagName(s)[0],p=/^http:/.test(d.location)?'http':'https';if(!d.getElementById(id)){js=d.createElement(s);js.id=id;js.src=p+'://platform.twitter.com/widgets.js';fjs.parentNode.insertBefore(js,fjs);}}(document, 'script', 'twitter-wjs');</script>

[![Build Status](https://drone.io/github.com/coronium-io/coronium-ace-raspbian/status.png)](https://drone.io/github.com/coronium-io/coronium-ace-raspbian/latest)

*Coronium ACE PI is the communication channel between your RPi device and client, allowing you to quickly build lightweight APIs to control your RPi.*

__Coronium Ace Pi (acepi) is available for installation on [Raspbian Jessie](https://www.raspberrypi.org/downloads/raspbian/), running on a Raspberry Pi version 1B or better.__

## Differences from Ace

 - Ace Pi is aimed at the [Raspberry Pi](https://www.raspberrypi.org) (RPi) small form factor "computer".  Specifically the [Raspbian Jessie](https://www.raspberrypi.org/downloads/raspbian/) Linux distribution.
 
 - The `ace` card is called as `acepi` on the Raspberry Pi devices.

 - Coronium Ace Pi is _not_ Docker based, and installs on the Raspbian OS directly.

 - There is only a single 'app' instance, but you can create as many modules as your sdcard can hold.
 
 - 12 additional namespaces relevant to the Raspberry Pi and low-level system tools have been added.
 
 - The __lpeg__ library has been added, which allows you to easily parse system data.
 
## Installation

This is a development version of Coronium Ace Pi and should be installed separate of your main system while testing.

___For best results, a fresh sdcard install with [Raspbian Jessie](https://www.raspberrypi.org/downloads/raspbian/) is highly recommended and will be assumed in the upcoming steps.___



## Additional Modules

Coronium Ace Pi comes with some extra API tools for interacting with the RPi hardware, and includes the following additional modules:

Module|Details
------|-------
[Periphery](https://github.com/vsergeev/lua-periphery/tree/master/docs)|A Lua library that provides access to GPIO, SPI, I2C, MMIO, and Serial interfaces.
[ljsyscall](https://github.com/justincormack/ljsyscall)|Unix system calls for LuaJIT. Usage requires some code diving.
[lpeg](http://www.inf.puc-rio.br/~roberto/lpeg/)|Parsing Expression Grammars For Lua. Next generation regex.

In addition to the [main Ace library](ace_api.md), The following namespaces are available on the Raspberry Pi to use in your Lua code as well:

__lpeg__

  - ace.lpeg
  
__Periphery__

  - ace.gpio
  - ace.i2c
  - ace.mmio
  - ace.spi
  - ace.serial
  
__RPi Userland__

  - ace.vccmd
  
__Unix syscall__

  - ace.sys
  - ace.sys.utils
  - ace.sys.nl
  
__System Command__

  - ace.cmd
  
