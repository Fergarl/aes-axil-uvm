# UVM Verification of an AES-128 Accelerator (AXI4-Lite)

UVM Testbench for an AES-128 accelerator with AXI4-Lite interface using Verilator

## Status
🚧 Work in progress

## Features
- AXI4-Lite agent (driver, monitor and sequencer), environment and virtual sequences
- UVM register model (RAL): register block, adapter, predictor and built-in hw_reset/bit_bash sequences
- C reference model integrated via DPI-C
- SVA AXI4-Lite protocol checker connected with bind and cover property
- Functional coverage (covergroups) and code coverage with HTML report
- Regression: Makefiles with seeds, pass/fail summary and CI in GitHub Actions
- Verification plan linking features to tests and coverpoints 


## Credits

- AES core: [secworks/aes](https://github.com/secworks/aes) by Joachim Strömbergson,
  BSD-2-Clause license, included as a git submodule in `third_party/aes`.