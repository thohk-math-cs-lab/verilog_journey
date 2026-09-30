# verilog_journey

Use a **vertical-slice curriculum**, not a conventional course sequence: learn one narrowly scoped skill, implement it immediately in the Tiny Tapeout repository, and move on once its acceptance test passes.

## Delivery map

| Stage | Deliverable | Follow this material | Stop when |
|---:|---|---|---|
| 0 | Reliable simulation loop | Tiny Tapeout: **Testing Your Design** | You can deliberately introduce a bug, catch it with cocotb, locate it in GTKWave and fix it.  [blog.janestreet](https://blog.janestreet.com/protocol-emulator-asic-competition/) |
| 1 | Minimum usable Verilog | **HDLBits:** modules, vectors, combinational logic, DFFs, counters, shift registers and FSMs | A counter drives `uo_out[0]` at exact verified cycles. Do not complete the whole site.  [tinytapeout](https://tinytapeout.com/hdl/testing/) |
| 2 | Fixed UART transmitter | Nandland: **Project 8 – UART TX** | Bytes `00`, `55`, `A5`, `FF` produce correct start/data/stop timing.  [hdlbits.01xz](https://hdlbits.01xz.net/wiki/Main_Page) |
| 3 | Python verification | cocotb: **First Steps** | A Python decoder reconstructs UART bytes using only the TX pin.  [nandland](https://nandland.com/uart-serial-port-module/) |
| 4 | Fixed UART receiver | Nandland: **Project 7 – UART RX** | cocotb sends phase-offset UART waveforms and your RTL recovers the bytes.  [docs.cocotb](https://docs.cocotb.org/en/stable/first_steps.html) |
| 5 | Fetch/decode/execute core | Bruno Levy: **From Blinker to RISC-V, Steps 3–9** | A reduced core reads ROM instructions, branches, loops and changes an output register. Do not port all of RV32I.  [nandland](https://nandland.com/project-7-uart-part-1-receive-data-from-computer/) |
| 6 | Custom protocol ISA | **Building a Mini CPU**, followed by the MicroPython **RP2040 PIO tutorial** | You have a documented encoding for `SET`, `DIR`, `WAIT`, `IN`, `OUT`, `JMP`, `JNZ`, `PUSH`, `PULL` and `HALT`.  [github](https://github.com/BrunoLevy/learn-fpga/tree/master/FemtoRV/TUTORIALS/FROM_BLINKER_TO_RISCV) |
| 7 | Assembler and program memory | Bruno Levy Step 7: machine words and `$readmemh()` | `assembler.py uart.asm > uart.hex` loads and executes without modifying RTL.  [nandland](https://nandland.com/project-7-uart-part-1-receive-data-from-computer/) |
| 8 | Programmable UART | Reuse Nandland UART as a reference oracle | The generic engine—not a UART FSM—executes `uart_tx.hex` and reproduces the reference waveform.  [nandland](https://nandland.com/project-8-uart-part-2-transmit-data-to-computer/) |
| 9 | Programmable SPI | EcrioniX: **SPI Master Controller in Verilog**, cross-check against Nandland SPI | The unchanged engine runs SPI Mode 0 and Mode 3 programs, including simultaneous MISO capture.  [docs.micropython](https://docs.micropython.org/en/v1.23.0/rp2/tutorial/pio.html) |
| 10 | Programmable I²C | EcrioniX: **I²C Master**, then fpga4fun I²C | Microcode generates START/address/ACK/data/STOP using `uio_oe` for drive-low/release behavior.  [deepwiki](https://deepwiki.com/nandland/spi-master/4.1-verilog-spi_master-module) |
| 11 | Host loader/interface | Bruno Levy Step 17: **memory-mapped I/O** | An external host can load instructions/data, start execution, poll status and retrieve received data.  [github](https://github.com/BrunoLevy/learn-fpga/blob/master/FemtoRV/TUTORIALS/FROM_BLINKER_TO_RISCV/README.md) |
| 12 | Complete regression | Extend the cocotb tests from Stage 3 | One command tests every opcode, UART, SPI, I²C, resets, boundaries and randomized payloads.  [nandland](https://nandland.com/uart-serial-port-module/) |
| 13 | Formal safety checks | SymbiYosys: **Getting Started** | Proofs cover reset state, PC bounds, output-enable safety and bounded instruction retirement.  [fpga4fun](https://www.fpga4fun.com/I2C.html) |
| 14 | Physical implementation | Tiny Tapeout: **Hardening Projects Locally** | The 6×4 design completes synthesis, placement and routing without fatal timing/DRC problems.  [symbiyosys.readthedocs](https://symbiyosys.readthedocs.io/en/latest/quickstart.html) |
| 15 | Gate-level testing | Tiny Tapeout template test guide | Loader, instruction, UART and SPI smoke tests pass with `GATES=yes`.  [tinytapeout](https://tinytapeout.com/guides/local-hardening/) |
| 16 | Submission package | Tiny Tapeout template instructions | A fresh user can clone, test, assemble firmware, harden and understand the design.  [github](https://github.com/TinyTapeout/tt07-verilog-template/blob/main/test/README.md) |

## Initial ISA

Start with a small instruction set rather than inventing everything at once:

| Instruction | Immediate purpose |
|---|---|
| `SET mask,value` | Change output-pin values |
| `DIR mask,value` | Configure bidirectional output enables |
| `WAIT cycles` | Generate deterministic delays |
| `WAITPIN pin,level` | Wait for an external signal |
| `OUT count` | Shift bits from the TX register onto pins |
| `IN count` | Sample pins into the RX register |
| `JMP address` | Repeat protocol sequences |
| `JNZ reg,address` | Implement bit-count and timeout loops |
| `PULL` | Obtain transmission data from the host |
| `PUSH` | Return captured data to the host |
| `HALT` | End a protocol job predictably |

RP2040 PIO is the best architectural reference because it similarly uses a compact instruction set for `JMP`, `WAIT`, `IN`, `OUT`, `PUSH`, `PULL`, `MOV` and `SET`, with instruction-level timing controls. Study its semantics, but design a smaller encoding suited to your 6×4-tile budget. [siliconwit](https://siliconwit.com/education/fpga-digital-design-verilog/building-a-mini-cpu/)

## Learning rule

For every tutorial, follow this three-pass pattern:

1. **Copy and run:** reproduce the author’s result unchanged.
2. **Rebuild:** recreate the essential module without looking at the implementation.
3. **Translate:** integrate only the concept into your Tiny Tapeout architecture.

Do not count watching videos as progress. Each stage ends with committed RTL, tests, waveforms or physical reports.

## First seven sessions

1. Preserve the passing template with a Git tag; inject and diagnose one counter bug.
2. Complete selected HDLBits exercises; implement an eight-cycle output toggle.
3. Reproduce Nandland’s UART transmitter and testbench.
4. Port UART TX into the Tiny Tapeout module hierarchy.
5. Write a cocotb UART decoder and test `00`, `55`, `A5`, `FF`.
6. Rewrite UART TX from scratch until the same tests pass.
7. Begin Bruno Levy Steps 3–5: instruction memory, decoder, register bank and execution FSM.

This gives you a real protocol deliverable before studying processor internals. The fixed UART then becomes the executable specification against which the programmable machine is tested.

## Defer until later

Do not study these before the submission is functionally stable:

- Full RISC-V
- Pipelining, caches and branch prediction
- AXI/APB
- UVM and class-based SystemVerilog
- CDC unless you introduce another clock domain
- Vendor FPGA IP
- Transistor-level CMOS design
- Hardcaml migration
- Advanced DFT
