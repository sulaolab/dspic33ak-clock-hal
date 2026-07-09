# Standalone Bootstrap

## §1 Standalone Clock HAL Bootstrap and Three-Way Source Identity

### Status

Standalone Clock HAL bootstrap:
  COMPLETE

Repository visibility:
  PRIVATE

Standalone mainline integration:
  NOT PERFORMED BY DESIGN

Three-way HAL source identity:
  PASS

Ready for coordinated three-way review:
  YES

### Repository Creation State

| Item | Value |
|---|---|
| Repository | `sulaolab/dspic33ak-clock-hal` |
| Visibility | `PRIVATE` |
| Default branch | `main` |
| Local folder | `C:\00_storage\_Git_Work\vscode-home\dspic33ak-clock-hal` |
| Initial `main` HEAD | `43b9f20381a3676c0dd86efa125d38882e5e23e3` |
| `origin/main` HEAD | `43b9f20381a3676c0dd86efa125d38882e5e23e3` |
| Initial main contents | Minimal bootstrap README only |

The HAL implementation was not added to `main`.

### Development Branch

| Item | Value |
|---|---|
| Branch | `exp/clock-hal-standalone-bootstrap` |
| Branch base | Initial `main` HEAD `43b9f20381a3676c0dd86efa125d38882e5e23e3` |
| Branch policy | Branch-only bootstrap; do not merge to `main` in this task |

The branch was pushed immediately after creation and used for all standalone HAL
work.

### Direct Starter Source Provenance

The direct import source is the hardware-validated Starter integration copy:

| Item | Value |
|---|---|
| Repository | `sulaolab/dspic33ak-hal-starter` |
| Commit | `7215105c02108d570f88d27837bbe58a87e6cd9d` |
| Source path | `src/hal_clock/` |
| Reason | This is the integration-tested Clock HAL copy from Starter. |

### Perseus Origin Provenance

The original generic HAL lineage is:

| Item | Value |
|---|---|
| Repository | `sulaolab/perseus_512_96K` |
| Branch | `exp/perseus-clock-hal-starter-readiness` |
| Commit | `89c2b88f44f0830ece99dcc9649b5e16382f3421` |
| Source path | `src/hal_clock/` |
| Reason | This is the prepared generic Clock HAL source that Starter imported. |

The six Starter source files at `7215105` are byte-identical to the six Perseus
source files at `89c2b88`.

### Import Method

The standalone repository intentionally flattens the HAL layout:

```text
Starter src/hal_clock/*
    ->
Standalone src/*
```

The final import used Git blob extraction from the exact Starter commit and
wrote the blob byte streams directly to `src/`. This avoided worktree
line-ending conversion from the Starter repository's `text eol=crlf` attributes.

No imported HAL source file was manually edited.

### Imported Files and Blob Identity

| Standalone path | Starter source path | Expected / actual blob |
|---|---|---|
| `src/dspic33ak_clock.h` | `src/hal_clock/dspic33ak_clock.h` | `7ae5e89a7adf8f2e3a1bc5512f6db7af41d3bce4` |
| `src/dspic33ak_clock.c` | `src/hal_clock/dspic33ak_clock.c` | `4f0640f5f819c75cc76002b26f88d706c59a7eaf` |
| `src/dspic33ak_clock_device.h` | `src/hal_clock/dspic33ak_clock_device.h` | `05b7f375c630d3f6799cd90c475138572fabe1b4` |
| `src/dspic33ak_clock_device.c` | `src/hal_clock/dspic33ak_clock_device.c` | `a230a806eceb51a0bf956d20352b74f27f1c660a` |
| `src/dspic33ak_clock_reg.h` | `src/hal_clock/dspic33ak_clock_reg.h` | `e9636729324a84c9e842d522a83508a9a282ba88` |
| `src/dspic33ak_clock_reg.c` | `src/hal_clock/dspic33ak_clock_reg.c` | `ab5a3d42476d2e22cee318ac611ffacf856de2d8` |

