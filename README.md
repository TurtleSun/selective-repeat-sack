# selective-repeat-sack

A simulation of the **Selective Repeat ARQ protocol with SACK (Selective Acknowledgment) extensions**, implemented in C as part of graduate coursework in Advanced Computer Networks at Boston University.

## Overview

Implements reliable unidirectional data transfer between two endpoints (A → B) over an unreliable network layer that simulates packet loss and corruption. Built on top of a network emulator originally authored by J.F. Kurose.

## Features

- Sliding window protocol with configurable window size
- Selective retransmission using SACK — only missing packets are retransmitted, not the entire window
- Circular buffer management with sequence number wraparound
- RTT measurement excluding retransmitted packets (Karn's algorithm)
- Configurable loss probability, corruption probability, and retransmission timeout
- Simulation statistics: average RTT, communication time, loss ratio, corruption ratio

## How to Run

**Requirements:** GCC, Linux/macOS
```bash
gcc sr_sack.c -o simulation
./simulation
```

Follow the prompts to set simulation parameters:

| Parameter | Description |
|---|---|
| Number of messages | Total packets to simulate |
| Loss probability | 0.0 = no loss, 1.0 = all lost |
| Corruption probability | Probability of bit-level corruption |
| Window size | Sliding window size |
| Retransmission timeout | Timer duration before retransmit |

## Key Design Decisions

- **SACK piggybacking** — B includes a SACK array on every ACK, telling A which out-of-order packets it already has. A skips retransmitting those.
- **Karn's algorithm** — RTT measurements exclude retransmitted packets to avoid skewing the average.
- **Single shared timer** — simplified from canonical SR; a production implementation would use per-packet timers.
- **Additive checksum** — sufficient for simulation purposes; CRC-32 would be appropriate in production.

## Protocol Flow
```
A (Sender)                          B (Receiver)
    |                                    |
    |------- packet 0 ------------------>|  delivered to Layer 5
    |------- packet 1 ------------------>|  delivered to Layer 5
    |------- packet 2 -------X           |  lost
    |------- packet 3 ------------------>|  buffered (out of order)
    |                                    |
    |<------ ACK 1, SACK=[3] ------------|
    |                                    |
    |------- packet 2 (retransmit) ----->|  delivered → 3 flushed from buffer
    |                                    |
    |<------ ACK 3 ----------------------|
```

## Statistics Output

At the end of each simulation run:

- Original packets transmitted
- Retransmissions
- Packets delivered to Layer 5
- ACKs sent
- Average RTT
- Average communication time
- Loss and corruption ratios
