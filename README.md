# mmcterm

Terminal for the custom "serial over IPMB" protocol used by DMMC-STAMP.

## Installation

```
pip3 install mmcterm
```

## Usage

```
mmcterm [-h] [-v] [-c CHANNEL] [-t INTERVAL] [-l] [-d] [-i] [-m MAX_PKT_SIZE] mch_addr mmc_addr

DESY MMC Serial over IPMB console

positional arguments:
  mch_addr              IP address or hostname of MCH
  mmc_addr              IPMB-L address of MMC or "AMCn" (n=1..12)

optional arguments:
  -h, --help            show this help message and exit
  -v, --version         show program's version number and exit
  -c CHANNEL, --channel CHANNEL
                        console channel (default 0)
  -t INTERVAL, --interval INTERVAL
                        polling interval in ms (default 10)
  -l, --list            list available channels
  -d, --debug           pyipmi debug mode
  -i, --ipmitool        make pyipmi use ipmitool instead of native rmcp
  -m MAX_PKT_SIZE, --max-pkt-size MAX_PKT_SIZE
                        max IPMB packet size to use (Higher numbers give better performance, but can break depending on MCH model)
```

## Channels

The DMMC-STAMP always provides channel 0 for the MMC console. This is also the default channel, if the `-c` argument is not given to `mmcterm`. Depending on the board implementation, there can be up to 2 additional channels for UARTs of payload FPGAs. The presence and names of those channels can be queried with the `-l` argument.

Note that only one channel can be opened to a MMC at the same time.

## Example

Open a console on AMC board at IPMB address 0x7a connected to the MCH `mskmchhvf1.tech.lab`:
```bash
mmcterm mskmchhvf1.tech.lab 0x7a
```

## Max packet size

Without the `-m` option, `mmcterm` uses the standard maximum IPMB packet size of 32 bytes. Due to the limitations of the IPMI protocol, this can quickly become a bottleneck, esp. when used with a SoC running Linux. To mitigate this bottleneck, bigger packet sizes can be set with `-m`. For NAT MCHs, `-m 100` was found to be working, even though it is more than 3 times the standard packet size.

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

| Byte    | Contents                   |
|---------|----------------------------|
| 0       | Channel Number             |
| 1       | 1 = start, 0 = stop        |
| 2       | Max packet size (optional) |

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
