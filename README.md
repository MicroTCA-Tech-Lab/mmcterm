# mmcterm

Terminal for the custom "serial over IPMB" protocol used by DMMC-STAMP.

## Usage

```
$ mmcterm [-h] [-v] [-c CHANNEL] [-l] [-d] [-i] mch_addr mmc_addr

DESY MMC Serial over IPMB console

positional arguments:
  mch_addr              IP address or hostname of MCH
  mmc_addr              IPMB-L address of MMC

optional arguments:
  -h, --help            show this help message and exit
  -v, --version         show program's version number and exit
  -c CHANNEL, --channel CHANNEL
                        console channel
  -l, --list            list available channels
  -d, --debug           pyipmi debug mode
  -i, --ipmitool        make pyipmi use ipmitool instead of native rmcp
```

## Channels

In the current DMMC-STAMP implementation, there is only channel 0 (MMC console) available. In future implementations, depending on the AMC board, additional serial channels can be addressed, for example UARTs of one or more payload FPGAs.

## Example

Open a console on AMC board at IPMB address 0x7a connected to the MCH `mskmchhvf1.tech.lab`:
```bash
mmcterm mskmchhvf1.tech.lab 0x7a
```

## Protocol description

"Serial over IPMB" is a lightweight "serial data forwarding" protocol using custom IPMI messages.

### Get channel info

NetFn 0x30, Command 0xf0

Request:

| Byte    | Contents        |
|---------|-----------------|
| 0       | Channel Number  |

Response:

| Byte    | Contents        |
|---------|-----------------|
| 0..n    | Channel Name    |

### Start/stop SOI session

NetFn 0x30, Command 0xf1

Request:

| Byte    | Contents            |
|---------|---------------------|
| 0       | Channel Number      |
| 1       | 1 = start, 0 = stop |

### Poll/exchange data

NetFn 0x30, Command 0xf2

Request:

| Byte    | Contents                       |
|---------|--------------------------------|
| 0..n    | Send data from terminal to MMC |

Response:

| Byte    | Contents                          |
|---------|-----------------------------------|
| 0..n    | Receive data from MMC to terminal |
