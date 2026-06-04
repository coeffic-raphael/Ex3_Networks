# RUDP Congestion Lab

An educational C networking project that implements a simple Reliable UDP protocol and compares it with TCP Reno and TCP Cubic under different network conditions.

## Overview

This repository contains four command-line programs:

- `RUDP_Sender` and `RUDP_Receiver`: a custom reliability layer built on top of UDP.
- `TCP_Sender` and `TCP_Receiver`: TCP transfer programs that can request either Reno or Cubic congestion control.

The programs generate and transfer random data, measure transfer time and bandwidth, and print per-run statistics. The repository also includes packet captures for RUDP, TCP Reno, and TCP Cubic experiments.

## Features

- UDP socket wrapper with a custom packet format.
- Stop-and-wait style reliable transfer using sequence numbers and ACKs.
- Basic checksum validation.
- SYN/SYN-ACK style connection setup for RUDP.
- FIN/ACK style connection termination for RUDP.
- TCP congestion-control selection for Reno and Cubic.
- Transfer timing and bandwidth reporting.
- Wireshark-compatible `.pcap` captures for protocol analysis.

## Repository Structure

```text
.
├── RUDP_API.c          # Reliable UDP implementation
├── RUDP_API.h          # RUDP packet structure and API
├── RUDP_Sender.c       # RUDP client
├── RUDP_Receiver.c     # RUDP server
├── TCP_Sender.c        # TCP client with Reno/Cubic selection
├── TCP_Receiver.c      # TCP server with Reno/Cubic selection
├── Makefile
├── rudp_pcap/          # RUDP packet captures
├── tcp_reno_pcap/      # TCP Reno packet captures
└── tcp_cubic_pcap/     # TCP Cubic packet captures
```

## Requirements

- `gcc`
- `make`
- Linux is recommended for the TCP Reno/Cubic programs.

The TCP programs use `TCP_CONGESTION`, which is a Linux socket option. On macOS, the default `make` target builds only the RUDP programs. Use Linux if you want to build and run the TCP Reno/Cubic comparison programs.

## Build

```bash
make
```

On Linux, `make` builds both the TCP and RUDP programs. On macOS, `make` builds the RUDP programs only because TCP Reno/Cubic selection depends on the Linux-specific `TCP_CONGESTION` socket option.

You can also build each group explicitly:

```bash
make rudp
make tcp
```

To remove compiled files:

```bash
make clean
```

## Usage

Run the receiver first, then start the sender in another terminal.

### RUDP

Terminal 1:

```bash
./RUDP_Receiver -p 8080
```

Terminal 2:

```bash
./RUDP_Sender -ip 127.0.0.1 -p 8080
```

The sender generates 2 MB of random data and sends it to the receiver. After each transfer, the sender asks whether to send the data again.

### TCP Reno

Terminal 1:

```bash
./TCP_Receiver -p 8080 -algo reno
```

Terminal 2:

```bash
./TCP_Sender -ip 127.0.0.1 -p 8080 -algo reno
```

### TCP Cubic

Terminal 1:

```bash
./TCP_Receiver -p 8080 -algo cubic
```

Terminal 2:

```bash
./TCP_Sender -ip 127.0.0.1 -p 8080 -algo cubic
```

## Packet Captures

The capture folders contain `.pcap` files that can be opened with Wireshark:

- `rudp_pcap/`: captures from the custom RUDP implementation.
- `tcp_reno_pcap/`: captures from TCP Reno tests.
- `tcp_cubic_pcap/`: captures from TCP Cubic tests.

These traces are useful for inspecting retransmissions, ACK behavior, connection setup, and the differences between the tested transport approaches.

## Implementation Notes

The RUDP packet format contains:

- control flags: `SYN`, `ACK`, `DATA`, `FIN`
- a sequence number
- a checksum
- payload size
- a fixed payload buffer of 5000 bytes

The RUDP sender splits the generated data into 5000-byte chunks and waits for an ACK before sending the next chunk. If an ACK is not received before the timeout, the packet is retransmitted.

## Known Limitations

This project is designed as a networking exercise, not as a production transport protocol.

- The RUDP checksum is intentionally simple and only checks a small part of the payload.
- RUDP uses a stop-and-wait transfer model, so throughput is limited compared with sliding-window protocols.
- The implementation handles a single sender/receiver flow.
- TCP congestion-control selection depends on Linux support for `TCP_CONGESTION`.
- The `.pcap` files are useful for analysis, but they make the repository heavier than a source-only project.

## License

This project is released under the MIT License. See [LICENSE](LICENSE).
