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

### Overview

This is a mynewt project skeleton for quickly and easily building the blehci
firmware that can be flashed to the SolidRun iMX8MQ SOM's NINA-B1 chip.  This
firmware allows the NINA-B1 to act like a normal HCI device and is compatible
with newer Bluez stacks.

### Building

Apache blehci contains an example Apache Mynewt Nimble application called blehci.
When loaded on compatible hardware this application allows the NINA-B1 to be 
used under Linux as a normal HCI device, using the traditional Bluez stack.

1. Download and install the requirements for SolidRun's Newt BSP.

You will need to download and install:
- the Apache Newt tool, as documented in the [mynewt documentation](https://mynewt.apache.org/latest/get_started/index.html)
- an embedded toolchain, e.g. `apt install gcc-arm-embedded`
- this fork of [openOCD for i.MX SoCs](https://github.com/SolidRun/openocd/blob/master/doc/BUILD-IMX.md).

As a shortcut and to avoid installing all development files to the target systems, a [fork of the mynewt docker image](https://github.com/Josua-SR/newt-docker) built for arm64 is available at *quay.io/josua-sr/newt:latest*. For the steps below, installing an alias for the *newt* command to docker is sufficient:

```no-highlight
sudo apt install docker.io
sudo usermod -a -G docker <username>
# log out and back in
alias newt='docker run -e NEWT_HOST=$(uname) $ti --rm --device=/dev/bus/usb --privileged -v $(pwd):/workspace -w /workspace quay.io/josua-sr/newt:latest /newt'
```

2. Download the SolidRun BSP, Apache Mynewt Core package, and Nimble (executed from the mynewt-sr-blehci directory).

```no-highlight
newt upgrade
```

3. Build the Newt bootloader for the NINA-B1 using the "nina-b1_boot" target
(executed from the mynewt-sr-blehci directory).

```no-highlight
newt build nina-b1_boot
```

4. Build the blehci application for the NINA-B1 using the "blehci" target
(executed from the mynewt-sr-blehci directory).

```no-highlight
newt build blehci
newt create-image blehci 0.0.1
```

5. load the bootloader and application to the NINA-B1. *this is stored in the onboard flash and only needs to be done just once*
(executed from the mynewt-sr-blehci directory).

```no-highlight
newt load nina-b1_boot
newt load blehci
```
### Initializing under Linux 

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
