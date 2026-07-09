# dspic33ak-clock-hal

Want to run it on hardware first?
Start with `dspic33ak-hal-starter`.

`dspic33ak-clock-hal` is a small, readable PLL and CLKGEN HAL for Microchip
dsPIC33AK devices. It accepts logical clock requests at the public API, keeps
device/register adaptation below the generic core, and leaves board or
application policy outside the HAL.

## Overview

This HAL programs PLL and CLKGEN blocks without making board-level choices for
the application. A board integration layer decides the boot source, frequency
plan, peripheral ownership, and bring-up ordering. The HAL validates the
requested PLL / CLKGEN mechanics and performs the hardware programming sequence.

It is not a turnkey board clock framework.

## Status

Current publication state:

```text
private staging branch
validated source import
not merged to standalone main
```

Validated target:

```text
Device:   dsPIC33AK512MPS512
Compiler: XC-DSC
DFP:      Microchip dsPIC33AK-MP DFP 1.3.185,
          or a compatible version with matching register definitions
```

The code has future-facing status values for device-family expansion, but this
repository currently publishes the dsPIC33AK512MPS512-validated implementation
only.

## What This HAL Does

The HAL provides:

- Logical clock source enums.
- PLL1 and PLL2 programming.
- Exact-target integer PLL solving.
- PLL input validation.
- Interim VCO validation.
- PLL output validation.
- Divider representability checks.
- CLKGEN programming for CLKGEN1, CLKGEN5, CLKGEN6, CLKGEN8, CLKGEN9, CLKGEN10, and CLKGEN12.
- Hardware switch sequencing.
- Ready polling.
- Timeout reporting.
- Device-specific NOSC encoding isolation.
- Raw SFR isolation below the generic core.

## What This HAL Does Not Do

The generic HAL does not own:

- Board pin routing.
- PPS policy.
- REFI pin selection.
- Boot source policy.
- Application frequency policy.
- UART clock policy.
- PWM clock policy.
- CAN FCAN policy.
- Audio sample-rate policy.
- Peripheral initialization ordering.
- Fallback, retry, delay, or hidden workaround logic.

For example, the HAL can program CLKGEN10. The application decides whether
CLKGEN10 is used for CAN FD at 20 MHz.

## Design Policy

Keep policy above the HAL:

```text
board / application policy
    ->
generic PLL / CLKGEN requests
    ->
device/register adaptation
    ->
dsPIC33AK SFRs
```

The public API uses logical PLL, CLKGEN, and source identifiers. XC-DSC bitfield
names and device-specific NOSC encodings are kept below the public interface.

## Layering

```text
src/dspic33ak_clock.h
  Public API.

src/dspic33ak_clock.c
  Generic validation, exact PLL solver, and orchestration.

src/dspic33ak_clock_device.h
src/dspic33ak_clock_device.c
  Internal logical-source to dsPIC33AK NOSC encoding adaptation.

src/dspic33ak_clock_reg.h
src/dspic33ak_clock_reg.c
  Internal raw PLL / CLKGEN SFR access, switch sequencing, ready polling,
  and timeout handling.
```

Applications normally include only:

```c
#include "dspic33ak_clock.h"
```

The device and register headers are internal adaptation interfaces.

## Files

```text
README.md
LICENSE

src/
  dspic33ak_clock.h
  dspic33ak_clock.c
  dspic33ak_clock_device.h
  dspic33ak_clock_device.c
  dspic33ak_clock_reg.h
  dspic33ak_clock_reg.c

docs/
  standalone_bootstrap.md
```

## Basic PLL Usage

```c
#include "dspic33ak_clock.h"

const dspic33ak_clock_pll_config_t pll1 = {
    .source = DSPIC33AK_CLOCK_SOURCE_FRC,
    .input_hz = 8000000UL,
    .target_hz = 200000000UL,
};

uint32_t actual_hz = 0u;

dspic33ak_clock_status_t status =
    dspic33ak_clock_pll_configure(
        DSPIC33AK_CLOCK_PLL_1,
        &pll1,
        &actual_hz);
```