Verification result:

```text
Perseus blob == Starter blob == Standalone blob
6/6 PASS
```

### Standalone Directory Layout

Final branch layout:

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

No `src/hal_clock/` directory was created in the standalone repository.

### README Decisions

The README was rewritten for a standalone HAL audience. It documents:

```text
Overview
Status
What this HAL does
What this HAL does not do
Design policy
Layering
Files
Supported / validated target
Basic PLL usage
Basic CLKGEN usage
Return status behavior
Validation provenance
Known limitation
Relationship to dspic33ak-hal-starter
```

The README explicitly keeps board policy outside the HAL and points hardware
bring-up readers to `dspic33ak-hal-starter`.

The README records the I2C2/I2C3 external loopback data-path check as waived
because the loopback board was unavailable; it is not described as PASS.

### License Decision

`LICENSE` uses the same license text as the existing SulaoLab standalone
dsPIC33AK HAL repositories:

```text
MIT No Attribution License
Copyright (c) 2026 SulaoLab
```

No Microchip source, DFP files, MPLAB X project files, or generated build files
were added.

### Publication-Context Source Review

Each imported source file was reviewed from the perspective of standalone
publication:

```text
src/dspic33ak_clock.h
src/dspic33ak_clock.c
src/dspic33ak_clock_device.h
src/dspic33ak_clock_device.c
src/dspic33ak_clock_reg.h
src/dspic33ak_clock_reg.c
```

Review focus:

```text
Perseus-only wording
Starter-only wording
development-history comments
claims that are too broad
unsupported device-family claims
wrong DFP references
stale ownership comments
application-repository-only comments
```

Result:

```text
PASS
```

No intrinsic HAL issue was found. The six imported source files remain unchanged.

### Intrinsic HAL Issues Found

None.

No synchronized three-repository correction is required for this bootstrap.

### Verification

| Check | Result |
|---|---|
| Repository visibility | PRIVATE |
| Default branch | `main` |
| Initial `main` preserved | PASS |
| Development branch created | PASS |
| Six HAL files present | PASS |
| Six standalone blobs match expected Starter blobs | PASS |
| Six standalone blobs match Perseus `89c2b88` blobs | PASS |
| Starter policy source copied | No |
| Perseus application source copied | No |
| Board / PPS source copied | No |
| MPLAB X project copied | No |
| README API names exist in public header | PASS |
| README CLKGEN list matches public enum | PASS |
| README clock facts match validated records | PASS |
| LICENSE present | PASS |
| `git diff --check` | PASS |
| Hardware use | Not used |

Hardware was not used because the standalone source remains byte-identical to the
exact HAL copy already validated in Starter and previously validated in Perseus.

### Branch and Push State

The final branch is intended to be pushed as:

```text
origin/exp/clock-hal-standalone-bootstrap
```

Expected final state after push:

```text
working tree clean
local/remote no difference
main unchanged
standalone HAL implementation not merged to main
```

### Mainline State

Standalone `main` remains the minimal repository bootstrap and does not contain
the HAL implementation.

No Perseus mainline, Starter mainline, or standalone mainline merge was performed.

### Three-Way Identity Gate

```text
Perseus
  89c2b88f44f0830ece99dcc9649b5e16382f3421

Starter
  7215105c02108d570f88d27837bbe58a87e6cd9d

Standalone
  exp/clock-hal-standalone-bootstrap final branch HEAD
```

For the six HAL files:

```text
Perseus blob
  ==
Starter blob
  ==
Standalone blob
```

Result:

```text
6/6 PASS
```

### Recommended Next One Task

Open the private branch review for `exp/clock-hal-standalone-bootstrap` as the
standalone side of the coordinated Clock HAL three-way review. Do not merge the
standalone branch to `main` until the Perseus readiness branch and Starter
integration branch are reviewed together.
