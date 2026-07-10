# Rationale: Why LoRa Simulation Didn't Exist

This document explains why Renode peripheral models for LoRa devices didn't exist before this project, and why we built them anyway.

## The Gap

As of 2026, you could simulate almost any embedded system in Renode:
- ARM Cortex-M, RISC-V, x86
- WiFi, BLE, Ethernet networking
- Flash, I2C, SPI, UART peripherals
- Sensors, displays, motor controllers

But if your design included a LoRa radio — one of the most popular LPWAN technologies — you were out of luck. No SX126x model. No SX127x model. No STM32WL SubGHz. Nothing.

## Why Nobody Built This

### 1. Tiny Market Intersection

The people who need this occupy a very small intersection:

```
┌─────────────────────────────────────────────────┐
│                                                 │
│         Embedded Developers                     │
│    ┌─────────────────────────────────┐         │
│    │                                 │         │
│    │      Renode Users               │         │
│    │   ┌─────────────────────┐      │         │
│    │   │                     │      │         │
│    │   │   LoRa Developers   │      │         │
│    │   │  ┌─────────────┐   │      │         │
│    │   │  │ CI-Motivated│   │      │         │
│    │   │  │   (~50?)    │   │      │         │
│    │   │  └─────────────┘   │      │         │
│    │   └─────────────────────┘      │         │
│    └─────────────────────────────────┘         │
│                                                 │
└─────────────────────────────────────────────────┘
```

Most embedded developers don't use Renode (QEMU, hardware, or nothing).
Most Renode users don't work with LoRa (WiFi/BLE are more common).
Most LoRa developers don't need simulation (hardware is cheap).
The intersection that wants CI for LoRa firmware in Renode? Maybe 50 people worldwide.

### 2. Hardware Is Cheap

A T-Echo costs $35. A RAK4631 costs $25. A Heltec board costs $18.

When hardware is cheaper than an hour of engineering time, most teams just buy a drawer full of boards and test on real hardware. The economic case for simulation is weak unless you're:
- Running CI at scale (hundreds of test runs per day)
- Testing scenarios that would destroy hardware (power faults, clock glitches)
- Working in an environment where hardware is impractical (submarine, spacecraft)

### 3. High Skill Barrier

Building a Renode peripheral requires:
- C# proficiency (Renode's extension language)
- Understanding of the Renode peripheral API
- Deep knowledge of the target device (SX126x datasheet is 130 pages)
- A working Zephyr/firmware toolchain to test against

The Venn diagram of "knows C#" ∩ "knows Renode API" ∩ "knows LoRa radio internals" ∩ "has time to write open source" is approximately zero people.

### 4. No Commercial Incentive

Semtech sells silicon. ST sells silicon. They have no incentive to help you avoid buying their silicon. Their business model is the opposite of simulation.

LoRaWAN network operators (TTN, Helium, etc.) care about deployed devices, not development tooling.

The only companies that might benefit — embedded consultancies — are too small to fund infrastructure projects.

### 5. Community Fragmentation

The LoRa ecosystem is fragmented:
- **Meshtastic:** Custom protocol, mobile app focus
- **RadioLib:** Arduino library, hobbyist focus
- **LoRaWAN stacks:** LMiC, LoRaMac-node, proprietary
- **Zephyr:** Has LoRa drivers but no simulation

These communities don't share infrastructure. Each reinvents testing approaches independently.

### 6. Chicken and Egg

To test a board definition, you need peripherals.
To test a peripheral, you need a board definition.

Nobody wants to build half of a solution that doesn't work alone.

## Why We Built It Anyway

The [LICHEN project](https://github.com/MarkAtwood/project-LICHEN) needed CI for LoRa mesh firmware. We had two choices:

1. **Hardware CI:** Buy boards, flash them, run tests, deal with USB/serial flakiness, replace boards when they die, maintain a physical test lab.

2. **Simulation CI:** Build the missing Renode peripherals, run tests in VMs, scale horizontally, never replace hardware.

Option 2 had higher upfront cost but lower ongoing cost. More importantly, it let us test scenarios that are hard to reproduce on hardware:
- Precise timing of RX/TX transitions
- Multi-node mesh behavior (would need N physical boards)
- Edge cases in the radio state machine
- Deterministic "random" behavior for debugging

Once we'd built it for ourselves, extracting it as a standalone project was minimal additional work. And maybe those ~50 people who need this will find it useful.

## Design Decisions

### Why AGPL-3.0?

Simulation infrastructure should be a commons. If a company uses this to build commercial LoRa products, they should contribute their improvements back. The AGPL ensures that.

MIT/Apache would allow proprietary forks that never return value to the community. Given how few people maintain this kind of infrastructure, that's not sustainable.

### Why Monorepo?

We considered separate repos for SX126x, STM32WL, and board definitions. But:
- The audience is tiny — don't over-engineer
- Boards depend on peripherals — keeping them together avoids version skew
- One CI matrix is easier than three
- If SX126x takes off independently, forking it out is easy

### Why Not Upstream to Renode?

We may eventually. Renode accepts peripheral contributions. But:
- Upstreaming takes time (reviews, iterations, bureaucracy)
- We wanted something working now
- A standalone repo lets us iterate faster
- External repo is easier for Meshtastic/RadioLib communities to discover

If this proves useful, upstreaming individual peripherals is straightforward.

## Limitations We Accept

### Timing Is Approximate

Real LoRa radios have precise timing based on spreading factor, bandwidth, coding rate, and preamble length. We don't model this accurately — packets transmit "instantly" in simulation time.

This is fine for protocol testing but wrong for timing-critical applications (TDMA, precise ranging). Accurate timing would require significant complexity.

### No RF Modeling

We don't model:
- Path loss, fading, interference
- Antenna patterns
- Frequency accuracy, drift
- Adjacent channel rejection

Packets either arrive or they don't. This is fine for protocol logic but useless for RF performance testing.

### Limited Error Injection

We don't model:
- CRC errors (partial packet corruption)
- Preamble detection failures
- SPI bus errors
- Power supply glitches

These could be added, but we haven't needed them yet.

## Future Work

If there's interest:
- **SX127x model:** Older but still common (RFM95, etc.)
- **Multi-node simulation coordinator:** External process that routes packets between Renode instances with configurable topology/loss
- **LoRaWAN testing:** NS/AS simulation for end-to-end testing
- **Timing accuracy:** Model actual ToA for TDMA applications

Contributions welcome.
