---
title: API Quick Reference
order: 2
section: API Reference
subsections:
  - API Commands
  - Async Notifications
---


# API Quick Reference

## Device IDs

(defined in OrbotixMsgSet.h)

val | name￼
----|-----
00h | DID_CORE
02h | DID_SPHERO
￼
## Core Commands, DID = 00h

(defined in OrbotixMsgSet.h)

val | name￼
----|-----
01h | CMD_PING
02h | CMD_VERSION
10h | CMD_SET_BT_NAME
11h | CMD_GET_BT_NAME
12h | CMD_SET_AUTO_RECONNECT
13h | CMD_GET_AUTO_RECONNECT
20h | CMD_GET_PWR_STATE
21h | CMD_SET_PWR_NOTIFY
22h | CMD_SLEEP
25h | SET_INACTIVE_TIMER
30h | CMD_GOTO_BL
40h | CMD_RUN_L1_DIAGS
41h | CMD_RUN_L2_DIAGS

## Sphero Commands, DID = 02h

(defined in OrbotixMsgSet.h)

val | name￼
----|-----
01h | CMD_SET_CAL
02h | CMD_SET_STABILIZ
03h | CMD_SET_ROTATION_RATE
06h | CMD_REENABLE_DEMO
09h | CMD_SELF_LEVEL
11h | CMD_SET_DATA_STREAMING
12h | CMD_SET_COLLISION_DET
13h | CMD_LOCATOR
14h | CMD_SET_ACCELERO
15h | CMD_READ_LOCATOR
20h | CMD_SET_RGB_LED
21h | CMD_SET_BACK_LED
22h | CMD_GET_RGB_LED
30h | CMD_ROLL
31h | CMD_BOOST
32h | CMD_MOVE
33h | CMD_SET_RAW_MOTORS
34h | CMD_SET_MOTION_TO
35h | CMD_SET_OPTIONS_FLAG
36h | CMD_GET_OPTIONS_FLAG
37h | CMD_SET_TEMP_OPTIONS_FLAG
38h | CMD_GET_TEMP_OPTIONS_FLAG
50h | CMD_RUN_MACRO
51h | CMD_SAVE_TEMP_MACRO
52h | CMD_SAVE_MACRO
54h | CMD_INIT_MACRO_EXECUTIVE
55h | CMD_ABORT_MACRO
56h | CMD_MACRO_STATUS
57h | CMD_SET_MACRO_PARAM
58h | CMD_APPEND_TEMP_MACRO_CHUNK
60h | CMD_ERASE_ORBBAS
61h | CMD_APPEND_FRAG
62h | CMD_EXEC_ORBBAS
63h | CMD_ABORT_ORBBAS
64h | CMD_ANSWER_INPUT

# API Commands

From here forward the redundant fields in both transmit and receive packets will be omitted for clarity.
We assume the `MRSP` is `00h` (for success), `SEQ` is echoed and `CHK` is computed correctly both ways.

## Core

The Core Device encapsulates actions that are fundamental to all Orbotix devices.

### Ping – 01h

#### Command

>

DID | CID | SEQ   | DLEN
----|-----|-------|------
00h | 01h | &#60;any&#62; | 01h


#### Response

>

|DLEN|
|----|
|01h |

The Ping command is used to verify both a solid data link with the Client and that Sphero is awake and dispatching commands.
Even though Ping is neither a set or get format command, it still enjoys a Simple Response.

### Get Versioning – 02h

#### Command

DID | CID
----|----
00h | 02h

#### Response

DLEN | &#60;data&#62;
-----|-------
0Bh  | see below

The Get Versioning command returns a whole slew of software and hardware information.
It’s useful if your Client Application requires a minimum version number of some resource within Sphero in order to operate.

The data record structure is comprised of fields for each resource that encodes the version number according to the specified format.

Name    | Byte index   | Description
-----   | ------------ | ------------
RECV    | 0            | This record version number, currently set to 02h. This will increase when more resources are added.
MDL     | 1            | Model number; currently 02h for Sphero
HW      | 2            | Hardware version code (ranges 1 through 9)
MSA-ver | 3            | Main Sphero Application version byte
MSA-rev | 4            | Main Sphero Application revision byte
BL      | 5            | Bootloader version in packed nibble format (i.e. 32h is version 3.2)
BAS     | 6            | orbBasic version in packed nibble format (i.e. 4.4)
MACRO   | 7            | Macro executive version in packed nibble format (4.4)
API-maj | 8            | major revision code this firmware implements
API-min | 9            | API minor revision code this firmware implements

### Set Device Name – 10h

#### Command

>

DID | CID | SEQ   | DLEN         | &#60;data&#62;
----|-----|-------|--------------|---------
00h | 10h | &#60;any&#62; | &#60;data&#62; + 01h | text name

#### Response

>

    Simple Response

This formerly reprogrammed the Bluetooth module to advertise with a different name, but this is no longer the case.
This assigned name is held internally and produced as part of the Get Bluetooth Info service below.
Names are clipped at 48 characters in length to support UTF-8 sequences; you can send something longer but the extra will be discarded.
This field defaults to the Bluetooth advertising name.

To alter the Bluetooth advertising name from the standard Sphero-RGB pattern you will need to $$$ into the RN-42 within 60 seconds after power up, issue the command `SN,mynewname` and finish with `r,1` to reboot the module.

### Get Bluetooth Info – 11h

#### Command

>

DID | CID
----|----
00h | 11h


#### Response

>

DLEN  | 16 bytes     | 12 bytes     | &#60;byte&#62;    | 3 bytes
----- | ------------ | ------------ | --------- | -----------
21h   | ASCII name   | ASCII BTA    | 00h       | ID colors

This returns:

- the ASCII name of the ball (defaults to Bluetooth advertising name)
- the Bluetooth address in ASCII
- ID colors the ball blinks when not connected

The ASCII name field is padded with zeros to its maximum size.

This is provided as a courtesy for Clients that have don’t have a method to interrogate their underlying Bluetooth stack for this information.

### Set Auto Reconnect – 12h

#### Command

>

DID | CID | SEQ   | DLEN | data 0 | data 1
----|-----|-------|------|--------|-------
00h | 12h | &#60;any&#62; | 03h  | flag   | time

#### Response

>

    Simple Response

This configures the control of the Bluetooth module in its attempt to automatically reconnect with the last mobile Apple device.
This is a courtesy behavior since the Apple Bluetooth stack doesn't initiate automatic reconnection on its own.
The two parameters are simple: `flag` is `00h` to disable or `01h` to enable, and `time` is the number of seconds after power-up in which to enable auto reconnect mode.
For example, if `time` = `30` then the module will be attempt reconnecting 30 seconds after waking up.
(refer to RN-APL-EVAL pg. 7 for more info)

### Get Auto Reconnect – 13h

#### Command

>

DID |￼CID | SEQ   |￼DLEN
----|-----|-------|-----
00h |￼13h | &#60;any&#62; |￼01h


#### Response

>

DLEN |￼data 0 |￼data 1
-----|--------|------
03h  |￼flag   |￼time

This returns the Bluetooth auto reconnect values as defined in the “Set Auto Reconnect” command.

### Get Power State – 20h

#### Command

>

DID | CID
----|----
00h | 20h

#### Response

>

DLEN | &#60;data&#62;
-----|-------
09h  | see below

This returns the current power state and some additional parameters to the Client.

They are detailed below.

offset  | name         | description
------- | ------       | -----------
00h     | RecVer       | Record version code – the following definition is for 01h
01h     | Power State  | High-level state of the power system as concluded by the power manager: 01h = Battery Charging, 02h = Battery OK, 03h = Battery Low, 04h = Battery Critical
02h     | BattVoltage  | Current battery voltage scaled in 100ths of a volt; 02EFh would be 7.51 volts (unsigned 16-bit value)
04h     | NumCharges   | Number of battery recharges in the life of this Sphero (unsigned 16-bit value)
06h     | TimeSinceChg | Seconds awake since last recharge (unsigned 16-bit value)

### Set Power Notification – 21h

#### Command

>

DID | CID | SEQ   | DLEN | data
----|-----|-------|------|-----
00h | 21h | &#60;any&#62; | 2h   | flag

#### Response

>

    Simple Response

This enables Sphero to asynchronously notify the Client periodically with the power state or immediately when the power manager detects a state change.
Timed notifications arrive every 10 seconds until they're explicitly disabled or Sphero is unpaired.
The flag is as you would expect, `00h` to disable and `01h` to enable.
This setting is volatile and therefore not retained across sleep cycles.

The complete power notification message is of the form:

SOP1 | SOP2 | CODE | DLEN-MSB | DLEN-LSB | data  | CHK
-----|------|------|----------|----------|-------|-----
Fh   | FEh  | 01h  | 00h      | 02h      | state | &#60;cmp&#62;

The power state byte mimics that of `CID` `20h` above:

- `01h` = Battery Charging
- `02h` = Battery OK
- `03h` = Battery Low
- `04h` = Battery Critical

### Sleep – 22h

#### Command

>

DID | CID | SEQ   | DLEN | Wakeup       | Macro | orbBasic
----|-----|-------|------|--------------|-------|---------
00h | 22h | &#60;any&#62; | 06h  | 16-bit val | val | 16-bit val

#### Response

>

    Simple Response

This command puts Sphero to sleep immediately.
There are three optional parameters that program the robot for future actions:

name     | description
---------|------------
Wakeup   | The number of seconds for Sphero to sleep for and then automatically reawaken. Zero does not program a wakeup interval, so he sleeps forever. FFFFh attempts to put him into deep sleep (if supported in hardware) and returns an error if the hardware does not support it.
Macro    | If non-zero, Sphero will attempt to run this macro ID upon wakeup.
orbBasic | If non-zero, Sphero will attempt to run an orbBasic program in Flash from this line number.

### Set Inactivity Timeout – 25h

#### Command

>

DID | CID | SEQ   | DLEN | TIME
----|-----|-------|------|-------------
00h | 25h | &#60;any&#62; | 03h  | 16-bit val

#### Response

>

    Simple Response

To save battery power, Sphero normally goes to sleep after a period of inactivity.
From the factory this value is set to 600 seconds (10 minutes) but this API command can alter it to any value of 60 seconds or greater.

The inactivity timer is reset every time an API command is received over Bluetooth or a shell command is executed in User Hack mode.
In addition, the timer is continually reset when a macro is running unless the `MF_STEALTH` flag is set, and the same for orbBasic unless the `BF_STEALTH` flag is set.

### Jump To Bootloader – 30h

#### Command

>

