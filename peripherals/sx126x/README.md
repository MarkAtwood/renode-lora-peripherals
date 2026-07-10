# SX126x Peripheral

Semtech SX1261/SX1262/SX1268 LoRa transceiver model for Renode.

**Status:** Pending extraction from [LICHEN project](https://github.com/MarkAtwood/project-LICHEN).

## Features (planned)

- SPI command interface (opcodes 0x80-0x9F)
- TX: SetTx, WriteBuffer, packet transmission
- RX: SetRx, ReadBuffer, DIO1 interrupt on RxDone
- IRQ status register and masking
- FIFO buffer (256 bytes)
- Loopback mode for single-node testing
- Bridge mode for multi-node simulation

## Usage

```
sx1262: Radio.SX1262 @ spi1
    -> gpio0@20  // DIO1

spi1:
    sx1262 -> gpio0@24  // CS
```

## Source

Will be extracted from `lichen/boards/renode/peripherals/SX1262.cs` (~490 lines).
