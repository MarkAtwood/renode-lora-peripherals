# renode-lora-peripherals

Renode peripheral models and board definitions for LoRa embedded development.

**Status:** Early development. Extracted from the [LICHEN](https://github.com/MarkAtwood/project-LICHEN) project.

## What This Is

A collection of [Renode](https://renode.io/) peripheral models and board definitions that enable simulation of LoRa-based embedded systems without physical hardware. Supports:

- **Semtech SX126x** (SX1261, SX1262, SX1268) — the most common LoRa transceiver
- **STM32WL SubGHz** — ST's integrated LoRa SoC
- **Meshtastic-compatible boards** — T-Echo, RAK4631, T-Deck, Heltec WiFi LoRa32 V3

## Why This Exists

LoRa firmware development has a simulation gap. See [RATIONALE.md](RATIONALE.md) for the full story, but briefly:

- Renode is excellent for embedded simulation, but has no LoRa peripheral models
- LoRa hardware is cheap ($20-35), so most developers just buy boards
- The intersection of "Renode expertise" + "LoRa expertise" + "CI motivation" is tiny
- We needed CI for [LICHEN](https://github.com/MarkAtwood/project-LICHEN) without burning hardware

## Repository Structure

```
renode-lora-peripherals/
├── peripherals/
│   ├── sx126x/              # SX1261/62/68 SPI peripheral model
│   └── stm32wl-subghz/      # STM32WL integrated SubGHz radio
├── boards/
│   ├── t-echo/              # LilyGO T-Echo (nRF52840 + SX1262)
│   ├── rak4631/             # RAKwireless RAK4631 (nRF52840 + SX1262)
│   ├── t-deck/              # LilyGO T-Deck (ESP32-S3 + SX1262)
│   ├── heltec-wifi-lora32-v3/  # Heltec WiFi LoRa32 V3 (ESP32-S3)
│   └── nucleo-wl55jc/       # ST Nucleo-WL55JC (STM32WL)
├── stubs/
│   ├── gps.cs               # NMEA GPS over UART
│   └── ble.cs               # nRF52840 BLE stub
├── examples/
│   ├── zephyr-lora-tx/      # Minimal Zephyr LoRa transmit
│   └── meshtastic-boot/     # Boot Meshtastic firmware
├── LICENSE                  # AGPL-3.0
├── README.md
└── RATIONALE.md
```

## Quick Start

### Prerequisites

- [Renode](https://renode.io/) 1.14+ (tested with 1.16)
- For examples: [Zephyr RTOS](https://zephyrproject.org/) toolchain

### Using the SX126x Peripheral

Add to your `.repl` file:

```
sx1262: Radio.SX1262 @ spi1
    -> gpio0@20  // DIO1 interrupt

spi1:
    sx1262 -> gpio0@24  // CS pin
```

### Using Board Definitions

```bash
# In Renode monitor
include @boards/t-echo/t_echo.repl
sysbus LoadELF @path/to/your/firmware.elf
start
```

## Peripheral Details

### SX126x (SX1261/SX1262/SX1268)

The SX126x model implements:

- SPI command interface (opcodes 0x80-0x9F)
- TX path: SetTx, WriteBuffer, packet transmission
- RX path: SetRx, ReadBuffer, DIO1 interrupt on RxDone
- IRQ status register and masking
- Basic FIFO buffer

**Modes:**
- **Loopback:** Single-node testing (TX packets loop back to RX)
- **Bridge:** Multi-node simulation via TCP socket to external coordinator

**Limitations:**
- No CAD (Channel Activity Detection)
- No frequency/bandwidth/SF validation (accepts any config)
- Timing is approximate (no precise ToA calculation)

### STM32WL SubGHz

Models the integrated SubGHz radio in STM32WL series:

- SUBGHZ_SPI internal peripheral
- Basic TX/RX operations
- IRQ to NVIC

Includes supporting stubs for STM32WL_PWR and STM32WL_RCC.

## Supported Boards

| Board | SoC | Radio | Status |
|-------|-----|-------|--------|
| LilyGO T-Echo | nRF52840 | SX1262 | Working |
| RAK4631 | nRF52840 | SX1262 | GPIO wiring in progress |
| LilyGO T-Deck | ESP32-S3 | SX1262 | Experimental |
| Heltec WiFi LoRa32 V3 | ESP32-S3 | SX1262 | Experimental |
| Nucleo-WL55JC | STM32WL55 | Integrated | Working |

## Contributing

Contributions welcome! Particularly:

- Additional board definitions
- SX127x peripheral model (older LoRa chips)
- Improved timing accuracy
- LoRaWAN stack testing examples

## License

AGPL-3.0. See [LICENSE](LICENSE).

If you use this in a commercial product, the AGPL requires you to share your modifications. This is intentional — LoRa simulation infrastructure should be a commons.

## Acknowledgments

Extracted from the [LICHEN project](https://github.com/MarkAtwood/project-LICHEN), a low-power encrypted mesh networking stack.

Built with [Renode](https://renode.io/) by [Antmicro](https://antmicro.com/).