￼DID | CID
-----|------
￼00h |￼30h

#### Response

>

    Simple Response

This command requests a jump into the Bootloader to prepare for a firmware download.
It always succeeds, because you can always stop where you are, shut everything down and transfer execution.
All commands after this one must comply with the Bootloader Protocol Specification, which is a separate document.

Note that just because you can always vector into the Bootloader, it doesn't mean you can get anything done.
Further details are explained in the associated document but in short: the Bootloader doesn't implement the entire Core Device message set and if the battery is deemed too low to execute reflashing operations, all you can do is return to the Main Application.

### Perform Level 1 Diagnostics – 40h

#### Command

>

DID | CID
----|------
00h | 40h

#### Response 1:

>

    Simple Response

#### Response 2:

>

SOP1 | SOP2 | CODE | DLEN-MSB | DLEN-LSB | data   | CHK
-----|------|------|----------|----------|--------|-----
FFh  | FEh  | 02h  | &#60;msb&#62;    | &#60;lsb&#62;    | &#60;data&#62; | &#60;cmp&#62;

This is a developer-level command to help diagnose aberrant behavior.
Most system counters, process flags, and system states are decoded into human readable ASCII.
There are two responses to this command: a Simple Response followed by a large async message containing the results of the diagnostic tests.

As of FW version 0.99, the answer was well over 1K in length and similar to:

```
[System]
Mode F, Boot code 12
0 rechrg, 32 min
since last, 0:51
alive
Cold:13, Warm:0,
Wakeup:0, NMI:0,
Hard:0
Dist rolled: 0, Vbatt
7.85, state: OK
SensorsHthy:1
BTError:0
AuthOK:1
Stabilize:1
TestPin:0
AutoRN:0
Mac:0
Bootldr=1.7
MA=0.98
Board=2
OrbBasic=0.8
MacExec=2
CB=111
AutoRecon En=1
AutoReconDel=0
ClientTimeOut=300
WakeUpSec=0

[Network]
Rx good:7, bad:1,
Tx:780
Rx overruns:0, Tx:0
Dev name:Sphero-OWG,
BTA:0006664440B8
BTver:Ver 5.36 IAP
11/04/11

[Sensors]
Fail: 0 Loc: 0 Code:
0

[Accel]
Xsc=0.0039 Ysc=0.0040
Zsc=0.0039
Xb=-0.0078 Yb=0.0010
Zb=0.0552

[Gyro]
Xsc=0.0680 Ysc=0.0683
Zsc=0.0680
Xb=-12.3322 Yb=-
10.2964 Zb=-28.3654
Temp=35
Therm: Xb1=-11.9700
Xb2=-37.6833 Yb1=-
10.0140 Yb2=-30.0675
Zb1=-29.7397
Zb2=67.8367
Tmp1=34 Tmp2=105
Xsl=-0.3622
GyroAdjCnt=0
Xint=0.3434 Ysl=-
0.2824 Yint=-0.4109
Zsl=1.3743 Zint=-
76.4665

[Control]
Pitch P=60.000
D=50.000
I=0.200 D=100.000
Roll P=21.000 I=0.300
D=1200.000
RotRate=0.228
GTtr=0x1
MinClks:777
Yaw P=90.000 I=0.230
[Test Res]
PCBAtr=0x3ff Stn=7
AGtr=0x1 Stn=1

[Idle loop]
MaxClks:73987
MinFreq:51875
MaxFreq:104952
CPU 56% idle
```

### Perform Level 2 Diagnostics – 41h

#### Command

>

DID | CID
----|----
00h | 41h

#### Response

>

    See below

This is a developers-only command to help diagnose aberrant behavior.
It is much less informative than the Level 1 command but it is in binary format and easier to parse.

Here is the layout of the data record which is currently 58h bytes long:

offset  | name               | description
------- | ------             | ------------
00h     | RecVer             |￼Record version code – the following definition is for 01h
02h     |￼&#60;empty&#62;            |￼Reserved
03h     | Rx_Good            | Good packets received (unsigned 32-bit value)
07h     | Rx_Bad_DID         | Packets with a bad Device ID (unsigned 32-bit value)
0Bh     | Rx_Bad_DLEN        | Packets with a bad data length (unsigned 32-bit value)
0Fh     | Rx_Bad_CID         | Packets with a bad Command ID (unsigned 32-bit value)
13h     | Rx_Bad_CHK         | Packets with a bad checksum (unsigned 32-bit value)
17h     | Rx_Buff_Ovr        | Receive buffer overruns (unsigned 32-bit value)
1Bh     | Tx_Msgs            | Messages transmitted (unsigned 32-bit value)
1Fh     | Tx_Buff_Ovr        | Transmit buffer overruns (unsigned 32-bit value)
23h     | LastBootReason     | Reason for last boot (8-bit value)
24h     | BootCounters       | 16 different counts of boot reasons
44h     | &#60;empty&#62;            | Reserved
46h     | ChargeCount        | Charge cycles (unsigned 16-bit value)
48h     | SecondsSinceCharge | Awake time in seconds since last charge (unsigned 16-bit value)
4Ah     | SecondsOn          | Life awake time in seconds (unsigned 32-bit value)
4Eh     | DistanceRolled     | Distance rolled (unsigned 32-bit value)
52h     | Sensor Failures    | Count of I2C bus failures (unsigned 16-bit value)
54h     | Gyro Adjust Count  | Lifetime count of automatic GACs (unsigned 32-bit value)

## Sphero

These commands are specific to the features that Sphero offers.

### Set Heading – 01h

#### Command

>

DID | CID | SEQ   | DLEN | HEADING
----|-----|-------|------|--------
02h | 01h | &#60;any&#62; | 03h  | 16-bit val

#### Response

>

    Simple Response

This allows the smartphone client to adjust the orientation of Sphero by commanding a new reference heading in degrees, which ranges from 0 to 359.
You will see the ball respond immediately to this command if stabilization is enabled.

In FW version 3.10 and later this also clears the maximum value counters for the rate gyro, effectively re-enabling the generation of an async message alerting the client to this event.

### Set Stabilization – 02h

#### Command

>

DID | CID | SEQ   | DLEN | FLAG
----|-----|-------|------|--------
02h | 02h | &#60;any&#62; | 02h  | &#60;bool&#62;

#### Response

>

    Simple Response

This turns on or off the internal stabilization of Sphero, in which the IMU is used to match the ball's orientation to its various set points.
The flag value is as you would expect, 00h for off and 01h for on.
Stabilization is enabled by default when Sphero powers up.
You will want to disable stabilization when using Sphero as an external input controller or even to save battery power during testing that doesn't involve movement (orbBasic, etc.)

An error is returned if the sensor network is dead; without sensors the IMU won't operate and thus there is no feedback to control stabilization.

### Set Rotation Rate – 03h

#### Command

>

DID | CID | SEQ   | DLEN | RATE
----|-----|-------|------|--------
02h | 03h | &#60;any&#62; | 02h  | value

#### Response

>

    Simple Response

This allows you to control the rotation rate that Sphero will use to meet new heading commands (`DID` `02h`, `CID` `01h`).
A lower value offers better control but with a larger turning radius.
A higher value will yield quick turns but Sphero may roll over on itself and lose control.

The commanded value is in units of 0.784 degrees/sec.
So, setting a value of `C8h` will set the rotation rate to 157 degrees/sec.
A value of `255` jumps to the maximum (currently 400 degrees/sec).
A value of zero doesn't make much sense so it's interpreted as 1, the minimum.

### Set Application Configuration Block – 04h

#### Command

>

DID | CID | SEQ   | DLEN | DATA
----|-----|-------|------|--------
02h | 04h | &#60;any&#62; | 21h  | 20h bytes

#### Response

>

    Simple Response

This allows you to write a 32 byte block of data from the configuration block that is set aside for exclusive use by applications.
The Sphero firmware does not interpret it on the way in or out.

### Get Application Configuration Block – 05h

#### Command

>

DID | CID | SEQ   | DLEN
----|-----|-------|-----
02h | 05h | &#60;any&#62; | 01h

#### Response

>

    Simple Response

This allows you to retrieve the application configuration block that is set aside for exclusive use by applications.

### Self Level – 09h

#### Command

>

DID | CID | SEQ   | DLEN |  Options | Angle Limit | Timeout | True Time
----|-----|-------|------|----------|-------------|---------|----------
02h | 09h | &#60;any&#62; | 05h  | &#60;byte&#62;   |  &#60;byte&#62;     |  &#60;byte&#62; |  &#60;byte&#62;

#### Response

>

    Simple Response

#### asynchronous response message

>

SOP1 | SOP2 |  CODE |  DLEN-MSB |  DLEN_LSB |  data  | CHK
-----|------|-------|-----------|-----------|--------|-----
FFh  | FEh  | 0Bh   | 00h       | 02h       | result | &#60;cmp&#62;

This command controls the self level routine.
The self level routine attempts to achieve a horizontal orientation where pitch and roll angles are less than the provided Angle Limit.
After both angle limits are satisfied, option bits control sleep, final angle (heading), and control system on/off.

An asynchronous message is returned when the self level routine completes (only when started by API call).

The required parameters are:

Name           | Value    | Description
-----          | -------  | -----------
Start/Stop     | Bit 0    | 0 aborts the routine if in progress. 1 starts the routine.
Final Angle    | Bit 1    | 0 just stops. 1 rotates to heading equal to beginning heading.
Sleep          | Bit 2    | 0 stays awake after leveling. 1 goes to sleep after leveling.
Control System | Bit 3    | 0 leaves control system off. 1 leaves control system on (after leveling).
Angle Limit    | 0        | Use the default value
Angle Limit    | 1 to 90  | Set the max angle for completion (in degrees)
Timeout        | 0        | Use the default value
Timeout        | 1 to 255 | Set maximum seconds to run the routine
True Time      | 0        | Use the default value
True Time      | 1 to 255 | Set the required “test for levelness” time to 10*True Time (in milliseconds)

Default values are:
- `Angle` = 3
- `Timeout` = 15
- `True Time` = 30 (300 milliseconds)

`True Time * 10` specifies the number of milliseconds that the pitch and roll angles must remain below the `Angle Limit` after the routine completes.
If one of the values exceeds the `Angle Limit`, the ball will self level again and the accuracy timer will start again from 0.

### Result byte

The result byte can be:

00h | Unknown
----|--------
01h | Timed Out (level was not achieved)
02h | Sensors Error
03h | Self Level Disabled (see Option Flags)
04h | Aborted (by API call)
05h | Charger not found
06h | Success