`actual_hz` is optional and is written only after successful hardware
programming.

## Basic CLKGEN Usage

```c
const dspic33ak_clock_clkgen_config_t clkgen = {
    .source = DSPIC33AK_CLOCK_SOURCE_PLL1,
    .divide_by = 1u,
};

status = dspic33ak_clock_clkgen_configure(
    DSPIC33AK_CLOCK_CLKGEN_8,
    &clkgen);
```

The application owns the meaning of the selected CLKGEN. For example, a board
integration layer may route CLKGEN8 to UART1, but that UART policy is not part
of the generic Clock HAL.

## Return Status Behavior

Current public status values:

```text
DSPIC33AK_CLOCK_OK
DSPIC33AK_CLOCK_ERR_INVALID_ARG
DSPIC33AK_CLOCK_ERR_NOT_PRESENT
DSPIC33AK_CLOCK_ERR_NOT_SUPPORTED
DSPIC33AK_CLOCK_ERR_UNREPRESENTABLE
DSPIC33AK_CLOCK_ERR_TIMEOUT
```

`DSPIC33AK_CLOCK_ERR_INVALID_ARG` covers null pointers, unsupported logical
sources for a selected block, zero CLKGEN dividers, and unknown enum values.

`DSPIC33AK_CLOCK_ERR_UNREPRESENTABLE` means the requested exact PLL output
cannot be represented within the current device limits and integer divider
ranges.

`DSPIC33AK_CLOCK_ERR_TIMEOUT` means a hardware switch or ready bit did not
complete within the register-layer polling budget.

`DSPIC33AK_CLOCK_ERR_NOT_PRESENT` and
`DSPIC33AK_CLOCK_ERR_NOT_SUPPORTED` are currently reserved for future
device-family capability handling and are not returned by the current
dsPIC33AK512MPS512 implementation paths.

## Validation Provenance

This standalone repository was bootstrapped from the hardware-validated Starter
integration copy:

```text
dspic33ak-hal-starter
7215105c02108d570f88d27837bbe58a87e6cd9d
```

That Starter copy is byte-identical to the original Perseus HAL source lineage:

```text
perseus_512_96K
89c2b88f44f0830ece99dcc9649b5e16382f3421
```

The relationship is:

```text
Perseus 89c2b88
    generic HAL origin
        |
        | byte-identical
        v
Starter 7215105
    hardware-validated integration copy
        |
        | direct import
        v
Standalone Clock HAL branch
```

### Perseus Validation

Validated Perseus policy:

```text
FRC 8 MHz
  -> PLL1 200 MHz

REFI1 12.288 MHz
  -> PLL2 798.72 MHz
  -> CLKGEN5 divide-by-1
```

The normal Perseus audio and application paths passed their existing build and
hardware verification.

### Starter Validation

Validated Starter policy:

```text
FRC 8 MHz
  -> PLL1 200 MHz

PLL1
  -> CLKGEN1/5/6/8/9 divide-by-1

PLL1
  -> CLKGEN10 divide-by-10
  -> CAN FD FCAN 20 MHz
```

The Starter hardware smoke covered boot, UART, timers, SPI flash, I2C scan, CAN,
PWM/ADC startup, and TDM runtime. The external I2C2/I2C3 loopback data-path
check was waived because the loopback board was unavailable; it is not recorded
as PASS.

## Known Limitation

Early cold-start FRC -> PLL2 behavior is not a validated path.

The current validated PLL2 application policy uses REFI1. The root cause of the
earlier FRC -> PLL2 cold-start failure remains unresolved. This HAL does not add
fallback, retry, startup delay, or hidden workaround logic.

## Relationship To dspic33ak-hal-starter

`dspic33ak-hal-starter` is the worked hardware-integration example. It owns the
board policy around this HAL:

```text
FRC boot policy
application SYSCLK
which CLKGEN feeds which peripheral
CAN FCAN policy
UART console policy
demo bring-up ordering
```

Use this standalone repository when you want the reusable PLL / CLKGEN HAL
source without Starter application code.

## License

MIT No Attribution License (`MIT-0`).

See [`LICENSE`](LICENSE).
