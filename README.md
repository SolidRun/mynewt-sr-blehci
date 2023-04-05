<!--
#
# Licensed to the Apache Software Foundation (ASF) under one
# or more contributor license agreements.  See the NOTICE file
# distributed with this work for additional information
# regarding copyright ownership.  The ASF licenses this file
# to you under the Apache License, Version 2.0 (the
# "License"); you may not use this file except in compliance
# with the License.  You may obtain a copy of the License at
#
# http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing,
# software distributed under the License is distributed on an
# "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
#  KIND, either express or implied.  See the License for the
# specific language governing permissions and limitations
# under the License.
#
-->

## Overview

This is a project for building BLE firmware that can be flashed to the Wireless MCUs available on some SolidRun products.
The firmware is based on Apache mynewt OS with the Apache NimBLE Application providing BLE functionality and HCI protocol support.

Supported Products:

- i.MX8MQ SoM
  - u-blox Nina-B1
- SolidSense N6
  - u-blox Nina-B1
  - u-blox Nina-B3
  - Fujitsu FWM7BLZ22

## Build Firmware from Source

### Install Build Dependencies and Tools

### Install ARM embedded toolchain

    sudo apt-get install gcc-arm-none-eabi

### Install Apache mynewt build tool

Either:

- Install natively according to [mynewt documentation](https://mynewt.apache.org/latest/get_started/index.html).
- Install with docker as a shortcut

      sudo apt install docker.io
      sudo usermod -a -G docker <username>
      # log out and back in
      alias newt='docker run -e NEWT_HOST=$(uname) $ti --rm --device=/dev/bus/usb --privileged -v $(pwd):/workspace -w /workspace quay.io/josua-sr/newt:latest /newt'


### Download Sources

    git clone https://github.com/SolidRun/mynewt-sr-blehci
    cd mynewt-sr-blehci

    newt upgrade

### Compile

First, choose a target:

- i.MX8MQ SoM (with u-blox Nina-B1): `imx8mqsom-nina-b1`
- SolidSense N6 with u-blox Nina-B1: `ssn6-nina-b111`
- SolidSense N6 with u-blox Nina-B3: `ssn6-nina-b301`
- SolidSense N6 with Fujitsu FWM7BLZ22: `ssn6-fwm7blz22`

Use substitute in the instructions below "imx8mq-nina-b1" with the correct target name.

Then compile bootloader & application:

    newt build imx8mqsom-nina-b1_boot
    newt build imx8mqsom-nina-b1_blehci

Finally create an application image (adding header with version number).
Note that the version number here is a user choice and has no functional impact.

    newt create-image imx8mqsom-nina-b1_blehci 0.0.1

## Install firmware

### Install openocd

OpenOCD is a debugger and programmer that will be used for installing firmware to the MCU - using the SWD protocol.

To compile & install openocd from source-code:

1. install compilers and library dependencies

       sudo apt-get install build-essential git autoconf libtool make pkg-config libusb-1.0-0 libusb-1.0-0-dev libgpiod-dev

2. download openocd source-code

       git clone https://git.code.sf.net/p/openocd/code openocd
       cd openocd
       git reset --hard v0.12.0

3. compile

       ./configure --prefix=/usr --sysconfdir=/etc --enable-imx_gpio --enable-linuxgpiod --disable-sysfsgpio --disable-stlink --disable-ti-icdi --disable-ulink --disable-usb-blaster-2 --disable-vsllink --disable-xds110 --disable-osbdm --disable-opendous --disable-aice --disable-usbprog --disable-rlink --disable-armjtagew --disable-cmsis-dap --disable-kitprog --disable-usb-blaster --disable-presto --disable-openjtag --disable-jlink --disable-buspirate --disable-esp-usb-jtag --disable-cmsis-dap-v2 --disable-ft232r --disable-ftdi
       make -j2

4. install

       sudo make install

### Install firmware

0. Substitute MCU-specific flash locations for loader & application in steps 6 & 7:

   - Nina-B1:
     - boot: `0x0000`
     - application: `0x8000`
   - Nina-B3:
     - boot: `0x0000`
     - application: `0xc000`
   - FWM7BLZ22:
     - boot: `0x0000`
     - application: `0x8000`

1. Generate platform-specific openocd configuration

   - i.MX8MQ SoM

         source [find interface/imx-native.cfg]
         transport select swd
         set WORKAREASIZE 0
         source [find target/nrf52.cfg]
         imx_gpio_swd_nums 8 66

   - SolidSense N6: Sink 1

         source [find interface/imx-native.cfg]
         transport select swd
         set WORKAREASIZE 0
         source [find target/nrf52.cfg]
         imx_gpio_swd_nums 82 81

   - SolidSense N6: Sink 2

         source [find interface/imx-native.cfg]
         transport select swd
         set WORKAREASIZE 0
         source [find target/nrf52.cfg]
         imx_gpio_swd_nums 59 125

2. Launch OpenOCD (server process)

       sudo openocd --file ./openocd.cfg

3. Connect to OpenOCD console using telnet (in separate console)

       telnet 127.0.0.1 4444
       Trying 127.0.0.1...
       Connected to 127.0.0.1.
       Escape character is '^]'.
       Open On-Chip Debugger
       >

4. Check MCU status

       > targets
           TargetName         Type       Endian TapName            State
       --  ------------------ ---------- ------ ------------------ ------------
        0* nrf52.cpu          cortex_m   little nrf52.cpu          running

**If the table shows State as "running" - then the MCU must be stopped** by running the halt command:

       > halt
       [nrf52.cpu] halted due to debug-request, current mode: Thread
       xPSR: 0x21000000 pc: 0x00012c68 psp: 0x20000288

5. Erase flash

       > nrf5 mass_erase
       RF52833-xxAA(build code: A0) 512kB Flash, 128kB RAM
       Mass erase completed.

6. program loader

       > program /home/debian/boot.elf.bin 0x0000
       ...
       ** Programming Finished **

**Note: Specify correct offset for target MCU (0x0000)!**

7. program application

       > program /home/debian/nimble.img 0x8000
       ...
       ** Programming Finished **

**Note: Specify correct offset for target MCU (0x0000)!**

8. restart MCU

       > reset

## Program MAC Address

Firmware expects a permanent MAC address stored in UICR registers.

MAC can be programmed with OpenOCD using the instructions above but stopping before step 5.
Then:

1. read existing UICR registers (on openocd console)

       flash read_bank 1 /home/debian/mac

2. write mac address as binary data to the UICR register dumpfile (on linux console)

       # 75:c1:1d:09:1c:07
       printf '\x07\x1c\x09\x1d\xc1\x75' | dd of=/home/debian/mac bs=1 seek=128 count=6 conv=notrunc

2. write mac address to UICR registers (on openocd console)

       > flash erase_sector 1 0 last
       > flash write_bank 1 /home/debian/mac

## Initialise BLE HCI Device in Linux

The device should now be ready to be initialized under Linux.

```no-highlight
sudo btattach -B /dev/ttymxc3 -S 1000000 -N
# Note: btattach is available with bluez on debian
```

The hci interface should now be available.

```no-highlight
hciconfig
hci0:	Type: Primary  Bus: UART
	BD Address: BB:EC:E6:0D:DC:04  ACL MTU: 255:12  SCO MTU: 0:0
	UP RUNNING 
	RX bytes:231 acl:0 sco:0 events:16 errors:0
	TX bytes:112 acl:0 sco:0 commands:16 errors:0
```
