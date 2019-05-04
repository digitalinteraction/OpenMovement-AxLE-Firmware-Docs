# Open Movement AxLE Firmware Documentation

Open Movement AxLE firmware documentation.

## Device Overview

Firmware is written for the *nRF51* and *nRF52*.

Devices have the name `axLE-Band` or `I5-Om`, and later versions of the firmware suffix this with `-######`, where `######` represents the least significant 24-bits of the device's Bluetooth address in hexadecimal.


## BLE Services

### `device_information` service (0x180A)

* `serial_number_string` characteristic (0x2A25)
* `firmware_revision_string` characteristic (0x2A26)
* `hardware_revision_string` characteristic (0x2A27)
<!-- * `software_revision_string` characteristic (0x2A28) -->
* `manufacturer_name_string` characteristic (0x2A29)

Note: `serial_number_string` is blocked by *WebBluetooth*.

### `battery_service` service (0x180F)

* `battery_level` characteristic (0x2A19)

### UART service (6E400001-B5A3-F393-E0A9-E50E24DCCA9E)

* TX Characteristic (6E400002-B5A3-F393-E0A9-E50E24DCCA9E)
* RX Characteristic (6E400003-B5A3-F393-E0A9-E50E24DCCA9E)


## Sending and receiving data

Register for changes to the *RX Characteristic* to receive responses over UART.  Buffer responses and split lines when an ASCII `LF` (value 10, line-feed) is recieved.  Remove any trailing `CR` (value 13, carriage-return) before the line-feed.

Writes are to the *TX Characteristic*.

Note that the UART is not fully "streamed" and the byte position in the transmit packet is significant to the firmware.


## Commands: Overview

The commands' initial letters are case-insensitive.

Commands, unless otherwise specified, require an authenticated connection.  If the connection is not authenticated, the response will be:

```
!
```

The outbound numeric parameters are all hex-encoded bytewise little-endian representations of the integers.  The 4-bit nibble indexes are as follows:

* `INT16HEX`: `1032`
* `INT32HEX`: `10325476`


## Commands: Status

#### Device address (unauthenticated)

Obtain the device address:

> `#`

If the command is supported, return will contain the device Bluetooth address:

```
#:<address>
```

Where `<address>` is in the format `##:##:##:##:##:##`, where `##` represents two hexadecimal digits of the address.

Otherwise, if the command is not supported, the response will be:

```
?
```


#### Query cycles

To query the device erase, reset and battery cycles:

> `E?`

The response will be:

```
B:<battery-cycles>
R:<reset-cycles>
E:<erase-cycles>
```


#### Battery level

To poll the battery level:

> `B`

The response will be:

```
B:<battery-percentage>%
```


#### Accelerometer sample

To poll the accelerometer values:

> `A`

The response will be:

```
A:<Ax>,<Ay>,<Az>,<interrupt-status>
```

Where `<Ax>`/`<Ay>`/`<Az>` are signed decimal raw values from the current accelerometer configuraiton, and `<interrupt-status>` is a two-byte hexadecimal value showing the state of the accelerometer interrupt 1 source (used for orientation).


## Commands: Device control

#### Unlock (unauthenticated)

Unlock (authenticate) the connection:

> `U<password>`

Where `<password>` is in the format `######` (defaults to be the master password).

This command first *unauthenticates* the connection, then attempts to *authenticate* with the specified password.

If the password is correct, the connection is now authenticated and the response will be:

```
Authenticated
```

Otherwise (if the password is incorrect) the connection will remain unauthenticated and the response will be:

```
!
```


#### Password set

Set the authentication password:

> `P<new-password>`

Where `<new-password>` is in the format `######`, and must be 6 alphanumeric characters (in the set `[0-9A-Za-z]`).

If the user is authenticated and the password is correctly formatted and updated, the response will be:

```
P:<new-password>
```

If the user is not authenticated, or there are not exactly 6 characters in the new password, the response will be:

```
!
```

If any of the 6 characters are not alphanumeric (in the set `[0-9A-Za-z]`), the response will be:

```
Error?
```

If the password cannot be updated, the response will be:

```
Error
```


#### Erase epoch data and password

To erase epoch data over an authenticated connection:

> `E`

If the connection is authenticated, the command will be successful: the password will be reset to the master password, the number of erase cycles will be incremented, any logging will be stopped and resumed after 5 seconds, and the response will be:

```
Erase data
```


#### Factory reset device

To erase *all* device data (including battery, reset and erase cycles) over an authenticated connection:

> `E!`

If the connection is authenticated, the command will be successful.

Alternatively, only if the connection in unauthenticated, you can erase *all* device data using the master password:

> `E<master-password>`

Where `<master-password>` is in the format `######`.  If the connection is unauthenticated and the password is correct, the command will be successful.  Note, if the connection is authenticated, this variation of the command will behave as "erase epoch data".

