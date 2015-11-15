# Coronium ACE Pi

<a href="https://twitter.com/share" class="twitter-share-button" data-via="develephant" data-size="large" data-hashtags="coroniumacerpi">Tweet</a>
<script>!function(d,s,id){var js,fjs=d.getElementsByTagName(s)[0],p=/^http:/.test(d.location)?'http':'https';if(!d.getElementById(id)){js=d.createElement(s);js.id=id;js.src=p+'://platform.twitter.com/widgets.js';fjs.parentNode.insertBefore(js,fjs);}}(document, 'script', 'twitter-wjs');</script>

[![Build Status](https://drone.io/github.com/coronium-io/coronium-ace-raspbian/status.png)](https://drone.io/github.com/coronium-io/coronium-ace-raspbian/latest)

*Coronium ACE Pi is the communication channel between your RPi device and client, allowing you to quickly build lightweight APIs to control your RPi.*

__Coronium ACE Pi (acepi) is available for installation on [Raspbian Jessie](https://www.raspberrypi.org/downloads/raspbian/), running on a Raspberry Pi version 1B or better.__

## Differences from ACE

 - ACE Pi is aimed at the [Raspberry Pi](https://www.raspberrypi.org) (RPi) small form factor "computer".  Specifically the [Raspbian Jessie](https://www.raspberrypi.org/downloads/raspbian/) Linux distribution.
 
 - The `ace` card is called as `acepi` on the Raspberry Pi devices.

 - Coronium ACE Pi is _not_ Docker based, and installs on the Raspbian OS directly.

 - There is only a single 'app' instance, but you can create as many modules as your sdcard can hold.
 
 - 12 additional namespaces relevant to the Raspberry Pi and low-level system tools have been added.
 
 - The __lpeg__ library has been added, which allows you to easily parse system data.
 
## ACE Pi Installation

__This is a development version of Coronium ACE Pi and should be installed separate of your main card while playing.__
  

___For best results, a fresh sdcard install with [Raspbian Jessie](https://www.raspberrypi.org/downloads/raspbian/) is highly recommended and will be assumed in the upcoming steps.___

__Preflight__

Install a current version of __Raspbian Jessie OS__ on your Raspberry Pi 1b or 2 device.

__[Click Here for Raspbian Installation Options](https://www.raspberrypi.org/downloads/)__

!!! info "Note"
  Use NOOBS if you're new to installing Raspbian on the RPi, though using an image is about 5 times faster.

---

__Initial setup__

When you first boot to the Raspbian UI, in the upper-left, select __Menu > Preferences > Raspberry Pi Configuration__

 1. Click __Change Password__ to update the password.
 1. In __Hostname__ enter `ace-pi` (or something).
 1. In The __Boot__ section, select "To CLI".
 1. In __Auto login__, uncheck "Login as user 'pi'"
 1. Click "OK".
 1. __Reboot when prompted.__
 
---
 
__raspi-config__

Your RPi will reboot into terminal mode (no more UI). The UI takes up precious RAM, and we really don't need a full OS UI for ACE Pi. If you are running a RPi 1b, you can't run Ace and the UI at the same time without serious performance degradation.

 - At the login prompt enter: __pi__.
 - Enter whatever password you created at first launch.

Once you are logged in, you should perform the following "tweaks" to the system. First run the `raspi-config` tool.

```bash
sudo raspi-config
```

Once the config tool loads you will see many different options. Make changes to the following:

!!! info "Tip"
  You can ignore any warnings that pop up. It all works itself out in the end.
  
___Required___
  
  - __Expand Filesystem__ (Will update on next reboot)
  - __Advanced Options > Memory Split__ : Set to "16"
  - __Internationalisation Options > Change Locale__ : User choice (UTF-8 prefferred)
  - __Internationalisation Options > Change Timezone__ : User choice
  - __Internationalisation Options > Change Keyboard Layout__ : User choice
  - __Advanced Options > SPI__ : Enable (at 'autoload' prompt: "yes")
  - __Advanced Options > I2C__ : Enable (at 'autoload' prompt: "yes")
  
___Optional___
  
  - __Enable Camera__
  - __Overclock__
  
Reboot when prompted.

---

__Install Redis__

 - After the system reboots login with __pi__.
 - Password should be what you set earlier in the setup.

```bash
sudo apt-get update
sudo apt-get install -y redis-server
```

---

__ACE Pi Installation__

Coronium ACE Pi is easily installed as a Debian package.

```bash
wget https://s3.amazonaws.com/coronium-ace/rpi/coronium-ace-rpi_0.3-1.deb
sudo dpkg -i coronium-ace-rpi_0.3-1.deb
```

Your new instance can be found at __/home/pi/Ace__

---

__Module Development__

Your modules should be placed in __/home/pi/Ace/app__

The port that the ACE Pi service runs on is __8081__

---

__ACEPi Card__

You can manage your ACE Pi instance by using the `acepi` tool:

```bash
acepi
```

---

__Uninstall ACE Pi__

`sudo dpkg -r coronium-ace-rpi`

__Uninstall Redis-Server__

`sudo apt-get remove redis-server`

!!! warning "Note"
  Any files added to the Ace Pi installation will not be removed by the uninstaller. This protects your added modules from accidental deletion.

---

## Additional Modules

Coronium ACE Pi comes with some extra API tools for interacting with the RPi hardware, and includes the following additional modules:

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
  
__[Learn more about module usage in the API docs.](ace_pi_api.md)__

---

# Troubleshooting

__Start the Instance__

Check to make sure the ACE Pi instance is running by logging into the RPi and issuing the following command, then checking for the "status" text.

```bash
acepi
```

If your instance is "__stopped__", then issue the following:

```bash
acepi start
```

It should now be in a "__running__" state. Run `acepi h` for more control options.

__Accessing ACE Pi__

Your API entrypoint can be found at: __http:\/\/your.rpi_domain_or_ip:8081__

Call an API endpoint: __http:\/\/your.rpi_domain_or_ip:8081/echo/test?hello=Pi__


__Logs, Modules and More__

You can view all the ACE Pi options available by entering:

```bash
acepi help
#OR
acepi h
```

__Coronium ACE Library__

In addition to the ACE Pi modules, you also have access to all the base [Coronium Ace API](ace_api.md) modules.
