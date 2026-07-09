# max17320-nvm-tool

A small, safety-gated STM32 HAL I2C driver for provisioning a MAX17320
(2S-4S ModelGauge m5 fuel gauge / protector) battery pack: presence check,
register read/write, shadow-RAM config backup/write/verify, remaining-NVM-
write-cycle query, and a datasheet-exact nonvolatile memory commit.

Built to safely push a known-good NV register configuration (captured from
a Maxim/Analog Devices EVKIT GUI export) onto a second board that only has
I2C access (no USB/GUI), without guessing register addresses, commands, or
timings, and without risking the part's limited (7 total) NVM write budget.

## Files

- `max17320.h` / `max17320.c` -- the driver.
- `max17320_config.h` -- target NV register table + `MAX17320_RSENSE_MOHM`
  parametric macro (change one define when the physical current-sense
  shunt changes; capacity/current-dependent registers recompute at
  compile time, verified with `_Static_assert`).
- `tools/serial_monitor.ps1` -- a small bidirectional PowerShell serial
  monitor for the ST-Link Virtual COM Port (auto-detects the port), used
  to watch the provisioning log and type the NVM-commit confirmation
  phrase back to the board.

## Design / safety notes

- Every I2C address, register address, command code, and timing value is
  taken directly from the MAX17320 datasheet (Table 93/94/95/116/15, the
  "Nonvolatile Block Programming" and "Determining Number of Remaining
  Updates" sections) -- nothing here is guessed.
- Shadow-RAM operations (`max17320_backup_config`,
  `max17320_write_shadow_config`, `max17320_verify_shadow_config`) never
  touch nonvolatile memory and are safe to run repeatedly.
- `max17320_commit_nvm()` -- the only function that actually burns a
  write cycle -- is gated twice: a compile-time
  `MAX17320_I_KNOW_THIS_BURNS_NVM` define (off by default; without it the
  function is a no-op) and a run-time confirmation token the caller must
  set deliberately.
- `max17320_read_remaining_nvm_updates()` should always be checked (and
  shown to a human) before ever calling `max17320_commit_nvm()` -- the
  part only supports 7 lifetime NV writes, one of which is consumed at
  Maxim's factory test.
- The post-write housekeeping (Config2.POR_CMD reset pulse, steps 9-12 of
  the datasheet's 12-step sequence) is split into its own
  `max17320_finish_post_commit_reset()` so a timing hiccup there can be
  retried without ever repeating the actual data-writing Copy NV Block
  step.

## Usage

Requires an STM32 HAL project with an initialized `I2C_HandleTypeDef`.
Typical flow:

```c
max17320_backup_config(&hi2c1, backup, MAX17320_TARGET_CONFIG_COUNT);
max17320_write_shadow_config(&hi2c1);                 // shadow RAM only
max17320_verify_shadow_config(&hi2c1, NULL, 0, NULL);  // reversible check

uint8_t used, remaining;
max17320_read_remaining_nvm_updates(&hi2c1, &used, &remaining);
// show `remaining` to a human, get explicit confirmation, THEN:

max17320_commit_nvm(&hi2c1, MAX17320_NVM_CONFIRM_TOKEN);  // irreversible
```
