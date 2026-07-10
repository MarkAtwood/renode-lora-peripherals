# STM32WL SubGHz Peripheral

STM32WL integrated SubGHz radio model for Renode.

**Status:** Pending extraction from [LICHEN project](https://github.com/MarkAtwood/project-LICHEN).

## Features (planned)

- SUBGHZ_SPI internal peripheral
- TX/RX operations
- IRQ to NVIC
- Supporting PWR and RCC stubs

## Source

Will be extracted from:
- `lichen/boards/renode/nucleo_wl55jc/peripherals/LichenSubGHz.cs` (~341 lines)
- `lichen/boards/renode/nucleo_wl55jc/peripherals/STM32WL_PWR.cs` (~99 lines)
- `lichen/boards/renode/nucleo_wl55jc/peripherals/STM32WL_RCC.cs` (~117 lines)