### Self Level Angle Accuracy

We have found that the real angle lags a bit behind the measured angle.
Also, the angles may shift some after “level” is achieved as the motors stop and the system comes to a rest.
A `True Time` value of 30 (300 milliseconds) is generally good enough to keep the angles within a degree or two of the specified `Angle Limit`.
If greater accuracy is required the `True Time` value can be increased up to 255 (2.55 seconds).

### Control System On/Off

When the control system is off, obviously self leveling can not happen.
There are several paths to this state:

* If the sensors are determined to be in an error state, self leveling will be skipped.
  Sleep requests will still trigger the go to sleep routine.
* The control system can be turned off using the “Set Stabilization” API call.
  This is used for certain games where Sphero is held in the hand as a controller.
* The control system can be turned off by a macro.
* The control system can be turned off using the shell command “l0”.
* The control system can be turned off using the shell command “x11”.
* The control system can be turned off through an orbBasic program.

When self level is called, leveling is skipped if the sensors are dead, as there is no recourse to this.
For all the other cases, the self level routine runs.
Since we have the System Options Flag to disable the self level routine, it is easy to override this behavior.
Use the control system on/off bit to specify whether to leave the control system on or off after the self level routine is complete.

The current behavior is if a macro or orbBasic program is running and the ball starts charging, the self level routine runs (but it doesn't go to sleep).
This could be desired behavior for some programs.

### System Options Flag

Refer to `DID 02h`, `CID 35h` for details.
Sleep requests made using this self level API call while the disable flag is asserted will still cause the ball to go to sleep.

### Set Data Streaming – 11h

#### Command

>

DID | CID | SEQ   | DLEN       | N           | M           | MASK        |  PCNT     | MASK2
----|-----|-------|------------|-------------|-------------|-------------|-----------|------
02h | 11h | &#60;any&#62; | 0ah or 0eh |  16-bit val |  16-bit val |  32-bit val | 8-bit val | 32-bit val

#### Response

>

    Simple Response

Sphero supports asynchronous data streaming of certain control system and sensor parameters.
This command selects the internal sampling frequency, packet size, parameter mask and optionally, the total number of packets.

param | description
------|------------
N     | Divisor of the maximum sensor sampling rate
M     | Number of sample frames emitted per packet
MASK  | Bitwise selector of data sources to stream
PCNT  | Packet count 1-255 (or 0 for unlimited streaming)
MASK2 | Bitwise selector of more data sources to stream (optional)

`MASK` and `PCNT` are pretty obvious but the `N`, `M` terms bear a little more explanation.
Currently the control system runs at 400Hz and because it's pretty unlikely you will want to see data at that rate, `N` allows you to divide that down.
`N` = 2 yields data samples at 200Hz, `N` = 10, 40Hz, etc.
Every data sample consists of a "frame" made up of the individual sensor values as defined by the `MASK`.
The `M` value defines how many frames to collect in memory before the packet is emitted.
In this sense, it controls the latency of the data you receive.
Increasing `N` and the number of bits set in `MASK` drive the required throughput.
You should experiment with different values of `N`, `M` and `MASK` to see what works best for you.

The `MASK2` bitfield was added to extend `MASK` when we developed more than 32 data sources.
The API processor is implemented so that this value is optional; if it isn't included then all of its bits are set to zero.
(Added in FW 1.15)

Each parameter is returned as a 16-bit signed integer.
The table below defines the bits in `MASK` to those parameters with the indicated ranges and units.
If the command is issued with a `MASK` of zero, then data streaming is disabled.

### MASK

bit   | sensor   | range units/LSB
----- | -------- | -----------------
8000  | 0000h    | accelerometer axis X, raw -2048 to 2047 4mG
4000  | 0000h    | accelerometer axis Y, raw -2048 to 2047 4mG
2000  | 0000h    | accelerometer axis Z, raw -2048 to 2047 4mG
1000  | 0000h    | gyro axis X, raw  -32768 to 32767 0.068 degrees
0800  | 0000h    | gyro axis Y, raw  -32768 to 32767 0.068 degrees
0400  | 0000h    | gyro axis Z, raw  -32768 to 32767 0.068 degrees
0200  | 0000h    | Reserved
0100  | 0000h    | Reserved
0080  | 0000h    | Reserved
0040  | 0000h    | right motor back EMF, raw -32768 to 32767 22.5 cm
0020  | 0000h    | left motor back EMF, raw  -32768 to 32767 22.5 cm
0010  | 0000h    | left motor, PWM, raw  -2048 to 2047 duty cycle
0008  | 0000h    | right motor, PWM raw  -2048 to 2047 duty cycle
0004  | 0000h    | IMU pitch angle, filtered -179 to 180 degrees
0002  | 0000h    | IMU roll angle, filtered  -179 to 180 degrees
0001  | 0000h    | IMU yaw angle, filtered -179 to 180 degrees
0000  | 8000h    | accelerometer axis X, filtered  -32768 to 32767 1/4096 G
0000  | 4000h    | accelerometer axis Y, filtered  -32768 to 32767 1/4096 G
0000  | 2000h    | accelerometer axis Z, filtered  -32768 to 32767 1/4096 G
0000  | 1000h    | gyro axis X, filtered -20000 to 20000 0.1 dps
0000  | 0800h    | gyro axis Y, filtered -20000 to 20000 0.1 dps
0000  | 0400h    | gyro axis Z, filtered -20000 to 20000 0.1 dps
0000  | 0200h    | Reserved
0000  | 0100h    | Reserved
0000  | 0080h    | Reserved
0000  | 0040h    | right motor back EMF, filtered  -32768 to 32767 22.5 cm
0000  | 0020h    | left motor back EMF, filtered -32768 to 32767 22.5 cm
0000  | 0010h    | Reserved 1
0000  | 0008h    | Reserved 2
0000  | 0004h    | Reserved 3
0000  | 0002h    | Reserved 4
0000  | 0001h    | Reserved 5

### MASK2

bit  | sensor   | range units
---- | -------- | -------------
8000 | 0000h    | Quaternion Q0 -10000 to 10000 1/10000 Q
4000 | 0000h    | Quaternion Q1 -10000 to 10000 1/10000 Q
2000 | 0000h    | Quaternion Q2 -10000 to 10000 1/10000 Q
1000 | 0000h    | Quaternion Q3 -10000 to 10000 1/10000 Q
0800 | 0000h    | Odometer X  -32768 to 32767 cm
0400 | 0000h    | Odometer Y  -32768 to 32767 cm
0200 | 0000h    | AccelOne  0 to 8000 1 mG
0100 | 0000h    | Velocity X  -32768 to 32767 mm/s
0080 | 0000h    | Velocity Y  -32768 to 32767 mm/s

### Configure Collision Detection – 12h

#### Command

>

DID | CID | SEQ   | DLEN |  Meth |  Xt   | Xspd  | Yt    | Yspd  | Dead
----|-----|-------|------|-------|-------|-------|-------|-------|------
02h | 12h | &#60;any&#62; | 07h  | val | val | val | val | val | val

#### Response

>

    Simple Response

Sphero contains a powerful analysis function to filter accelerometer data in order to detect collisions.
Because this is a great example of a high-level concept that humans excel and – but robots do not – a number of parameters control the behavior.
When a collision is detected an asynchronous message is generated to the client.

### Configuration fields

The configuration fields are defined as follows:

Param      | Description
-----------|------------
Meth       | Detection method type to use. Currently the only method supported is 01h. Use 00h to completely disable this service.
Xt, Yt     | An 8-bit settable threshold for the X (left/right) and Y (front/back) axes of Sphero. A value of 00h disables the contribution of that axis.
Xspd, Yspd | An 8-bit settable speed value for the X and Y axes. This setting is ranged by the speed, then added to Xt, Yt to generate the final threshold value.
Dead       | An 8-bit post-collision dead time to prevent retriggering; specified in 10ms increments.

### Payload

The data payload of the async message is 10h bytes long and formatted as follows:

X            | Y             | Z             | Axis           |  xMagnitude  |  yMagnitude   |  Speed       | Timestamp
-------------|---------------|---------------|----------------|--------------|---------------|--------------|-------------
16-bit val |  16-bit val |  16-bit val |  8-bit field | 16-bit val |  16-bit val |  8-bit val | 32-bit val

Param      | Description
-----------|------------
X, Y, Z    | Impact components normalized as a signed 16-bit value. Use these to determine the direction of collision event. If you don't require this level of fidelity, the two Magnitude fields encapsulate the same data in pre-processed format.
Axis       | This bitfield specifies which axes had their trigger thresholds exceeded to generate the event. Bit 0 (01h) signifies the X axis and bit 1 (02h) the Y axis.
xMagnitude | This is the power that crossed the programming threshold Xt + Xs.
yMagnitude | This is the power that crossed the programming threshold Yt + Ys.
Speed      | The speed of Sphero when the impact was detected.
Timestamp  | The millisecond timer value at the time of impact; refer to the documentation of CID 50h and 51h to make sense of this value.

### Additional information

For additional information, refer to `SPAN01`, "Sphero Collision Detection Feature."
Note also that this feature relies on the accelerometer range being set to ±8Gs; if altered with the next command then don't count on it working in a useful way.

### Configure Locator – 13h

#### Command

>

DID | CID | SEQ   | DLEN |  Flags | X             | Y             | Yaw Tare
----|-----|-------|------|--------|---------------|---------------|-------------------
02h | 13h | &#60;any&#62; | 02h  | 8 bit  | 16 bit signed | 16 bit signed | 16 bit signed

#### Response

>

    Simple Response

Through the streaming interface, Sphero provides real-time location data in the form of (X,Y) coordinates on the ground plane.
When Sphero wakes up it has coordinates (0,0) and heading 0, which corresponds to facing down the positive Y-axis with the positive X-axis to your right.
This command allows you to move Sphero to a new location and change the alignment of locator coordinates with IMU headings.

When Sphero receives a Set Heading command it changes which direction corresponds to heading 0.
By default, the locator compensates for this by modifying its value for yaw tare so that the Y-axis is still pointing in the same real-world direction.
For instance, if you wake up Sphero and drive straight you will be driving down the Y-axis.
If you use the Set Heading feature in the drive app to turn 90 degrees, you will still have heading 0, but the locator knows you have turned 90 degrees and are now facing down the X-axis.
This feature can be turned off, in which case the locator knows nothing about the Set Heading command.
This can lead to some strange results.
For instance, if you drive using only roll commands with heading 0 and set heading commands to change direction the locator will perceive your entire path as lying on the Y-axis.

Parameters | Description
---------- | -----------
Flags      | Bit 0 – Determines whether calibrate commands automatically correct the yaw tare value. When false, the positive Y axis coincides with heading 0 (assuming you do not change the yaw tare manually using this API command). Other Bits - Reserved
X, Y       | The current (X,Y) coordinates of Sphero on the ground plane in centimeters.
Yaw Tare   | Controls how the X,Y-plane is aligned with Sphero’s heading coordinate system. When this parameter is set to zero, it means that having yaw = 0 corresponds to facing down the Y- axis in the positive direction. The value will be interpreted in the range 0-359 inclusive.

### Set Accelerometer Range – 14h

#### Command

>

DID | CID | SEQ   | DLEN |  Range Idx
----|-----|-------|------|------------
02h | 14h | &#60;any&#62; | 02h  | 8 bit val

#### Response

>

    Simple Response

Normally, Sphero's solid state accelerometer is set for a range of ±8Gs.
There may be times when you would like to alter this, say to resolve finer accelerations.
This command takes an index for the supported range as explained below.

Idx  | Range
-----|------
0    | ±2Gs
1    | ±4Gs
2    | ±8Gs (default)
3    | ±16Gs

Note that setting this to other than the default value will have indeterminate consequences for driving and collision detection.
You shouldn't expect either to work.

### Read Locator – 15h

#### Command

>

DID | CID | SEQ   | DLEN
----|-----|-------|-----
02h | 15h | &#60;any&#62; | 01h

#### Response

>

DLEN |  XPOS |  YPOS |  XVEL |  YVEL |  SOG
-----|-------|-------|-------|-------|------
0Bh  | 16 bit val  | 16 bit val | 16 bit val | 16 bit val | 16 bit val

This reads Sphero's current position (X,Y), component velocities and SOG (speed over ground).
The position is a signed value in centimeters, the component velocities are signed cm/sec while the SOG is unsigned cm/sec.

### Set RGB LED Output – 20h

#### Command

>

DID | CID | SEQ   | DLEN |  RED    | GREEN   | BLUE    | FLAG
----|-----|-------|------|---------|---------|---------|-------
02h | 20h | &#60;any&#62; | 05h  | value | value | value | &#60;bool&#62;

#### Response

>

    Simple Response

This allows you to set the RGB LED color.
The composite value is stored as the "application LED color" and immediately driven to the LED (if not overridden by a macro or orbBasic operation).
If `FLAG` is true, the value is also saved as the "user LED color" which persists across power cycles and is rendered in the gap between an application connecting and sending this command.

### Set Back LED Output – 21h

#### Command

>

DID | CID | SEQ   | DLEN |  BRIGHT
----|-----|-------|------|--------
02h | 21h | &#60;any&#62; | 02h  | value

#### Response

>

    Simple Response

This allows you to control the brightness of the back LED.
The value does not persist across power cycles.

### Get RGB LED – 22h

#### Command

>

DID | CID | SEQ   | DLEN
----|-----|-------|-----
02h | 22h | &#60;any&#60; | 01h

#### Response

>

DLEN |  RED    | GREEN   | BLUE
-----|---------|---------|-----
04h  | value | value | value

This retrieves the "user LED color" which is stored in the config block.
This may or may not be actively driven to the RGB LED.

### Roll – 30h

#### Command

>

DID | CID | SEQ   | DLEN |  Speed | Heading | Heading | STATE
----|-----|-------|------|--------|---------|---------|------
02h | 30h | &#60;any&#62; | 05h  | val  | &#60;msb&#62;   | &#60;lsb&#62;   | val

#### Response

>

    Simple Response

This commands Sphero to roll along the provided vector.
Both a speed and a heading are required; the latter is considered relative to the last calibrated direction.
A state value is also provided.
In the CES firmware, this was used to gate the control system to either obey the roll vector or ignore it and apply optimal braking to zero speed.
Please refer to Appendix C for detailed information.

The client convention for heading follows the 360 degrees on a circle, relative to the ball: 0 is straight ahead, 90 is to the right, 180 is back and 270 is to the left.
The valid range is 0..359.

### Set Raw Motor Values – 33h

#### Command

>

DID  | CID   | SEQ     | DLEN   | L-MODE       | L-POWER    | R-MODE   | R-POWER
---- | ----- | ------- | ------ | ------------ | ---------- | -------- | --------
02h  | 33h   | &#60;any&#62;   | 05h    | val          | val        | val      | val

#### Response

>

    Simple Response

This allows you to take over one or both of the motor output values, instead of having the stabilization system control them.
Each motor (left and right requires a mode (see below) and a power value from 0- 255.
This command will disable stabilization if both modes aren't "ignore" so you'll need to re-enable it via `CID 02h` once you're done.

MODE | Description
---- | -----------
00h  | Off (motor is open circuit)
01h  | Forward
02h  | Reverse
03h  | Brake (motor is shorted)
04h  | Ignore (motor mode and power is left unchanged)

### Set Motion Timeout – 34h

#### Command

>

DID | CID | SEQ   | DLEN | TIME
----|-----|-------|------|--------
02h | 34h | &#60;any&#62; | 03h  | 16-bit val

#### Response

>

    Simple Response

This sets the ultimate timeout for the last motion command to keep Sphero from rolling away in the case of a crashed (or paused) client app.
The `TIME` parameter is expressed in milliseconds and defaults to `2000` upon wake-up.

If the control system is enabled, the timeout triggers a stop otherwise it commands zero `PWM` to both motors.
This "termination behavior" is inhibited if a macro is running with the flag `MFEXCLUSIVEDRV` set, or an orbBasic program is executing with a similar flag, `BFEXCLUSIVEDRV`.

Note that you must enable this action by setting System Option Flag #4.

### Set Permanent Option Flags – 35h

#### Command

>

DID | CID | SEQ   | DLEN |  FLAGS
----|-----|-------|------|--------
02h | 35h | &#60;any&#62; | 05h  | 32-bit val

#### Response

>

    Simple Response

Assigns the permanent option flags to the provided value and writes them to the config block for persistence across power cycles.
See below for the bit definitions.

### Get Permanent Option Flags – 36h

#### Command

>

DID | CID | SEQ   | DLEN
----|-----|-------|-----
02h | 36h | &#60;any&#62; | 01h

#### Response

>

DLEN  | FLAGS
------|------
05h   | 32-bit val

Returns the permanent option flags as a bitfield as defined below:


bit # | Description
----- | -----------
0     | Set to prevent Sphero from immediately going to sleep when placed in the charger and connected over Bluetooth.
1     | Set to enable Vector Drive, that is, when Sphero is stopped and a new roll command is issued it achieves the heading before moving along it.
2     | Set to disable self-leveling when Sphero is inserted into the charger.
3     | Set to force the tail LED always on.
4     | Set to enable motion timeouts (see DID 02h, CID 34h)
5     | Set to enable retail Demo Mode (when placed in the charger, ball runs a slow rainbow macro for 60 minutes and then goes to sleep).
6-31  | Unassigned

### Set Temporary Option Flags – 37h

#### Command

>

DID | CID | SEQ   | DLEN |  FLAGS
----|-----|-------|------|--------
02h | 37h | &#60;any&#62; | 05h  | 32-bit val

#### Response

>

    Simple Response

Assigns the temporary option flags to the provided value.
These do not persist across a power cycle.
See below for the bit definitions.

### Get Temporary Option Flags – 38h

#### Command

>

DID | CID | SEQ   | DLEN
----|-----|-------|-----
02h | 38h | &#60;any&#62; | 01h

#### Response

>

DLEN  | FLAGS
------|------
05h   | 32-bit val

Returns the temporary option flags as a bitfield as defined below:

bit # | Description
----- | -----------
0     | Enable Stop On Disconnect behavior: when the Bluetooth link transitions from connected to disconnected, Sphero is commanded to stop rolling. This is ignored if a macro or orbBasic program is running though both have option flags to allow this during their execution. This flag is cleared after it is obeyed, thus it is a one-shot.
1-31  | Unassigned, return zero

### Run Macro – 50h

#### Command

>

DID | CID | SEQ   | DLEN |  ID
----|-----|-------|------|-----
02h | 50h | &#60;any&#62; | 02h  | 8-bit value

#### Response

>

    Simple Response

This attempts to execute the specified macro.
Macro IDs are organized into groups:

IDs    | Description
---    | -----------
01-31  | System Macros. Compiled into the Main Application. Always available to run, cannot be deleted.
32-253 | User Macros. Downloaded and permanently stored, can be deleted in total.
254    | Stream Macro, a special user macro that doesn't require this call to begin execution
255    | Temporary Macro, a special user macro that's held in RAM for execution

This command will fail if there is currently an executing macro or the specified ID Code isn't found.
In the case of the former, send an abort command first.

### Save Temporary Macro – 51h

#### Command

>

DID | CID | SEQ   | DLEN      | MACRO
----|-----|-------|-----------|--------
02h | 51h | &#60;any&#62; | &#60;len + 1&#62; | &#60;data&#62;

#### Response

>

    Simple Response

This stores the attached macro definition into the temporary RAM buffer for later execution.
Any existing macro ID can be sent through this command and it is then renamed to `ID FFh`.
If this command is sent while a Temporary or Stream Macro is executing it will be terminated so that its storage space can be overwritten.
As with all macros, the longest definition that can be sent is 254 bytes (thus requiring `DLEN` to be `FFh`).

You must follow this with a Run Macro command to begin execution.

### Save Macro – 52h

#### Command

>

DID | CID | SEQ   | DLEN      |  MACRO
----|-----|-------|-----------|--------
02h | 52h | &#60;any&#62; | &#60;len + 1&#62; | &#60;data&#62;

#### Response

>

     Simple Response

This stores the attached macro definition into the persistent store for later execution.
This command can be sent even if other macros are executing.
You will receive a failure response if you attempt to send an ID number in the System Macro range, `255` for the Temp Macro and ID of an existing user macro in the storage block.
As with all macros, the longest definition that can be sent is `254` bytes (thus requiring `DLEN` to be `FFh`).

A special case of this command is to start and continue execution of the Stream Macro, ID `254`.
If a Temporary Macro is running it will be terminated and the Stream Macro will begin.
If a Stream Macro is already running, this chunk will be appended (if there is room).
Stream Macros terminate via Abort or with a special `END` code.
Refer to the Sphero Macro documentation for more detail.

### Reinit Macro Executive – 54h

#### Command

>

DID | CID | SEQ   | DLEN
----|-----|-------|-----
02h | 54h | &#60;any&#62; | 01h

#### Response

>

    Simple Response

This terminates any running macro and reinitializes the macro system.
The table of any persistent user macros is cleared.

### Abort Macro – 55h

#### Command

>

DID | CID | SEQ   | DLEN
----|-----|-------|-----
02h | 55h | &#60;any&#62; | 01h

#### Response

>

DLEN |  ID   |  Cmd Num | Cmd Num
-----|-------|----------|--------
04h  | &#60;any&#62; | &#60;msb&#62;    | &#60;lsb&#62;

This command aborts any executing macro and returns both its ID code and the command number currently in process.
An exception is a System Macro that is executing with the `UNKILLABLE` flag set.
A normal return code indicates the ID Code of the aborted macro as well as the command number at which execution was stopped.
A return ID code of `00h` indicates that no macro was running and an ID code with `FFFFh` as the `CmdNum` that the macro was unkillable.

### Get Macro Status – 56h

#### Command

>

DID | CID | SEQ   | DLEN
----|-----|-------|-----
02h | 56h | &#60;any&#62; | 01h

#### Response

>

DLEN |  ID   | Cmd Num | Cmd Num
-----|-------|---------|--------
04h  | &#60;any&#62; | &#60;msb&#62;   | &#60;lsb&#62;

This command returns the ID code and command number of the currently executing macro.
If no macro is currently running, `00h` is returned for the ID code while the command number is left over from the last macro.

### Set Macro Parameter – 57h

#### Command

>

DID | CID | SEQ   | DLEN |  Param | Val1  | Val2
----|-----|-------|------|--------|-------|------
02h | 57h | &#60;any&#62; | 04h  | &#60;idx&#62;  | &#60;any&#62; | &#60;any&#62;

#### Response

>

    Simple Response

This command allows system globals that influence certain macro commands to be selectively altered from outside of the macro system itself.
The values of `Val1` and `Val2` depend on the parameter index.

Index | Description
------|------------
00h   | Assign System Delay 1: Val1 = MSB, Val2 = LSB
01h   | Assign System Delay 2: Val1 = MSB, Val2 = LSB
02h   | Assign System Speed 1: Val1 = speed, Val2 = 0 (ignored)
03h   | Assign System Speed 2: Val1 = speed, Val2 = 0 (ignored)
04h   | Assign System Loops: Val1 = loop count, Val2 = 0 (ignored)

Details of what these system variables change are presented in the Sphero Macro document.

### Append Macro Chunk – 58h

#### Command

>

DID | CID | SEQ   | DLEN      | MACRO Chunk
----|-----|-------|-----------|--------
02h | 58h | &#60;any&#62; | &#60;len + 1&#62; | &#60;data&#62;

#### Response

>

    Simple Response

This stores the attached macro definition into the temporary RAM buffer for later execution.
It is similar to the Save Temporary Macro call but allows you to build up longer temporary macros.

Any existing macro ID can be sent through this command and executed through the Run Macro call using ID `FFh`.
If this command is sent while a Temporary or Stream Macro is executing it will be terminated so that its storage space can be overwritten.
As with all macros, the longest chunk that can be sent is `254` bytes (thus requiring `DLEN` to be `FFh`).

You must follow this with a Run Macro command (ID `FFh`) to actually get it to go and it is best to prefix this command with an Abort call to make certain the larger buffer is completely initialized.

### Erase orbBasic Storage – 60h

#### Command

>

DID | CID | SEQ   | DLEN | Area
----|-----|-------|------|--------
02h | 60h | &#60;any&#62; | 02h  | val

#### Response

>

    Simple Response

This erases any existing program in the specified storage area.
Specify `00h` for the temporary RAM buffer or `01h` for the persistent storage area.

### Append orbBasic Fragment – 61h

#### Command

>

DID | CID | SEQ   | DLEN  |  Area | Program Code
----|-----|-------|-------|-------|-------------
02h | 61h | &#60;any&#62; | val | val | &#60;any&#62;

#### Response

>

    Simple Response

Sending an orbBasic program to Sphero involves appending blocks of text to existing ones in the specified storage area (`00h` for RAM, `01h` for persistent.
Complete lines are not required.
A line begins with a decimal line number followed by a space and is terminated with a .
See the orbBasic Interpreter document for complete information.

Possible error responses would be `ORBOTIXRSPCODEEPARAM` if an illegal storage area is specified or `ORBOTIXRSPCODEEEXEC` if the specified storage area is full.

### Execute orbBasic Program – 62h

#### Command

>

DID | CID | SEQ   | DLEN |  Area | Start Line | Start Line
----|-----|-------|------|-------|------------|------------
02h | 62h | &#60;any&#62; | 04h  | val | &#60;msb&#62;      | &#60;lsb&#62;

#### Response

>

    Simple Response

This attempts to run a program in the specified storage area beginning at the specified line number.
This command will fail if there is already an orbBasic program executing.

### Abort orbBasic Program – 63h

#### Command

>

DID | CID | SEQ   | DLEN
----|-----|-------|-----
02h | 63h | &#60;any&#62; | 01h

#### Response

>

    Simple Response

Aborts execution of any currently running orbBasic program.

### Submit Value to Input Statement – 64h

#### Command

>

DID | CID | SEQ   | DLEN |  VAL
----|-----|-------|------|--------
02h | 64h | &#60;any&#62; | 05h  | (32-bit signed val)

#### Response

>

    Simple Response

This takes the place of the typical user console in orbBasic and allows a user to answer an input request.
If there is no pending input request when this API command is sent, the supplied value is ignored without error.
Refer to the orbBasic language document for further information.

# Macro Commands

## Set Stabilization

Cmd | Flag   |  PCD
----|--------|-------
03h | &#60;bool&#62; |  &#60;any&#62;

This turns on and off the control system which actively stabilizes Sphero.
If you intend to drive around, you should make sure the system that allows you to do it is enabled.
Note that sending raw motor commands implicitly disables the stabilization system.
Flag is `00h` for OFF, `01h` for ON with control system reset and `02h` for ON without a reset.

## Set Stabilization

Cmd | Heading | Heading | PCD
----|---------|---------|-----
04h | &#60;msb&#62;   | &#60;lsb&#62;   | &#60;any&#62;

This reassigns Sphero's current heading to the supplied value.
The units are degrees so the valid range is 0 to 359.
This forms the basis for future roll commands.
For example if you assign the current heading to zero and issue a roll command along heading 90, Sphero will make a right turn.

## Set Rotation Rate

Cmd | Rate
----|-----
13h | &#60;any&#62;

Sphero's control system implements an intermediate rate limiter for the yaw axis, feeding smoothed transitions to that servo loop.
This sets the maximum increment.
As of firmware version 0.92 the formula for converting the rate parameter R to degrees/second is `40 + R/2`, which yields a smoothed range from 40 to 167 deg/s.
This only applies to Roll commands; if you use the macro command Rotate Over Time this setting is bypassed.

## Delay

Cmd | Time  | Time
----|-------|-------
0Bh | &#60;msb&#62; | &#60;lsb&#62;

This causes an immediate delay in the execution of additional macro commands while allowing the background ones to keep running.

## Set SD1, SD2

Cmd | System Delay 1 |  System Delay 1
----|----------------|---------------
01h | &#60;msb&#62;          | &#60;lsb&#62;

Cmd | System Delay 2 | System Delay 2
----|----------------|---------------
02h | &#60;msb&#62;          | &#60;lsb&#62;

Two system delay settings are provided.
Certain commands inherit these values in place of the PCD byte.

## Set SPD1, SPD2

Cmd | System Speed 1 |  System Speed 1
----|----------------|---------------
0fh | &#60;msb&#62;          | &#60;lsb&#62;

Cmd | System Speed 2 | System Speed 2
----|----------------|---------------
10h | &#60;msb&#62;          | &#60;lsb&#62;

Two system speed settings are provided.
Certain roll commands use these values in place of explicit speed values.

## Roll

Cmd | Speed | Heading | Heading | PCD
----|-------|---------|---------|------
05h | &#60;any&#62; | &#60;msb&#62;   | &#60;lsb&#62;   | &#60;any&#62;

This command gets Sphero to start rolling along the commanded speed and heading.
If the stabilization system is off, this command will do nothing.
A speed of `00h` also engages ramped down braking of roll speed.

## Roll2

Cmd | Speed | Heading | Heading | Delay | Delay
----|-------|---------|---------|-------|------
1Dh | &#60;any&#62; | &#60;msb&#62;   | &#60;lsb&#62;   | &#60;msb&#62; | &#60;lsb&#62;

This is just like the Roll command above but it accepts a 2-byte delay value.

## Set Speed

Cmd | Speed | PCD
----|-------|----
25h | &#60;any&#62; | &#60;any&#62;

This is like the Roll command but it does not effect the heading.
It is especially useful when you don't know the current heading but want to stop without experiencing a turning glitch (Speed = 0).

## Roll with SD1

Cmd | Heading | Heading | Speed
----|---------|---------|------
06h | &#60;msb&#62;   | &#60;lsb&#62;   | &#60;any&#62;

This is just like the roll command 05h but the PCD is omitted and instead derived from the SD1 value.

## Roll at SPD1 (or SPD2) with SD1

Cmd | Heading | Heading
----|---------|-------
11h | &#60;msb&#62;   | &#62;lsb&#62;

Cmd | Heading | Heading
----|---------|-------
12h | &#60;msb&#62;   | &#62;lsb&#62;

This is the ultimate in roll commands: the speed comes from one of the system speed values and the post command delay from SD1.
All you need to provide is a heading.
Use command code `11h` to select SPD1 and `12h` for SPD2.

## Send Raw Motor Commands

Cmd | Left Mode   | Left Power | Right Mode  | Right Power | PCD
----|-------------|------------|-------------|-------------|-----
0Ah | &#60;see table&#62; | &#60;any&#62;      | &#60;see table&#62; | &#60;any&#62;       | &#60;any&#62;

This allows you to take over one or both of the motor output values, instead of having the stabilization system control them.
Each motor (left and right requires a mode (see below) and a power value from `0` - `FFh`.
This command will disable stabilization if both modes aren't "ignore" so you'll need to re-enable it once you're done.

Mode  |  Description
------|---------------
00h   | Off (motor is open circuit)
01h   | Forward
02h   | Reverse
03h   | Brake (motor is shorted)
04h   | Ignore (motor mode and power is left unchanged)

## Rotate Over Time

Cmd | Angle | Angle | Time  |  Time
----|-------|-------|-------|--------
1Ah | &#60;msb&#62; | &#60;lsb&#62; | &#60;msb&#62; | &#60;lsb&#62;

This command drives the yaw control system directly to effect an angular change over time.
The angle parameter is a signed number of degrees and time is of course in milliseconds.
For example, Sphero will spin around clockwise twice in four seconds if your parameters are `720` and `4000` (the byte sequence would be `02h`, `D0h`, `0Fh`, `A0h`).
Counterclockwise in five seconds would be `-720`, `5000` (bytes `FDh`, `30h`, `13h`, `88h`).

NOTE: This command runs in the background.
Any roll commands executed before it is finished will be ignored.
In the above examples you need to be doing something for 4 and 5 seconds to give it time to finish (either other commands or simply the delay command).

## Rotate Over SD1 (or SD2)

Cmd | Angle | Angle
----|-------|-------
21h | &#60;msb&#62; | &#60;lsb&#62;

This is the same as Rotate Over Time but instead of requiring an immediate value, command code `21h` inherits this value from System Delay 1.
Likewise use code `22h` to inherit from SD2.

## Wait Until Stopped

Cmd | Time  |  Time
----|-------|-------
19h | &#60;msb&#62; | &#60;lsb&#62;

This clever command will pause execution of macros until Sphero is determined "stopped" by the stabilization system or until the provided timeout expires.
You can use this, for example, at corners where you want roll commands to make sharp turns.

## Loop Start

Cmd  | Count
-----|------
1Eh  | &#60;any&#62;

Begins a looping block, repeating the commands between this one and Loop End the specified number of times.
A count of 0 is treated as 1, neither of which do anything additional.
A second Loop Start before a Loop End replaces the previous Loop Start.
You can use `Goto` and `Gosub` from within loop blocks.

## Loop Start System

Cmd |
----|
24h |

This is a different way to begin a loop block, where the loop count is specified by API command instead of by an immediate value.

## Loop End

Cmd |
----|
1Fh |

Terminates a looping block.
If no actual loop is in process, or if the ID of the current macro doesn't match that of the Loop Start, this command is ignored.

## Comment

Cmd | Length |  Length |  Data
----|--------|---------|------
20h | &#60;msb&#62;  | &#60;lsb&#62;   | <...>

This is out of band data and no processing is performed upon it.
The macro is aborted if the Length points to a place outside of the current macro or outside of the valid data area on the Temp or Stream macro buffer.

## Set RGB LED

Cmd | Red   | Green | Blue  |  PCD
----|-------|-------|-------|--------
07h | &#60;any&#62; | &#60;any&#62; | &#60;any&#62; | &#60;any&#62;

This command drives the RGB LED to the desired values.
When macros are running RGB LED commands take precedence over all others in the system (except for battery warnings).

## Set RGB LED with SD2

Cmd | Red   | Green | Blue
----|-------|-------|------
08h | &#60;any&#62; | &#60;any&#62; | &#60;any&#62;

Just like the command above but the delay is inherited from SD2.

## Fade to LED Over Time

Cmd | Red   | Green | Blue  |  Time |  Time
----|-------|-------|-------|-------|------
14h | &#60;any&#62; | &#60;any&#62; | &#60;any&#62; | &#60;msb&#62; | &#60;lsb&#62;

This powerful command fades the RGB LED from its current value to the provided one over the time provided.
The current LED value is from the last LED macro command.
Intermediate colors are derived from the individual fractional movements of each of the red, green and blue components, not some clever movement through the color space.

NOTE: This command runs in the background so you will need to provide a suitable delay to allow it to complete.

## Set Back LED

Cmd | Value | PCD
----|-------|-------
09h | &#60;any&#62; | &#60;any&#62;

This controls the intensity of the blue "aiming" LED.
That's it.

## Goto

Cmd | Target ID
----|-----------
0Ch | &#60;any *&#62;

You can chain macros with this and the Gosub command.
The sole parameter is the Macro ID of where you want to go to.
If the target ID doesn't exist the macro aborts.
If it does then control is transferred with the current system state intact.
The macro flags of the target macro replace those currently in use.

* You cannot specify the Stream Macro ID as a destination.

## Gosub

Cmd | Target ID
----|----------
0Dh | &#60;any *&#62;

You can factor out common command sets and then call them using Gosub.
An illegal target aborts the macro and it is ignored in stream macro mode.
The call stack is currently one level deep and once it's full this command is just ignored.
There is no explicit return needed, just a macro end command.

* Like Goto, you cannot specify the Stream Macro ID as a destination.

## Branch On Collision

Cmd | Target ID
----|----------
23h | &#60;any *&#62;

This command ties the macro system to collision detection system as of FW ver 1.10 and Macro Ver 4.
When enabled, the collision detection system sets a flag which the macro executive acknowledges and then executes a Goto command to the specified Macro ID.

* Like Goto, you cannot specify the Stream Macro ID as a destination.
  But you can set the target as ID 00h which makes sure this feature is turned off – required if you're chaining between macros that alternately enable and disable this feature.

You must program and arm the collision detector separately (through an API macro or orbBasic command) before this macro command will have any effect.

## Configure Collision Detection

Cmd | Method |  Xthreshold |  Xspeed |  Ythreshold |  Yspeed |  DeadTime
----|--------|-------------|---------|-------------|---------|----------
27h | 0..1   |  &#60;any&#62;      | &#60;any&#62;   | &#60;any&#62;       | &#60;any&#62;   | &#60;any&#62;

This configures the collision detection subsystem that ties in with the Branch On Collision command.
Rather than reproduce the details here, please refer to the collision detection document.

Note that there is no PCD for this command.

## Go to Sleep Now

Cmd | Time  |  Time
----|-------|-------
0Eh | &#60;msb&#62; | &#60;lsb&#62;

This puts Sphero to sleep, able to be awaken from a double-shake.
The time parameter is optional and is the number of milliseconds for him to automatically reawaken.
If set to zero, he goes to sleep forever.
If set to `FFFFh` the actual time is inherited from the API command (`DID 00h`, `CID 22h`).
Which just proves that the API command calls a system macro which implements this.

## End

Cmd |
----|
00h |

This signals the end of a normal macro.
If there is an address in the gosub stack then execution will resume after the Gosub that called it.
If a stream macro is running this command is ignored.

The macro flags contain some options that can be executed at the end of a macro.

## Stream End

Cmd |
----|
1Bh |

This signals the end of a stream macro.
Most of the same rules as above apply, but if you use this command out of a stream macro then processing will abort.

## Emit Marker

Cmd  | Marker
-----|---------
15h  | &#60;val&#62;

This emits an asynchronous message to the client with the supplied marker value.
Marker value zero is reserved for the end of macro option, so don t emit it unless you want to confuse yourself.
The format of the async message payload is below:

Marker |  Macro ID |  Command # | Command #
-------|-----------|------------|----------
&#60;val&#62;  | &#60;lsb&#62;     | &#60;msb&#62;      | &#60;lsb&#62;

Async ID code `06h` is reserved for macro notifications, the Marker field comes from the command and the last two bytes are the command number of this marker within the current macro.
You can read more about async messages in the Sphero API document.

# Macro Codes Quick Reference

Macro Commands (defined in macro.h)

val | name
----|------
00h | Macro End
01h | Set SD1
02h | Set SD2
03h | Set Stabilization
04h | Set Heading
05h | Roll
06h | Roll with SD1
07h | Set RGB LED
08h | Set RGB LED with SD2
09h | Set Front LED
0Ah | Set Raw Motor Values
0Bh | Delay
0Ch | Goto
0Dh | Gosub
0Eh | Go To Sleep
0Fh | Set SPD1
10h | Set SPD2
11h | Roll at SPD1 with SD1
12h | Roll at SPD2 with SD1
13h | Set Rotation Rate
14h | Fade to RGB
15h | Emit Marker
19h | Wait Until Stopped
1Ah | Rotate Over Time
1Bh | Stream End
1Ch | Reserved
1Dh | Roll2
1Eh | Loop Start
1Fh | Loop End
20h | Comment
21h | Rotate Over SD1
22h | Rotate Over SD2
23h | Branch On Collision
24h |  Loop Start System
25h | Set Speed
27h | Configure Collision Detection

# OrbBasic

## Variables

### User Variables

50 unique user controlled variables are supported, half directly named and the other half indexed:

* Direct variables are named A..Y (or as a..y if you prefer)
* Z is in the index variable and when accessed, the Yth index is dereferenced, implying Z(Y).
  There are no zero indices in Basic so the valid range of Y when accessing Z is 1..25

All variables are 32-bit signed integers yielding a range of `-2,147,483,647` to `+2,147,483,647`.

If you want to use fractions then you will need to learn how to use fixed point math.

There is currently no support for strings, other than their temporary assembly in print statements.

When a program is run, all variables, timers, pending delays and flags are initialized to zero.
You can assume a clean slate.

### System Variables

A number of special system variables are implemented allowing enhanced functionality and control.

#### timerA, timerB, timerC

* Access: read, write
* Size: 16-bit positive value
* Available: ver 0.9 (enh in 1.07)
￼
These are built-in timers that count either up or down by themselves, once per millisecond.
They are both readable and writeable.
If you assign a value greater than zero to the timer, it will count down and stop at zero.
If you assign it a zero it will begin counting up and wrap at 65,535.

The following demonstrates a 125ms delay between print statements (though it would be smarter to use the delay keyword):

    10 print "Before"
    20 timerA = 125
    30 if timerA > 0 then goto 30
    40 print "After"

You can also use them to time events:

    10 goroll 0,255,2
    20 delay 1000
    30 goroll 0,255,0
    40 timerB = 0 ' Start counting upwards 50 if speed > 0 then goto 50
    60 print timerB " milliseconds to stop"

#### ctrl

* Access: read, write
* Size: 1-bit value
* Available: ver 0.9
￼
When read, this variable returns 0 if the control system if off and 1 if it is on.
When assigned, 0 turns off the control system and any other value turns it on.

Example:

    10 ctrl = 1 'change to 0 and re-run 20 print "Control system is",ctrl,;
    30 if ctrl = 1 then ?"(on)" else ?"(off)"

#### speed

* Access: read
* Size: 8-bit positive value
* Available: ver 0.9
￼
Returns the approximate speed of the robot in real time.
This value is the filtered average speed of both motors so there is some delay between what you see and what this returns.
It is read-only and ranges from `0..255`.

#### yaw

* Access: read
* Size: 16-bit positive value
* Available: ver 0.9

Returns the current heading of the robot as reported by the IMU.
It is read-only and ranges from `0..359` in the usual convention of how Sphero manages headings.
This program turns the back LED on and off in 45 degree sectors; download it and spin Sphero like a top.

    10 ctrl = 0
    20 LEDC 8
    30 X = 255 * ((yaw/45) & 1)
    40 backLED X
    50 goto 30

#### pitch

* Access: read
* Size: 16-bit positive value
* Available: ver 0.9

Returns the current pitch angle of the robot as reported by the IMU.
It is read-only and ranges from `-90` when the front of Sphero is pointing straight down to `+90` when it is pointing straight up.

#### roll

* Access: read
* Size: 16-bit positive value
* Available: ver 0.9

Returns the current roll angle of the robot as reported by the IMU.
It is read-only and ranges from `-180` as Sphero rolls to the left and `+180` when it rolls completely over to the right.

Here is a good program to display the relationship of all three of the above variables.

    10 ctrl = 0
    20 print pitch, roll, yaw
    30 delay 100
    40 goto 20

#### accelX, accelY, accelZ

* Access: read
* Size: 16-bit signed value
* Available: ver 0.9

Returns the current filtered reading from the accelerometer.
The range is `-32768` to `+32767` for full-scale, which defaults to ±8Gs.
The convention is that X is the pitch axis, Y is roll and Z is yaw.

#### gyroX, gyroY, gyroZ

* Access: read
* Size: 16-bit signed value
* Available: ver 0.9

Returns the current filtered reading from the rate gyro in units of 0.1 degrees/second.
The range is `-20000` to `+20000` for full-scale.
The convention is that X is the pitch axis, Y is roll and Z is yaw.

#### Vbatt

* Access: read
* Size: 16-bit signed value
* Available: ver 0.9

Returns the current battery voltage in 100ths of a volt, so a return value of 756 indicates 7.56V.

#### Sbatt

* Access: read
* Size: 16-bit signed value
* Available: ver 0.9

Returns the current state of the power system, as follows:

*￼1￼-￼Battery is charging
*￼2 -￼Battery is OK
*￼3 -￼Battery voltage low
*￼4 -￼Battery voltage critical

#### cmdroll

* Access: read
* Size: Boolean
* Available: ver 0.9

Returns a value of 1 when a new roll command has been sent by a Bluetooth client, essentially making this a "fresh data" flag.
Reading this system variable automatically sets it to zero.

Roll commands usually don't arrive faster than 10Hz so there isn’t much use in checking this flag faster than every 100ms.
When this variable reads as one, the other two system variables `spdval` and `hdgval` will have fresh values in them.

Older versions of orbBasic prior to 1.6 named this cmdval.

#### spdval

* Access: read
* Size: 8-bit positive value
* Available: ver 0.9

Returns the last commanded speed portion of a roll command sent from the smartphone client, ranging from 0 (stop) to 255 (full speed).

#### hdgval

* Access: read
* Size: 16-bit positive value
* Available: ver 0.9

Returns the last command heading portion of a roll command sent from the smartphone client ranging from 0 to 359 degrees.

    10 if cmdroll > 0 then ?"New heading, speed:" hdgval, spdval 20 delay 100
    30 goto 10

#### cmdrgb

* Access: read
* Size: Boolean
* Available: ver 1.6

Returns a value of 1 when a new RGB LED command has been sent by a Bluetooth client, essentially making this a "fresh data" flag.
Reading this system variable automatically sets it to zero.
This command allows you to intercept RGB commands and either use the supplied parameters for other purposes or reprocess and re-express it through the RGB command.

The same message timing applies here like it does to cmdval.

#### redval, grnval, bluval
* Access: read
* Size: 8-bit positive value
* Available: ver 1.6

Returns the last commanded red, green and blue values of an RGB command sent from the smartphone client.
Each value ranges from 0 to 255.

#### isconn

* Access: read
* Size: Boolean
* Available: ver 0.9

This variable provides real time access to the current state of a connected smartphone over Bluetooth.

    10 if isconn > 0 then LEDC 1 else LEDC 2
    20 delay 100
    30 goto 10

#### dshake

* Access: read
* Size: Boolean
* Available: ver 0.9

Returns a value of 1 when double-shake event has been detected.
Reading this system variable automatically sets it to zero.

    10 if dshake > 0 then LEDC 1 else LEDC 2
    20 delay 100
    30 goto 10

#### accelone

* Access: read
* Size: 16-bit positive value
* Available: ver 0.9

Returns the effective acceleration vector that Sphero is experiencing.
The IMU computes this as the vector sum of the X, Y and Z axis components so it is expressed here multiplied by 1,000.
A perfectly calibrated Sphero will return 1,000 when still, 0 when in free fall and up to 8,000 when being shaken.
But since few things are perfect, you will need to apply some soft ranges to convert this hard value to a state.

    10 if accelone < 200 then ? "Free fall! (" accelone ")"
    20 if accelone > 900 then ?"Still"
    30 delay 200
    40 goto 10

#### xpos, ypos
￼
* Access: read
* Size: 16-bit signed value
* Available: ver 0.9

Returns the current x and y position of Sphero as determined by the internal locator.
The approximate scale is in centimeters.
Use the function locate to assign these values.

Run this program in the background while using the Drive App to move Sphero in and out of the 40cm circle.

    10 locate 0,0
    20 delay 100
    30 R = sqrt (xpos*xpos + ypos*ypos)
    40 if R < 20 then LEDC 1 else LEDC 2
    50 goto 20

#### Qzero, Qone, Qtwo, Qthree

* Access: read
* Size: 16-bit signed value
* Available: ver 0.9

Sphero's IMU is quaternion based and these variables access the elements of that vector.
Each value ranges from `-10,000` to `+10,000` which corresponds to the normalized internal value of -1.0 to +1.0.
Reading these system variables faster than 400 Hz is not useful and will return duplicate data.

## Expressions

Simple mathematical operators are supported up to a reasonable expression depth.

Operator |  Description              | Example               | Precedence
-------- | -------------             | --------------------- | ----
(        | Begin a new subexpression | 2 + 3 * 5 yields 17   | high
)        | End a subexpression       | (2 + 3) * 5 yields 25 | high
*        | *Integer multiplication   | 2 * 3 yields 6        | medium
/        | Integer division          | 10 / 3 yields 3       | medium
%        | Integer remainder         | 10 % 3 yields 1       | medium
{        | Binary left shift         | 3 { 2 yields 12       | medium
}        | Binary right shift        | 20 } 2 yields 5       | medium
+        | +Addition                 | 1 + 2 yields 3        | low
-        | -Subtraction              | 1 - 2 yields -1       | low
&        | Bitwise AND               | 45 & 85 yields 5      | low
/|       | Bitwise OR                | 45                    | 85 yields 125 | low



The goal was to allow an expression anywhere a variable or a numeric literal was permitted, offering the most flexibility.
But this may not always be the case (testing will reveal!)

## Functions

Few of these are supported but if demanded, I will expand the list.
I started with the most useful ones.

### sqrt x

Returns the square root of any expression that follows.
A negative number halts on error.

Since orbBasic only supports integer math you need to understand the following identity to really use this for smaller numbers:

`√a * b = √a * √b` where a is the number you want to take the square root of and b is a multiplier to increase the precision.
Since the maximum positive value of any expression in orbBasic is around 2.1 million, choosing b as 1002 or 10,000 is a good choice.
Consider the following example where you want to take the square root of 42:

Let `a = 42` and `b = 10000`, so sqrt 420000 yields 648.
Since `√10000 == 100`, this answer is 100 times too large.
Which lines up with the real answer of √42 which is about 6.48

    10 for X = 10 to 100 step 10
    20 Q = sqrt X*10000
    30 print "sqrt(" X ") = " Q/100 "." Q % 100
    40 next X

#### outputs

    sqrt(10) = 3.16
    sqrt(20) = 4.47
    sqrt(30) = 5.47
    sqrt(40) = 6.32
    sqrt(50) = 7.7
    sqrt(60) = 7.74
    sqrt(70) = 8.36
    sqrt(80) = 8.94
    sqrt(90) = 9.48
    sqrt(100) = 10.0

Using a multiplier of 10,000 to extract two extra decimal places of precision in your answer will allow you to take the square root of up to about 214,000 or 462^2
￼
* Access: read
* Size: 32-bit positive value
* Available: ver 0.9
￼
### rnd x

* Access: read
* Size: 32-bit positive value
* Available: ver 0.9

This returns a pseudo random number between 1 and the value provided, which can be either a literal number or an expression.
Use the random statement to reseed the number generator. For example:

    10 for X = 1 to 50
    20 RGB rnd 255,rnd 255,rnd 255
    30 delay rnd 250
    40 next X

### abs x

* Access: read
* Size: 32-bit positive value
* Available: ver 0.9

This returns the absolute value of an expression, which is always positive.

## Statements

These are a mix of the traditional ones supported in Basic plus extensions that allow interaction with Sphero's other systems.
Case is regarded so goto is not the same as GOTO.
I chose statements in lower case so that a) programs don't look like shouting in email and b) capitalizing user variables helps them stand out.

This is the classic if...then...else construct.
In some variants of Basic the then keyword was implied and therefor optional; here it is required.
The else clause is optional.
You can use and/or to glue two relations together if necessary.

    <conditional> := if <relation> { and, or } <relation> then <statement> { else <statement> }

    <relation> := <expression> <operator> <expression>

Currently the only operators supported are:
￼
* = - Equality
* ! - Inequality
* < - Less than
* > - Greater than

Examples:

* `if A>5 then A=5`
* `if A*B = B*C then goto 60`
* `if timer1 = 0 then print "Out of time!"`
* `if A ! B then print "Not equal"`
* `if A>1 and A<10 then ?"In range" else return`

It isn't possible to accommodate every possible nested statement in the true/false clause processing but the following are supported: print, data, RGB, LEDC, read, rstr, goto, gosub, on X goto/gosub, return, delay, backLED, tron, troffandgoroll.

### Looping

    <loop> := for <loop variable> = <start value> to <end value> { step <step value> }
    { statements }
    next <loop variable>

The standard method for looping over a number range in orbBasic is the for..next loop.
The loop variable can be any user variable.
The start and end values can be literal numbers or expressions – even involving other variables (both user and system).
The step value is optional and if excluded, 1 is assumed.
Of course this won't work if your ending value is lower than your starting one.

There is a maximum nesting level and when exceeded an appropriate error is generated.
It is acceptable to change the loop variable in the middle of the loop, for example, if you wish to terminate before completing the original numbers of loops.
(If you choose to do this just remember that the loop terminates when the loop variable falls outside the end value.k

Examples:

* `for X = 1 to 5 step 2 ... next x o yields 3 loops with X = 1,3,5`
* `for Y = 10 to 3 step -3 ... next x o yields 3 loops with Y = 10,7,4`

### Branching

    <branch> := goto <line number> | gosub <line number>

This is pretty straight forward.
In both cases the target line number must be a numeric literal (i.e. it cannot be an expression).
Not too surprisingly the way to return from a subroutine is to use the return statement.
There is a maximum nesting level and when exceeded, an appropriate error is generated.

### Indexed Branching

    <indexed branch> := on <expression> goto, gosub <line number list >

This statement implements an N-way transfer of control to one of N line numbers.
The expression must evaluate to a number between 1 and N, when there are N line numbers in the list.

Example:

* `on X*Y goto 100,200,300,400`

### print

orbBasic wouldn't be too useful if you couldn't export information from your running program back to the smartphone client.
The print statement takes care of this.

The final length of each print statement is limited to 32 bytes.
( If you exceed this, a warning message is printed instead.)
The print statement is a sequence of string literals and expressions.
String literals get copied exactly as written while expressions are evaluated and their numeric value is expressed.

If the $ symbol precedes an expression, it is evaluated but instead of the number itself a single character, the ASCII value of the expression, is emitted instead.

Commas between print elements insert a space in the output.
If the final character of the print string is terminated with a semicolon then no linefeed is appended to the message sent back to the client.

The output of print is enqueued for transmit so it is possible to fill up that queue faster than it can be emptied over the Bluetooth link.
If this happens, data is not lost but program execution is paused until the print statement can complete normally.

The single ? character is shorthand for print and can help save program space.

Examples:

    10 print "The answer is: " A*44

    10 ? "Line " L ": the speed is: " speed;

    10 for X = 65 to 67
    20 print X " as a number, " $X " as a char"
    30 next X

outputs

    65 as a number, A as a char
    66 as a number, B as a char
    67 as a number, C as a char

###input

* Access: see below
* Size: see below
* Available: ver 1.5

Just like print, the absence of an input statement would severely limit the interaction of orbBasic with the user (or a smartphone app).
Unlike most forms of BASIC, our input statement takes two forms because Sphero is a multitasking robot and running an orbBasic program is just ONE of the things it can do.

The first form is the traditional blocking type you've seen before:

    10 print "Enter a number:"
    20 input X
    30 print "You entered " X;
    40 print " and half of that is " X/2

In this form the input statement will wait forever until the API command to deliver a number is sent to Sphero.
(Note that the API command is `DID 02h`, `CID 64h`) The only other way out is by sending an abort command to the orbBasic interpreter.

The second form helps manage the infinite wait:

    10 print "Enter a number:"
    20 input X,3000,0
    30 if X = 0 then ?"No answer!" else ?"You entered " X

The two extra parameters added to the input statement are respectively how long to wait for an answer (in milliseconds) and what value to assign to the variable if that time limit expires.
In this example the program will wait for 3 seconds and if no API command delivers a new value for X, it is assigned a zero.

### delay x

* Access: parameterized
* Size: 16-bit positive value
* Available: ver 0.9

This is a built-in delay statement that doesn't use the three timers.
It takes one parameter, an expression resolving to the number of milliseconds in the delay.
Here is an interesting use of this statement:

    10 for X = 1 to 74
    20 RGB X,X,X
    30 delay X*4
    40 RGB 0,0,0
    50 delay X{1  'same as X*2
    60 next X

### end

Programs terminate in the following ways: the final line number is executed, an error occurs, an abort command is sent or an end statement is encountered.
This statement allows you to, well, end in the middle of a program.

### RGB r,g,b

* Access: parameterized
* Size: 8-bit positive values
* Available: ver 0.9

This commands the RGB LED with the three provided parameters, each of which is evaluated as an expression.
Values range from 0,0,0 (off) to 255,255,255 (white).
Note that this statement is in all-caps since its still an abbreviation.
The RGB LED is updated every 1ms so changing its value faster than this with orbBasic is ineffective.

When orbBasic is running, the RGB LED assumes the color of any programmatic RGB commands unless a macro is also running – where it will take the value of the macro commands.
You can override this precedence with an orbBasic flag, however.

### LEDC x

* Access: parameterized
* Size: 8-bit positive value
* Available: ver 0.9

This is like the RGB command but sets the LED to one of eight predefined colors, without requiring you to know the three component values.


Value  | Color
------ | ------
0      | &#60;off&#62;
1      | red
2      | green
3      | blue
4      | orange
5      | purple
6      | white
7      | yellow
8      | &#60;off&#62;

Example:

    10 for X = 1 to 8
    20 LEDC X
    30 delay 100
    40 next X

Advanced example:

    10 for X = 1 to 8
    20 LEDC (X % 2) * 5  'modulo 2 operation returns 0 or 1
    30 delay 100
    40 next X

### backLED x

* Access: parameterized
* Size: 8-bit positive value
* Available: ver 0.9

This commands the aiming LED with an intensity from 0..255.
For example:

    10 X = 1
    20 backLED X
    30 delay 50
    40 X = X { 1
    50 if X < 256 then goto 20
    60 goto 10

### random

This reseeds the pseudorandom number generator based on the jitter of the sensor network.
Since this operation is performed prior to the execution of an orbBasic program, you shouldn’t normally need to call this – but hey, you can if you wish.

### goroll h,s,g

* Access: parameterized
* Size: three values, see below
* Available: ver 1.2

This executes a roll command to the control system with the provided values of heading and speed.
Heading is over the usual range of 0..359 and speed over 0..255.
The last parameter, a flag for "go" tells the control system whether to force fast rotate to on (g=2), attempt to drive normally (g = 1) or perform a controlled stop (g=0).

If you haven't turned on exclusive drive ability from orbBasic via the basflg statement, roll commands from macros or the Bluetooth client can overwrite this and cause unexpected behavior.

### heading h

* Access: parameterized
* Size: 16-bit positive value
* Available: ver 1.2

This assigns the current yaw heading to the provided value, in the range of 0..359.

### raw Lmode,Lspeed,Rmode,Rspeed

* Access: parameterized
* Size: four values, see below
* Available: ver 1.3

This allows you to take over one or both of the motor output values, instead of having the stabilization system control them.
Each motor (left and right) requires a mode (see below) and a power value from 0- 255.
This command will disable stabilization if both modes aren't "ignore" so you'll need to re-enable it once you're done.

*￼0 - Off (motor is open circuit)
* 1 -￼Forward
* 2 -￼Reverse
* 3 -￼Brake (motor is shorted)
* 4 - Ignore (motor mode and power is left unchanged)

### locate x,y

* Access: parameterized
* Size: two values, see below
* Available: ver 0.9

This assigns the XY position of the internal locator to the provided values.

    10 locate 0,0
    20 goroll 0,100
    30 delay 3000
    40 goroll 0,0
    50 delay 1000
    60 print xpos,ypos

### basflg x

* Access: write only
* Size: 16-bit positive value
* Available: ver 0.9

This statement accepts a decimal value to assign state flags that are effective during execution of the program.
If you want to set multiple flags then add up their values.

At the end of program execution these effect of these flags is removed.
I will be adding more as needs arise.

Value  | Symbolic Name    | ￼Description
------ | ---------------  | ------------
1      | BF_EXCLUSIVE_DRV | Gives the program exclusive control of driving and excludes those commands from the Bluetooth client.
2      | ￼BF_STEALTH      | Execution of an orbBasic program will NOT reset the inactivity timeout
4      | ￼BF_KEEP_LED     | Normally macros own the RGB LED even when orbBasic is executing; setting this flag will keep orbBasic as the owner.
8      | BF_ALLOW_SOD     | Stop on Disconnect action is normally disabled when an orbBasic program is running. Setting this re-enables that action to take place.

### data d1{,d2,d3...}

* Access: parameterized
* Size: 32-bit signed values
* Available: ver 0.9

Defines a set of constant data values that can be sequentially read via the read statement.
You can specify more than the maximum number of values (currently set to 25) and they will be ignored.
When this statement is encountered, previously stored values are overwritten so it is possible to re-assign the constants during program execution.

### rstr

Forces the next read statement to start back at the beginning of the current constant data set.

### read X{,Y...}

* Access: parameterized
* Size: One or more 32-bit signed values
* Available: ver 1.2

Reads the next value(s) from the current data set into the specified variable(s).
If you attempt to read past the end, you will get an error.

Example:

    10 data 10,20,30
    20 read A,B
    30 rstr
    40 read C
    50 print A,B,C
    60 data 100
    70 read A
    80 print A,B,C

displays:

    10 20 10
    100 20 10

### tron

This turns line number tracing on to aid in debugging your program.
Line numbers are emitted through the standard print channel prior to the execution of that line.
This statement takes effect at the next line number.

For example:

    10 print "at line 10"
    20 tron
    30 ? "at line 30"
    40 ? "and now, 40"

displays:

    at line 10
    <30>at line 30
    <40>and now, 40

### troff

This turns line number tracing off.

### reset

This resets the orbBasic environment and restarts the program freshly from the initial starting line number.
All gosub and for/next frames are cleared and all variables set to zero.

### sleep duration, macro, line_number

* Access: parameterized
* Size: see below
* Available: ver 0.9

This command immediately puts Sphero to sleep with a few modifiers:

*￼duration -￼If non-zero, the number of seconds in the future to automatically reawaken from 1 to 65535.
  Zero causes Sphero to sleep forever (well, until double- shaken awake).
*￼macro - The macro ID to run upon reawakening; zero for none.
  This ID cannot be the temporary or stream macro ID (254 or 255).
* line_number -￼The line number of an orbBasic program to run on reawakening; zero for none.
  Note that this line number relates to programs stored in the Flash area.

Since macros and orbBasic programs can run simultaneously you can specify both actions to occur.

#### macrun x

* Access: parameterized
* Size: 8-bit unsigned value
* Available: ver 0.9

Attempts to run the macro number specified.
If a macro is currently running, this will have no effect (and will not cause an error).

### mackill

Kills any currently executing macro, with the exception of system macros that have the "unkillable" flag bit set.

###macstat

* Access: read only
* Size: 32-bit unisgned value
* Available: ver 0.9

This returns information about the currently executing macro.
In the upper 16 bits is the ID of the currently executing macro, or zero if none is running.
In the lower 16 bits is the command number currently being executed.

    10 X = macstat
    20 M = X } 16
    30 if M = 0 then ?"None running" else ?"Macro:" M, "Cmd:" X & 65535


