# Protocol Emulator ASIC — Learning & Build Map

This repository is a hands-on learning path for building an open-source, reprogrammable protocol-emulator ASIC for the Jane Street competition.

**Working rule:** at every stage, copy and run the smallest example, rebuild the useful part, then integrate it into the competition repository. A stage is complete only when its acceptance check passes.

## Competition and template

- [Jane Street — Protocol Emulator ASIC Competition](https://blog.janestreet.com/protocol-emulator-asic-competition/)
- [Tiny Tapeout IHP Verilog Template](https://github.com/TinyTapeout/ttihp-verilog-template)
- [Tiny Tapeout — Testing Your Design](https://tinytapeout.com/hdl/testing/)
- [Tiny Tapeout — Local Hardening with LibreLane](https://tinytapeout.com/guides/local-hardening/)

## Delivery stages

| Stage | Build outcome | Exact lesson/chapter to follow | Completion check |
|---:|---|---|---|
| 0 | Edit, simulate, test and waveform-debug loop | [Tiny Tapeout: Writing Your First Test](https://tinytapeout.com/hdl/testing/) — complete **Getting Started**, **Writing a Test**, **Running Tests**, **Viewing Waveforms**, and **GitHub Actions** | Introduce one known bug, catch it in cocotb, inspect it in GTKWave, then fix it |
| 1 | Basic synthesizable RTL | [HDLBits problem index](https://hdlbits.01xz.net/wiki/Problem_sets) — complete **Verilog Language > Modules**; **Vectors**; **Basic Gates**; **Combinational Logic**; **Sequential Logic > Flip-Flops**; **Counters**; **Shift Registers**; **Finite State Machines** | `uo_out[0]` toggles every eight clocks, with reset and cocotb timing assertions |
| 2 | Fixed UART transmitter oracle | [Nandland Project 8 — UART Part 2: Transmit Data](https://nandland.com/project-8-uart-part-2-transmit-data-to-computer/) — follow the full article: state diagram, Verilog TX module, testbench and waveform | Correct start, data and stop timing for `00`, `55`, `A5`, and `FF` |
| 3 | Python simulation scoreboard | [cocotb: Quickstart](https://docs.cocotb.org/en/stable/quickstart.html) — complete **Install cocotb**, **Write a Test**, **Run a Test**, **Accessing Handles**, and **Logging** | Python decodes bytes solely from the UART TX pin |
| 4 | Fixed UART receiver oracle | [Nandland Project 7 — UART Part 1: Receive Data](https://nandland.com/project-7-uart-part-1-receive-data-from-computer/) — follow the full article: start-bit detection, sampling timing, RX module and testbench | cocotb-generated UART frames are received across phase offsets |
| 5 | Tiny fetch/decode/execute core | [Bruno Levy: From Blinker to RISC-V](https://github.com/BrunoLevy/learn-fpga/blob/master/FemtoRV/TUTORIALS/FROM_BLINKER_TO_RISCV/README.md) — complete **Steps 3–9 only**: ROM, instruction decode, register bank, fetch/decode/execute, ALU, assembler, branches/jumps | ROM program changes an output, decrements a register and branches in a loop |
| 6 | Protocol-machine ISA | [SiliconWit: Building a Mini CPU](https://siliconwit.com/education/fpga-digital-design-verilog/building-a-mini-cpu/) — complete the page in order; then [MicroPython RP2040 PIO](https://docs.micropython.org/en/latest/rp2/tutorial/pio.html) — read **PIO program**, **state machine**, and every `@asm_pio()` example | `docs/isa.md` specifies every instruction and its exact cycle/pin behavior |
| 7 | Assembler and program loading | [Bruno Levy Step 7 — using the VERILOG assembler](https://github.com/BrunoLevy/learn-fpga/blob/master/FemtoRV/TUTORIALS/FROM_BLINKER_TO_RISCV/README.md#step-7-using-the-verilog-assembler) — adapt its machine-word/program-memory pattern; use `$readmemh()` for simulation | `assemble.py firmware/uart_tx.asm > firmware/uart_tx.hex` runs without RTL edits |
| 8 | Programmable UART | [Nandland UART serial-port module](https://nandland.com/uart-serial-port-module/) — use the **UART Transmitter**, **UART Receiver**, and linked testbench as a waveform oracle | Generic engine executes `uart_tx.hex`; the fixed UART FSM is not on the submission datapath |
| 9 | Programmable SPI | [EcrioniX SPI protocol guide](https://ecrionix.org/protocols/spi/) — read **Signals**, **CPOL/CPHA modes**, **timing diagrams**, and **Verilog RTL master/testbench**; then inspect [Nandland SPI master `src/`](https://github.com/nandland/spi-master/tree/master/src) | One engine runs Mode 0 and Mode 3 microcode with MOSI/MISO verification |
| 10 | Programmable I²C | [EcrioniX I²C guide](https://ecrionix.org/serial-protocols/i2c/) — read **Open-Drain Bus**, **START/STOP**, **Addressing**, **ACK/NACK**, **Clock Stretching**, and **Verilog Master FSM**; then complete [fpga4fun I²C Parts 1–4](https://www.fpga4fun.com/I2C.html) | Microcode performs START, address, ACK, data and STOP using open-drain drive/release |
| 11 | Host loader and result interface | [Bruno Levy Step 17 — System Calls and I/O Devices](https://github.com/BrunoLevy/learn-fpga/blob/master/FemtoRV/TUTORIALS/FROM_BLINKER_TO_RISCV/README.md#step-17-system-calls-and-io-devices) — adapt its memory-mapped data/status-register pattern | Host can load a program/data, start it, read status, and retrieve captured data |
| 12 | Repeatable regression | [cocotb: Writing Testbenches](https://docs.cocotb.org/en/stable/writing_testbenches.html) — complete **Triggers**, **Concurrent Execution**, and **Assertions**; use the [Regression Manager API](https://docs.cocotb.org/en/stable/library_reference.html#cocotb.regression) only when needed | One command tests ISA, UART, SPI, I²C, resets, limits and seeded randomized cases |
| 13 | Formal safety checks | [SymbiYosys quickstart](https://symbiyosys.readthedocs.io/en/latest/quickstart.html) — complete **Example: FIFO**, including `bmc`, `prove`, and `cover`; then read [Formal Verilog extensions](https://symbiyosys.readthedocs.io/en/latest/verilog.html): `assert`, `assume`, `cover`, `$past`, and `FORMAL` | Prove reset behavior, program-counter bounds, output-enable safety, and instruction-retirement rules |
| 14 | RTL-to-GDS hardening | [Tiny Tapeout: Hardening Projects Locally](https://tinytapeout.com/guides/local-hardening/) — complete **Environment Setup**, **Setup Dependencies**, **Harden a Design**, and the **IHP** command variant | 6×4 design completes synthesis, place-and-route and timing checks; archive reports/GDS |
| 15 | Gate-level sign-off | [Tiny Tapeout: Gate-Level Testing](https://tinytapeout.com/hdl/testing/#gate-level-testing) — run the documented `make -B GATES=yes` flow | Loader/core/UART/SPI smoke tests pass against the gate-level netlist |
| 16 | Submission-ready repository | [IHP template README](https://github.com/TinyTapeout/ttihp-verilog-template) — complete **Set up your project**, **`info.yaml`**, **Testing**, and documentation requirements; then re-read the [competition brief](https://blog.janestreet.com/protocol-emulator-asic-competition/) | A new contributor can clone, test, assemble protocol firmware, harden, and understand the design |

## Initial ISA target

Design a small custom instruction set; do not implement full RISC-V or clone RP2040 PIO.

| Instruction | Required effect |
|---|---|
| `SET mask, value` | Set selected output-pin values |
| `DIR mask, value` | Set selected bidirectional output enables |
| `WAIT cycles` | Delay by a deterministic number of core cycles |
| `WAITPIN pin, level` | Stall until an input reaches a level |
| `OUT count` | Shift transmit-register bits onto protocol pins |
| `IN count` | Sample protocol pins into a receive register |
| `JMP address` | Unconditional loop/branch |
| `JNZ reg, address` | Conditional bit-count or timeout loop |
| `PULL` | Receive host-provided transmit data |
| `PUSH` | Return captured data to the host |
| `HALT` | End the current program predictably |

## Required repository evidence

The final competition repository should contain:

- Architecture/block diagram and pin map
- ISA specification including instruction cycle timing
- Host program-loading and result-retrieval protocol
- Python assembler and usage example
- UART, SPI and I²C assembly programs
- Reproducible cocotb regression tests
- SymbiYosys properties and proof commands/results
- Synthesis, place-and-route and timing reports
- Gate-level test evidence
- Known limits, tested clock rates and protocol modes
- An open-source license

## Primary tool references

- [OSS CAD Suite](https://github.com/YosysHQ/oss-cad-suite-build)
- [Icarus Verilog](https://steveicarus.github.io/iverilog/)
- [Verilator](https://www.veripool.org/verilator/)
- [GTKWave](https://gtkwave.github.io/gtkwave/)
- [cocotb](https://www.cocotb.org/)
- [SymbiYosys](https://github.com/YosysHQ/SymbiYosys)
- [LibreLane](https://github.com/librelane/librelane)

## First seven work sessions

1. Tag the passing template; inject, diagnose and fix one counter bug.
2. Complete the selected HDLBits exercises; implement an eight-cycle output toggle in the real template.
3. Reproduce Nandland UART TX and its waveform.
4. Port UART TX to the Tiny Tapeout hierarchy; add cocotb checking for `0x55`.
5. Extend the cocotb decoder for `00`, `A5`, `FF` and two baud divisors.
6. Rebuild UART TX from scratch until the tests pass.
7. Begin Bruno Levy Steps 3–5 in the learning repository: instruction memory, decoder, register bank and execution FSM.