If the command is successful: the password will be reset to the master password, the cycles are reset, any logging will be stopped and resumed after 5 seconds, and the response will be:

```
Erase all
```

Otherwise, the response will be:

```
!
```


## Commands: Time

To set the device time:

> `T<time:INT32HEX>`

This clears the device cueing (?).  The response will be the same as reading the device time.

Note, this is not a recommended way of using the device: it is probably simpler (and allows for multiple synchronizing devices) to allow the time to freely tick without overwriting it, and store the synchronized equivalent time in the receiver.

To read the device time (optionally suffix with `?`):

> `T`

The response will be (in decimal):

```
T:<time>
```

Where `<time>` is an unsigned 32-bit decimal number.


## Commands: Hibernate

To set the *hibernate*/*stop logging* time:

> `H<time:INT32HEX>`

The response will be the same as querying the hibernate time.

To query the hibernate time (optionally suffix with `?`):

> `H`

The response will be (in decimal):

```
H:<time>
```

Where `<time>` is an unsigned 32-bit decimal number.





## Commands: hardware outputs

#### Outputs off

To turn off LED and motor outpus:

> `0`

(Zero), alternatively the letter:

> `O`

If the connection is authenticated, outputs will stop and the response will be:

```
OFF
```


#### Motor on

To set the motor output on (2 seconds?):

> `1`

Alternatively, to set the pulsed motor output on (1 second?):

> `M`

If the connection is authenticated, motor output will be enabled as requested, and the response will be:

```
MOT
```


#### LED2 on

To set the LED2 (green) output on:

> `2`

If the connection is authenticated, LED2 output will be enabled for 1 second, and the response will be:

```
LED2
```


#### LED3 on

To set the LED3 (blue) output on:

> `3`

If the connection is authenticated, LED3 output will be enabled for 1 second, and the response will be:

```
LED3
```




---
L: Request lower power slower connection
F: Request faster higher power connection
V or V?: Query connection interval. Response for 48ms is V:48
VXXXX: Set connection interval milliseconds

C or C?: Toggle or query cueing on for an hour or off
CXXXX: Set new cuing period to X* and turn on

NXXXX or N?: Set epoch period span with NXX*. 
  Default is 60 seconds. - Not fully tested

Y: Hardware control, empty battery using outputs - Not for normal use

WXXXX: Write download block number, responds as with 'Q'uery
Q or Q?: Query command. 
  Response: time, T:
  active block,  B:
  active samples,  N:
  active epoch,  E:
  block count,  C:
  download block  I:
  
SXXXX: Synchronise epoch timing by offset X*. 
  Response: S:xxxx
  
R: Read command. 
  Response: Active block in ascii hex - A block is 510 bytes + Checksum (2 bytes)
  Data structure:
  typedef union Epoch_sample_tag  {
    uint16_t w[4];
    uint8_t b[8];
    struct {
      int8_t batt;
      int8_t temp;
      int8_t accel;
      int8_t steps;
      int8_t epoch[4];
    } part;
  } Epoch_sample_t;

  // Information tag in each block
  typedef struct EpochBlockInfo_tag {
    uint16_t block_number;
    uint16_t data_length;
    uint32_t time_stamp;  
  } EpochBlockInfo_t;

  // Epoch data block type
  typedef struct Epoch_block_tag {
    // Tag at start of each block - 8 bytes
    EpochBlockInfo_t info;  
    // Block data format
    uint16_t blockFormat;
    // Extended data area, implementation specific
    uint8_t meta_data[20]; 
    // 480 bytes of sequential epoch entries (1 hour/ 60 mins)
    Epoch_sample_t epoch_data[EPOCH_BLOCK_DATA_COUNT];
    // Checksum, ECC or CRC etc.
    uint16_t check;
  } Epoch_block_t;

I: IMU stream on/off
  Response: Streaming accel data - Raw ascii-hex data stream in format

          // Add streaming packet data header for time, batt, temp, count, etc... 
          timeStamp,   // 8 chars
          battRaw,  // 4 chars
          tempRaw,  // 4 chars
          accel_samples[25],  // 25 * 12 chars
          terminator,  // "\r\n", 2 chars
          
D: Debug
  Response: Debug values stream output - Experimental only
  
G: Goal function setup  - Not fully tested
  GOXXXX: Set goal period offset to X*
  GPXXXX: Set goal period value to X*
  GGXXXX: Set the goal threshold value to X*
  Response: O:X P:Y G:Z value output
  
X: Device reset - Used to reset to bootloader for firmware update
  X!: Reset now
  XD: Stop app, start bootloader
  XNNNN: Pause logger for N* seconds
  
* Value setting entered in ascii hex format. 
i.e. 266 = 256 + 10 = 0A01
```

