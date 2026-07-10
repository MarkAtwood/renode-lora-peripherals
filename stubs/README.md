# Peripheral Stubs

Supporting peripheral stubs for board simulation.

## GPS (gps.cs)

NMEA sentence generator over UART. Emits GGA and RMC sentences at 1Hz.

Configurable position via Renode properties:
```
gps: Sensors.GPS @ uart1
    latitude: 37.2350
    longitude: -115.8111
    altitude: 1360.0
```

~218 lines.

## BLE (ble.cs)

Minimal nRF52840 BLE stub for boot and basic operations.

~511 lines.

## Source

Will be extracted from `lichen/boards/renode/peripherals/` in the LICHEN project.
