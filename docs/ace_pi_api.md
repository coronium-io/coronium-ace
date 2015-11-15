# ~ ACE PI Modules ~

**In the Coronium ACE PI Developer release anything is subject to change.**

*Make sure to read the [Installation](ace_pi.md#installation) and [Developing](usage.md) sections first.*

# lpeg

## ace.lpeg
The `ace.lpeg` namespace gives access to the LPeg parsing library.

```lua
-- matches a word followed by end-of-string
local p = ace.lpeg.R"az"^1 * -1
return { res = p:match("hello") } --> 6
```

!!! info "Documentation"
  __[Learn more about how to use this library here.](http://www.inf.puc-rio.br/~roberto/lpeg/)__

---

# Periphery

## ace.gpio

```lua
local GPIO = ace.gpio

-- Open GPIO 10 with input direction
local gpio_in = GPIO(10, "in")
-- Open GPIO 12 with output direction
local gpio_out = GPIO(12, "out")

local value = gpio_in:read()
gpio_out:write(not value)

gpio_in:close()
gpio_out:close()

return { res = value }
```

!!! info "Documentation"
  __[Learn more about how to use this library here.](https://github.com/vsergeev/lua-periphery/blob/master/docs/gpio.md)__

## ace.i2c

```lua
local I2C = ace.i2c

-- Open i2c-0 controller
local i2c = I2C("/dev/i2c-0")

-- Read byte at address 0x100 of EEPROM at 0x50
local msgs = { {0x01, 0x00}, {0x00, flags = I2C.I2C_M_RD} }
i2c:transfer(0x50, msgs)
local val = ace.sf("0x100: 0x%02x", msgs[2][1])

i2c:close()

return { res = val }
```

!!! info "Documentation"
  __[Learn more about how to use this library here.](https://github.com/vsergeev/lua-periphery/blob/master/docs/i2c.md)__

## ace.spi

```lua
local SPI = ace.spi

-- Open spidev1.0 with mode 0 and max speed 1MHz
local spi = SPI("/dev/spidev1.0", 0, 1e6)

local data_out = {0xaa, 0xbb, 0xcc, 0xdd}
local data_in = spi:transfer(data_out)

local v1 = ace.sf("shifted out {0x%02x, 0x%02x, 0x%02x, 0x%02x}", unpack(data_out))
local v2 = ace.sf("shifted in  {0x%02x, 0x%02x, 0x%02x, 0x%02x}", unpack(data_in))

spi:close()

return { v1 = v1, v2 = v2 }
```

!!! info "Documentation"
  __[Learn more about how to use this library here.](https://github.com/vsergeev/lua-periphery/blob/master/docs/spi.md)__

## ace.serial
```lua
local Serial = ace.serial

-- Open /dev/ttyUSB0 with baudrate 115200, and defaults of 8N1, no flow control
local serial = Serial("/dev/ttyUSB0", 115200)

serial:write("Hello World!")

-- Read up to 128 bytes with 500ms timeout
local buf = serial:read(128, 500)
local res = ace.sf("read %d bytes: _%s_", #buf, buf)

serial:close()

return { res = res }
```

!!! info "Documentation"
  __[Learn more about how to use this library here.](https://github.com/vsergeev/lua-periphery/blob/master/docs/serial.md)__

## ace.mmio

```lua
local MMIO = ace.mmio

-- Open am335x real-time clock subsystem page
local rtc_mmio = MMIO(0x44E3E000, 0x1000)

-- Read current time
local rtc_secs = rtc_mmio:read32(0x00)
local rtc_mins = rtc_mmio:read32(0x04)
local rtc_hrs = rtc_mmio:read32(0x08)

local v1 = ace.sf("hours: %02x minutes: %02x seconds: %02x", rtc_hrs, rtc_mins, rtc_secs)

rtc_mmio:close()

-- Open am335x control module page
local ctrl_mmio = MMIO(0x44E10000, 0x1000)

-- Read MAC address
local mac_id0_lo = ctrl_mmio:read32(0x630)
local mac_id0_hi = ctrl_mmio:read32(0x634)

local v2 = ace.sf("MAC address: %04x%08x", mac_id0_lo, mac_id0_hi)

ctrl_mmio:close()

return { v1 = v1, v2 = v2 }
```

!!! info "Documentation"
  __[Learn more about how to use this library here.](https://github.com/vsergeev/lua-periphery/blob/master/docs/mmio.md)__
  
---

# RPi Userland
  
## ace.vccmd

The RPi contains a userland binary that you can query with different commands. Parameters are not well documented, but you can find some interesting items on Google in regards to the [Raspberry Pi vcgencmd](https://www.google.com?q=Raspberry Pi vcgencmd)

```lua
local ok, result = ace.vccmd( 'measure_temp' )
if ok then
  return { temp = result }
end
```

__Parameters:__

Name      | Type  | Description | Required
--------- | ----- | ----------------------- |
command | String | The `vcgencmd` command to call. | Yes
rw_mode | Constant | Defaults to `ace.READ`. To set a value using `vcgencmd`, you must set the mode to `ace.WRITE` | No
read_mode | Constant | How the result from the command is read in. Defaults to `ace.READALL`. Can also be `ace.READLINE` or `ace.READNUM` | No

__Returns:__

Type  | Description
----- | ------------------------------------
Boolean or `nil` |The return status of the operation.
Variable|The result or error from the operation.

The `ace.vccmd` method returns 2 results; the status, and the actual result on success, or nil and usually an error message on error.
  
___The current list of `vcgencmd` commands available at this time:___


```shell
vcos, ap_output_control, ap_output_post_processing, vchi_test_init,
vchi_test_exit, pm_set_policy, pm_get_status, pm_show_stats, pm_start_logging,
pm_stop_logging, version, commands, set_vll_dir, led_control, set_backlight,
set_logging, get_lcd_info, set_bus_arbiter_mode, cache_flush, otp_dump, test_result,
codec_enabled, get_camera, get_mem, measure_clock, measure_volts, scaling_kernel,
scaling_sharpness, get_hvs_asserts, measure_temp, get_config, hdmi_ntsc_freqs,
hdmi_adjust_clock, hdmi_status_show, hvs_update_fields, pwm_speedup, force_audio,
hdmi_stream_channels, hdmi_channel_map, display_power, read_ring_osc, memtest,
get_rsts, schmoo, render_bar, disk_notify, inuse_notify, sus_suspend, sus_status,
sus_is_enabled, sus_stop_test_thread, egl_platform_switch, mem_validate, mem_oom,
mem_reloc_stats, file, vctest_memmap, vctest_start, vctest_stop, vctest_set, vctest_get
```

You can use `ace.vccmd( 'commands' )` for the listing of available commands.

```lua
local ok, res = ace.vccmd( 'commands' )
if ok then
  return { cmds = res }
end
```
  
The actual parameters that these commands take are still being discovered, and some don't take any. Again, [a quick Google search](https://www.google.com?q=Raspberry Pi vcgencmd) will provide more usage information and tips.

__Example:__

Get temperature of the RPi.

```lua
local ok, res = ace.vccmd( 'measure_temp' )
--Check that the call succeeded
if not ok then
  --Error
  return ace.error( res )
end

 --Temp result
return { temp = res }
```

Measure `arm` clock.

```lua
local ok, res = ace.vccmd( 'measure_clock arm' )
if not ok then
  return error()
end

return { arm_clock = res }
```

Turn off display.

__When setting a value using `vccmd`, you must pass `ace.WRITE` as the rw_mode__

```lua
local ok, res = ace.vccmd( 'display_power off', ace.WRITE )
```

Turn on display. Return only the first line of the result.

```lua
local ok, res = ace.vccmd( 'display_power on', ace.WRITE, ace.READLINE )
if ok then
  return { res = res }
end
```

---

# Command
  
## ace.cmd

This method allows one to pass a command to the system process. You can also run other RPi scripts -- in python or perl for example -- on your system with this method.

Get the system load average.

```lua
local ok, res = ace.cmd( 'cat /proc/loadavg' )
if ok then
  return { top = res }
end
```

__Parameters:__

Name      | Type  | Description | Required
--------- | ----- | ----------------------- |
command | String | The system command to call. | Yes
rw_mode | Constant | Defaults to `ace.READ`. When setting a value, you must set the mode to `ace.WRITE` | No
read_mode | Constant | How the result from the command is read in. Defaults to `ace.READALL`. Can also be `ace.READLINE` or `ace.READNUM` | No

__Returns:__

Type  | Description
----- | ------------------------------------
Boolean or `nil` |The return status of the operation.
Variable|The result or error from the operation.

!!! info "Tip"
  This method is similar to running a command in a terminal program.

---

# ljsyscall

The [__ljsyscall__](https://github.com/justincormack/ljsyscall) library is not very well documented, but is a popular Lua module for issuing low level commands on your Unix/Linux system. Not every command works, so only through testing will you know which ones will work on your RPi.

!!! warning "Documentation"
  This module will require some code diving to discover its abilities. You can [start here](https://github.com/justincormack/ljsyscall/tree/master/test).

## ace.sys

```lua
local ok, res = ace.sys.mkdir('/usr/local/stuff')
```

## ace.sys.utils

```lua
local ok, res = ace.sys.utils.mv('/usr/local/stuff', '/opt/stuff')
```

## ace.sys.nl

Networking tools.
