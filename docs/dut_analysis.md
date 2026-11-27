# DUT Analysis: secworks `aes.v`

## Scope
- DUT: `third_party/aes/src/rtl/aes.v` (top) + `aes_core.v` and submodules
- Revision: secworks/aes @ `83cc270`, version register "0.60"
- Verification target: <AES-128 only, or AES-128 and AES-256?>

## Interface
| Port       | Dir | Width | Description                                                |
|------------|-----|-------|------------------------------------------------------------|
| clk        | in  | 1     | Clock, all logic on rising edge                            |
| reset_n    | in  | 1     | Reset, asynchronous, active low                            |
| cs         | in  | 1     | There is a transaction on the bus directed to this module. |
| we         | in  | 1     | Write enable when cs = 1, 1 for writing 0 for reading      |
| address    | in  | 8     | Which register is written or read                          |
| write_data | in  | 32    | Data that is written to the selected register              |
| read_data  | out | 32    | Data that is read from the selected register               |

## Bus protocol
- **Write:** single-cycle transaction. The master drives `cs = 1`, `we = 1`, 
  `address` and `write_data` for one clock cycle; the register is updated 
  on the next rising edge of `clk`.  
- **Read:** single-cycle transaction. The master drives `cs = 1`, `we = 0` and
  `address` for one clock cycle; `read_data` is combinational and 
  valid during that same cycle, so the master must sample it on the next
  rising edge of `clk`. Reads have no side effects.  
- **Handshake:** none. The slave always accepts a transaction in a single
  cycle; there is no ready or acknowledge signal.
- **Errors:** the bus has no error signal. A read from an unmapped address
  returns `0x00000000` in `read_data`; a write to an unmapped address is
  silently ignored. The master is never notified of an invalid access.

## Register map
| Addr      | Name            | Access | Fields                                                                                                                                                                                                      | Reset value                 |
|-----------|-----------------|--------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|-----------------------------|
| 0x00      | NAME0           | RO     | -                                                                                                                                                                                                           | 0x61657320 ("aes ")         |
| 0x01      | NAME1           | RO     | -                                                                                                                                                                                                           | 0x20202020 ("    ")         |
| 0x02      | VERSION         | RO     | -                                                                                                                                                                                                           | 0x302e3630 ("0.60")         |
| 0x08      | CTRL            | mixed  | [0] INIT (W, self-clearing), [1] NEXT (W, self-clearing), [2] ENCDEC (RO, mirror of CONFIG[0]), [3] KEYLEN (RO, mirror of CONFIG[1]), [31:4] reserved                                                       | 0x00000000                  |
| 0x09      | STATUS          | RO     | [0] READY, [1] VALID, [31:2] reserved                                                                                                                                                                       | 0x00000000 (volatile)       |
| 0x0a      | CONFIG          | WO     | [0] ENCDEC, [1] KEYLEN, [31:2] reserved  (readable via CTRL[3:2])                                                                                                                                           | 0x00000000                  |
| 0x10-0x17 | KEY0-KEY7       | WO     | KEY0 = bits [255:224], KEY1 = bits [223:192], KEY2 = bits [191:160], KEY3 = bits [159:128], KEY4 = bits [127:96],KEY5 = bits [95:64], KEY6 = bits [63:32], KEY7 = bits [31:0] (AES-128 uses KEY0–KEY3 only) | 0x00000000 (reads always 0) |
| 0x20-0x23 | BLOCK0-BLOCK3   | WO     | BLOCK0 = bits [127:96], BLOCK1 = bits [95:64],BLOCK2 = bits [63:32], BLOCK3 = bits[31:0]                                                                                                                    | 0x00000000 (reads always 0) |
| 0x30-0x33 | RESULT0-RESULT3 | RO     | RESULT0 = bits [127:96], RESULT1 = bits [95:64], RESULT2 = bits [63:32], RESULT3 = bits [31:0]                                                                                                              | 0x00000000 (volatile)       |

## Operation
<Programming sequence: what to write, in which order, how to
know it has finished. Approximate latency.>

## Findings
### F1: RESULT is not latched
<What the code does (line 197), the concrete scenario, the impact.
Status: hypothesis, to be reproduced by test.>

### F2: <encdec and keylen are written to CONFIG (0x0a, bits 0–1), but are read from CTRL (0x08, bits 2–3). Reading CONFIG always returns 0.>   

## Verification implications
<For each finding: which test or check will confirm it.>