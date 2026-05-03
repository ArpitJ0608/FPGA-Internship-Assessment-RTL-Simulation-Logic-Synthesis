# FPGA Internship Assessment – RTL Simulation & Logic Synthesis
**VSD Open EDA Lab | 12-Hour Assessment Submission**

---

## Repository Structure

```
├── Day1_RTL_Simulation/
│   ├── good_mux/
│   │   ├── good_mux.v
│   │   ├── tb_good_mux.v
│   │   └── waveforms/
│   │       ├── GTK_SIM1.png
│   │       └── GTK_SIM2.png
│   ├── dff_asyncres/
│   │   ├── dff_asyncres.v
│   │   ├── tb_dff_asyncres.v
│   │   └── waveforms/
│   │       ├── GTK_ASYRES_SIM1.png
│   │       ├── GTK_ASYRES_SIM2.png
│   │       └── GTK_ASYRES_SIM3.png
│   ├── dff_async_set/
│   │   ├── dff_async_set.v
│   │   ├── tb_dff_async_set.v
│   │   └── waveforms/
│   │       ├── GTK_ASY_SET_S1.png
│   │       └── GTK_ASY_SET_S2.png
│   └── dff_syncres/
│       ├── dff_syncres.v
│       ├── tb_dff_syncres.v
│       └── waveforms/
│           ├── GTK_SYRES_S1.png
│           └── GTK_SYRES_S2.png
├── Day2_Synthesis/
│   ├── good_mux/
│   │   ├── mux.ys
│   │   ├── block_goodmux.png
│   │   ├── synth_sim1.png
│   │   └── synth_sim2.png
│   ├── multiple_modules/
│   │   ├── multiple_modules.v
│   │   ├── multi_hier.ys
│   │   ├── multi_flat.ys
│   │   ├── block_multi_hier.png
│   │   ├── block_multi_flat.png
│   │   ├── block_subm1.png
│   │   ├── synth_multi_hier_res1.png
│   │   ├── synth_multi_hier_res2.png
│   │   └── synth_multi_flat_res.png
│   ├── mul2_mul8/
│   │   ├── mult_2.v
│   │   ├── mult_8.v
│   │   ├── mult2.ys / mult8.ys
│   │   ├── block_mult2.png
│   │   ├── block_mult8.png
│   │   ├── synth_mult2_res.png
│   │   └── synth_mult8_res.png
│   └── dff_variants/
│       ├── asyres.ys / asyset.ys
│       ├── block_asyres.png
│       ├── block_asyset.png
│       ├── block_syncres.png
│       ├── synth_asynres_res.jpeg
│       ├── synth_asyn_setres.jpeg
│       └── synth_syncres_res.png
└── README.md
```

---

## Tools Used

| Tool | Purpose |
|---|---|
| **Icarus Verilog (iverilog)** | RTL compilation and simulation |
| **GTKWave** | Waveform viewer for `.vcd` dumps |
| **Yosys** | Logic synthesis |
| **SKY130 PDK** (`sky130_fd_sc_hd__tt_025C_1v80.lib`) | Standard cell library for synthesis |

---

## Day 1 – RTL Simulation & Waveform Analysis

### Overview

The objective of Day 1 was to write Verilog RTL modules with corresponding testbenches, simulate them using Icarus Verilog, and analyze the resulting signal waveforms in GTKWave. Each design is compiled and simulated using the following flow:

```bash
iverilog -o sim_out <module>.v <testbench>.v
vvp sim_out
gtkwave <dumpfile>.vcd
```

---

### Module 1 – 2:1 Multiplexer (`good_mux`)

#### RTL Code

```verilog
module good_mux (input i0, input i1, input sel, output reg y);
  always @ (*) begin
    if (sel)
      y <= i1;
    else
      y <= i0;
  end
endmodule
```

**Design Description:**  
A simple 2-to-1 multiplexer. When `sel = 0`, the output `y` follows input `i0`. When `sel = 1`, the output `y` follows input `i1`. The `always @(*)` sensitivity list makes this a purely combinational block — any change in `sel`, `i0`, or `i1` immediately triggers a re-evaluation of `y`.

#### Testbench

```verilog
`timescale 1ns / 1ps
module tb_good_mux;
  reg i0, i1, sel;
  wire y;

  good_mux uut (.sel(sel), .i0(i0), .i1(i1), .y(y));

  initial begin
    $dumpfile("tb_good_mux.vcd");
    $dumpvars(0, tb_good_mux);
    sel = 0; i0 = 0; i1 = 0;
    #300 $finish;
  end

  always #75  sel = ~sel;
  always #10  i0  = ~i0;
  always #55  i1  = ~i1;
endmodule
```

**Stimulus Explanation:**  
- `i0` toggles every 10 ns → fast toggling input  
- `i1` toggles every 55 ns → slower toggling input  
- `sel` toggles every 75 ns → selects between the two inputs every half-period  
- Total simulation time: 300 ns

---

#### Waveform Analysis – GTK_SIM1 (Marker at 121 ns, `sel = 1`)

![GTK_SIM1](Day1_RTL_Simulation/good_mux/waveforms/GTK_SIM1.png)

At **t = 121 ns**, the marker is placed with `sel = 1`. In this region:

- **`sel` went HIGH at t = 75 ns** (after the first 75 ns half-period), switching the mux to track `i1`.
- **`i1 = 0`** at this moment (i1 just toggled to 0 at t = 110 ns), so **`y = 0`**.
- Before t = 75 ns (i.e., `sel = 0`), `y` was tracking `i0` exactly — you can see `y` mirroring the fast 10 ns toggling pattern of `i0` in the left portion of the waveform.
- After t = 75 ns (sel goes HIGH), `y` stops following `i0` and now follows the slower `i1` pattern. This is clearly visible as `y` becomes less frequent in its transitions, matching `i1`'s 55 ns period.
- The transition at the marker boundary confirms the mux switching behavior: the output instantaneously changes its source from `i0` to `i1` the moment `sel` rises, validating the combinational sensitivity of the always block.

---

#### Waveform Analysis – GTK_SIM2 (Marker at 64900 ps, `sel = 0`)

![GTK_SIM2](Day1_RTL_Simulation/good_mux/waveforms/GTK_SIM2.png)

At **t ≈ 65 ns**, `sel = 0`, so `y` is tracking `i0`:

- **`i1 = 1`** throughout most of this early window (i1 starts at 0, flips to 1 at 55 ns).
- **`i0`** is rapidly toggling every 10 ns — `y` mirrors this exactly in the `sel = 0` region.
- At approximately **t = 150 ns**, `sel` goes HIGH (second blue vertical marker). At that exact instant, `y` transitions to follow `i1`. Since `i1 = 1` at that point, `y` jumps to 1 and stays there until `i1` changes again.
- The two blue cursor lines at ~100 ns and ~150 ns bracket the region where `sel` is HIGH, clearly showing `y` behaving differently (tracking `i1`'s pattern) vs. the `sel = 0` regions.

**Key Observation:** The mux correctly demonstrates zero-glitch switching — `y` always immediately reflects the correct input without any intermediate undefined state, confirming the RTL is functionally correct.

---

### Module 2 – D Flip-Flop with Asynchronous Reset (`dff_asyncres`)

#### RTL Code

```verilog
module dff_asyncres (input clk, input async_reset, input d, output reg q);
  always @ (posedge clk, posedge async_reset) begin
    if (async_reset)
      q <= 1'b0;
    else
      q <= d;
  end
endmodule
```

**Design Description:**  
A D flip-flop with active-high asynchronous reset. The sensitivity list includes both `posedge clk` and `posedge async_reset`. This means:
- When `async_reset` goes HIGH, `q` is **immediately forced to 0**, regardless of the clock state.
- When `async_reset` is LOW, `q` captures the value of `d` on every rising clock edge.
- "Asynchronous" means the reset does not wait for the clock — it overrides everything instantly.

#### Testbench

```verilog
`timescale 1ns / 1ps
module tb_dff_asyncres;
  reg clk, async_reset, d;
  wire q;

  dff_asyncres uut (.clk(clk), .async_reset(async_reset), .d(d), .q(q));

  initial begin
    $dumpfile("tb_dff_asyncres.vcd");
    $dumpvars(0, tb_dff_asyncres);
    clk = 0; async_reset = 1; d = 0;
    #3000 $finish;
  end

  always #10  clk         = ~clk;          // 20 ns clock period
  always #23  d           = ~d;
  always #547 async_reset = ~async_reset;  // reset toggles every 547 ns
endmodule
```

**Stimulus Explanation:**  
- Clock period: 20 ns (50 MHz).  
- `d` toggles every 23 ns — not aligned to clock, so it captures arbitrary data.  
- `async_reset` starts HIGH (reset active), and toggles every 547 ns, creating long stretches of reset-active and reset-inactive regions.

---

#### Waveform Analysis – GTK_ASYRES_SIM1 (Marker at 1596500 ps / ~1597 ns)

![GTK_ASYRES_SIM1](Day1_RTL_Simulation/dff_asyncres/waveforms/GTK_ASYRES_SIM1.png)

This screenshot shows the transition region around **t = 1596 ns** where `async_reset` **deasserts (goes LOW)**:

- In the left half of this view, `async_reset = 1` (HIGH). During this period, `q` is held firmly at **0** regardless of what `d` or `clk` do — this is the asynchronous reset in action.
- At approximately **t = 1600 ns** (right side of the red marker), `async_reset` falls LOW. From this point, the DFF begins normal operation: it captures `d` at the next rising edge of `clk`.
- **`d = 1`** just before reset deasserts, so on the very next positive clock edge after reset goes LOW, `q` immediately jumps from 0 to 1 — visible as the rising edge of `q` in the right portion.
- This confirms that the reset release is clean and the flip-flop resumes data capture without any glitch or delay beyond one clock edge.

---

#### Waveform Analysis – GTK_ASYRES_SIM2 (Marker at 1641 ns — zoomed detail)

![GTK_ASYRES_SIM2](Day1_RTL_Simulation/dff_asyncres/waveforms/GTK_ASYRES_SIM2.png)

This is a zoomed-in view of the **1620–1660 ns** region with `async_reset = 0` (reset inactive):

- `d` toggles mid-window (LOW → HIGH around 1632 ns, then HIGH → LOW around 1655 ns).
- `clk` transitions are clearly visible every 10 ns.
- `q` updates only on the **rising edge of clk** — you can see `q` staying low until `clk` rises, then picking up `d`'s current value.
- The red marker at **1641 ns** is positioned just after a positive clock edge where `d = 1`, confirming `q` has captured and held `1`.
- This zoomed view is critical for verifying setup/hold relationship: `d` is stable before each clock edge, so `q` captures correctly with no metastability.

---

#### Waveform Analysis – GTK_ASYRES_SIM3 (Marker at 1094 ns — reset assertion)

![GTK_ASYRES_SIM3](Day1_RTL_Simulation/dff_asyncres/waveforms/GTK_ASYRES_SIM3.png)

This view shows the moment `async_reset` **goes HIGH** (asserts) at approximately **t = 1094 ns** (red marker):

- Before the marker, `async_reset = 1` (already active from the very start of simulation), so `q` is held at 0 throughout the early window shown.
- At around **t = 1094 ns**, we are near the boundary where `async_reset` transitions. Even though `d` is toggling and `clk` continues to pulse, `q` **does not change** while reset is asserted — `q` stays at 0 with no clock-dependent behavior.
- After the marker (blue cursor at ~1100 ns), you can observe `q` beginning to follow `d` values at clock edges as the reset de-asserts at `t = 1094 ns`.
- This screenshot beautifully illustrates the **asynchronous nature** of the reset: `q` doesn't wait for the next clock edge to respond to `async_reset` going HIGH — it responds immediately mid-cycle.

---

### Module 3 – D Flip-Flop with Asynchronous Set (`dff_async_set`)

#### RTL Code

```verilog
module dff_async_set (input clk, input async_set, input d, output reg q);
  always @ (posedge clk, posedge async_set) begin
    if (async_set)
      q <= 1'b1;
    else
      q <= d;
  end
endmodule
```

**Design Description:**  
This is the complement of the async reset DFF. When `async_set` asserts HIGH, `q` is **immediately forced to 1** regardless of clock. When `async_set` is LOW, `q` captures `d` on rising clock edges. The critical difference from the reset variant: instead of clearing `q` to 0, this sets `q` to 1, which maps to a different standard cell in the library (`dfstp` instead of `dfrtp`).

#### Testbench

```verilog
`timescale 1ns / 1ps
module tb_dff_async_set;
  reg clk, async_set, d;
  wire q;

  dff_async_set uut (.clk(clk), .async_set(async_set), .d(d), .q(q));

  initial begin
    $dumpfile("tb_dff_async_set.vcd");
    $dumpvars(0, tb_dff_async_set);
    clk = 0; async_set = 1; d = 0;
    #3000 $finish;
  end

  always #10  clk       = ~clk;
  always #23  d         = ~d;
  always #547 async_set = ~async_set;
endmodule
```

The testbench is structurally identical to the async reset testbench — same timing parameters — which allows direct behavioral comparison between the two modules.

---

#### Waveform Analysis – GTK_ASY_SET_S1 (Marker at 570 ns)

![GTK_ASY_SET_S1](Day1_RTL_Simulation/dff_async_set/waveforms/GTK_ASY_SET_S1.png)

This view covers the **500–650 ns** region, showing the **deassert** of `async_set`:

- In the left portion, `async_set = 1` — `q` is **held at 1** (HIGH) because the async set overrides everything. Even though `d` is toggling and clk continues, `q` does not move.
- At approximately **t = 547 ns**, `async_set` goes LOW (deasserts). This is the handoff point.
- The marker at **570 ns** is placed right after this deassert. Looking at `q` after the marker: `q` was at 1 (due to the set), and since `d = 0` at the next clock edge, `q` **drops to 0** — the DFF is now tracking `d` normally.
- This transition from forced-HIGH to normal clock-sampled behavior is clearly captured: `q` goes from a flat-1 line to a toggling pattern matching `d` at clock edges.

---

#### Waveform Analysis – GTK_ASY_SET_S2 (Marker at 1094 ns — async set assertion)

![GTK_ASY_SET_S2](Day1_RTL_Simulation/dff_async_set/waveforms/GTK_ASY_SET_S2.png)

This view shows the **1050–1200+ ns** range where `async_set` **goes HIGH** again:

- Before the red marker at **t = 1094 ns**, `async_set = 0`. `q` is toggling normally with `clk` and `d` — you can see `q` going HIGH and LOW following clock edges.
- At **t = 1094 ns** (red marker), `async_set` rises. **Immediately** (not at the next clock edge), `q` is forced to **1**. This is clearly visible as `q` jumps to HIGH mid-clock-cycle, not aligned to any clock edge.
- After this point, `q` stays at 1 indefinitely until `async_set` de-asserts, regardless of `d` or `clk`.
- **Key insight:** The mid-cycle assertion response (q goes 1 without waiting for clk edge) is the defining characteristic of an *asynchronous* set, distinguishing it from a synchronous one. This is exactly what this waveform demonstrates.

---

### Module 4 – D Flip-Flop with Synchronous Reset (`dff_syncres`)

#### RTL Code

```verilog
module dff_syncres (input clk, input async_reset, input sync_reset, input d, output reg q);
  always @ (posedge clk) begin
    if (sync_reset)
      q <= 1'b0;
    else
      q <= d;
  end
endmodule
```

**Design Description:**  
This DFF has **only `posedge clk`** in its sensitivity list — no async signal. `sync_reset` is evaluated **only at the rising edge of the clock**. If `sync_reset` is HIGH when a clock edge arrives, `q` resets to 0. Otherwise, `q` captures `d`. The reset can only take effect at a clock boundary — this is the fundamental difference from the async variants. Note: The port `async_reset` in the module definition is unused (it's declared but not referenced in the `always` block), so it has no effect.

#### Testbench

```verilog
`timescale 1ns / 1ps
module tb_dff_syncres;
  reg clk, sync_reset, d;
  wire q;

  dff_syncres uut (.clk(clk), .sync_reset(sync_reset), .d(d), .q(q));

  initial begin
    $dumpfile("tb_dff_syncres.vcd");
    $dumpvars(0, tb_dff_syncres);
    clk = 0; sync_reset = 0; d = 0;
    #3000 $finish;
  end

  always #10  clk        = ~clk;
  always #23  d          = ~d;
  always #113 sync_reset = ~sync_reset; // shorter period than async versions
endmodule
```

Notable difference: `sync_reset` toggles every 113 ns (much more frequently than the 547 ns in async variants), creating many visible reset/normal alternation windows across the 3 µs simulation.

---

#### Waveform Analysis – GTK_SYRES_S1 (Marker at 565 ns — sync reset active)

![GTK_SYRES_S1](Day1_RTL_Simulation/dff_syncres/waveforms/GTK_SYRES_S1.png)

This view covers **400–750 ns**, showing multiple `sync_reset` windows:

- **`sync_reset = 1`** in the first visible window (approximately 400–565 ns). During this time, `q` is driven to 0 **on each clock edge** — not immediately when `sync_reset` rises, but at the very next positive clock edge.
- **`d`** is toggling throughout this window but has no effect — every clock edge samples `sync_reset = 1`, so `q` is repeatedly reset to 0.
- At the **red marker (565 ns)**, `sync_reset` deasserts. However, looking closely, `q` does **not change immediately at this 565 ns point** — it waits for the next rising edge of `clk` before capturing `d`.
- After the marker, `sync_reset = 0` and `q` begins following `d`'s values at each clock edge — the toggling `q` pattern resumes.
- **Contrast with async:** If this were an async reset, `q` would spring to 0 the instant `sync_reset` went HIGH, even between clock edges. Here, `q` only updates at clock boundaries.

---

#### Waveform Analysis – GTK_SYRES_S2 (Marker at 1130 ns)

![GTK_SYRES_S2](Day1_RTL_Simulation/dff_syncres/waveforms/GTK_SYRES_S2.png)

This view covers the **1050–1250 ns** region with `sync_reset = 0`:

- `sync_reset` is LOW throughout (having de-asserted before this view), so `q` is in normal data-capture mode.
- `d = 1` for most of this window (only toggling briefly), and `q` captures `d` at each rising clock edge.
- At the **red marker (1130 ns)**: `clk` is rising, `d = 1`, so `q` captures 1 and remains HIGH.
- `q` transitions are clearly aligned with rising clock edges — note how `q` never changes in the middle of a clock cycle, only right at the clock edge.
- This view confirms synchronous operation: all state changes are strictly clock-edge driven.

**Critical Comparison — Async vs Sync Reset:**

| Property | `dff_asyncres` / `dff_async_set` | `dff_syncres` |
|---|---|---|
| Sensitivity list | `posedge clk, posedge reset/set` | `posedge clk` only |
| Reset/Set response | Immediate (mid-cycle) | Only at next clock rising edge |
| Waveform indicator | `q` changes without clock edge | `q` only changes on clock edge |
| Standard cell | `dfrtp_1` / `dfstp_2` | `dfxtp_1` + `nor2b_1` |

---

---

## Day 2 – Logic Synthesis & Optimization

### Overview

Day 2 focuses on synthesizing the RTL designs using **Yosys** with the **SKY130 HD standard cell library** (`sky130_fd_sc_hd__tt_025C_1v80.lib`). The synthesis flow converts behavioral Verilog into a gate-level netlist mapped to real standard cells. The `show` command in Yosys generates a visual schematic of the synthesized netlist.

General Yosys flow:
```
read_verilog <module>.v
synth -top <module>
dfflibmap -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
clean
show
stat -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
write_verilog -noattr <netlist>.v
```

---

### Synthesis 1 – `good_mux`

**Yosys Script (`mux.ys`):**
```
read_verilog good_mux.v
hierarchy -check -top good_mux
proc; opt;
synth -top good_mux -flatten
dfflibmap -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
techmap; opt;
abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
show
stat -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
write_verilog -noattr mux_netlist.v
```

#### Synthesized Schematic

![block_goodmux](Day2_Synthesis/good_mux/block_goodmux.png)

**Schematic Analysis:**  
- `i0`, `i1`, and `sel` each pass through **input buffers (BUF)** — standard cell libraries add input buffers for signal integrity.
- The buffered inputs feed into a **`sky130_fd_sc_hd__mux2_1`** cell (`$97`) — a native 2-to-1 mux from the SKY130 library with ports A0 (i0), A1 (i1), S (sel), and X (output).
- The output passes through a final **output buffer (BUF)** before reaching port `y`.
- Yosys correctly inferred a single MUX2 standard cell — no decomposition into NANDs or NORs was needed since SKY130 has a dedicated mux cell. This is optimal synthesis.

#### Gate-Level Simulation Waveforms

**Post-Synthesis Simulation 1 (`synth_sim1.png` — `sel = 0`):**

![synth_sim1](Day2_Synthesis/good_mux/synth_sim1.png)

The post-synthesis netlist simulation includes internal wires `_0_`, `_1_`, `_2_`, `_3_` (generated by Yosys for internal net connections through the BUFs and mux cell). With `sel = 0`:
- `y` tracks `i0` faithfully — the fast-toggling 10 ns pattern is reproduced at the output.
- The internal wires show intermediate signal states passing through the BUF stages, all settling correctly.
- The final `y` output matches the RTL simulation, confirming **functional equivalence** between the RTL and the synthesized gate-level netlist.

**Post-Synthesis Simulation 2 (`synth_sim2.png` — `sel = 0` ending, transitioning to `sel = 1`):**

![synth_sim2](Day2_Synthesis/good_mux/synth_sim2.png)

With `sel = 0` in the early portion, `y` tracks `i0` (which is 1 at the start of this view). At approximately t = 150 ns, `sel` goes HIGH, and `y` switches to track `i1`. The internal wires again show the propagation through the BUF chain. This simulation validates that the synthesized netlist correctly switches its output source with zero functional error — **RTL-to-gates correctness is confirmed**.

---

### Synthesis 2 – `multiple_modules` (Hierarchical vs Flat Synthesis)

#### Module Code

```verilog
module sub_module1 (input a, input b, output y);
  assign y = a & b;   // AND gate
endmodule

module sub_module2 (input a, input b, output y);
  assign y = a | b;   // OR gate
endmodule

module multiple_modules (input a, input b, input c, output y);
  wire net1;
  sub_module1 u1 (.a(a), .b(b), .y(net1));  // net1 = a & b
  sub_module2 u2 (.a(net1), .b(c), .y(y)); // y = (a & b) | c
endmodule
```

**Design Description:**  
A top-level module instantiating two sub-modules. `sub_module1` computes AND, `sub_module2` computes OR. Together: `y = (a & b) | c`.

---

#### Hierarchical Synthesis

**Script (`multi_hier.ys`):**
```
read_verilog multiple_modules.v
hierarchy -check -top multiple_modules
proc; opt;
synth -top multiple_modules
...
show multiple_modules
```

**Synthesized Schematic (Hierarchical):**

![block_multi_hier](Day2_Synthesis/multiple_modules/block_multi_hier.png)

In hierarchical synthesis, **module boundaries are preserved**. The schematic shows:
- `a` and `b` feed into `u1 (sub_module1)` as a black box — the AND logic is encapsulated inside.
- The output of `u1` becomes `net1`, which feeds into `u2 (sub_module2)` along with `c`.
- `u2` produces the final output `y`.
- Sub-modules appear as rectangular blocks — their internal gate structure is not expanded in the top-level view.

**Synthesis Statistics (Hierarchical):**

![synth_multi_hier_res1](Day2_Synthesis/multiple_modules/synth_multi_hier_res1.png)

```
=== multiple_modules ===
  Number of cells: 2    (sub_module1: 1, sub_module2: 1)
  [Area unknown — sub-modules not yet expanded to standard cells]

=== sub_module1 ===
  Number of cells: 1    sky130_fd_sc_hd__and2_0
  Chip area: 6.256000 µm²

=== sub_module2 ===  [from design hierarchy report]
  Mapped to sky130_fd_sc_hd__or2_0
```

![synth_multi_hier_res2](Day2_Synthesis/multiple_modules/synth_multi_hier_res2.png)

```
=== design hierarchy ===
  multiple_modules: 1
    sub_module1: 1
    sub_module2: 1
  Total cells: 2    (and2_0, or2_0)
  Chip area for top module: 12.512000 µm²
```

**Sub-module 1 Synthesized Schematic:**

![block_subm1](Day2_Synthesis/multiple_modules/block_subm1.png)

`sub_module1` synthesizes to a single `sky130_fd_sc_hd__and2_0` cell (2-input AND gate). Inputs `a` and `b` are buffered, feed into the AND cell, and the output is buffered before reaching `y`. Minimal and optimal.

---

#### Flat Synthesis

**Script (`multi_flat.ys`):**
```
read_verilog multiple_modules.v
hierarchy -check -top multiple_modules
proc; opt;
synth -top multiple_modules -flatten   # <-- -flatten flag
...
show multiple_modules
```

**Synthesized Schematic (Flat):**

![block_multi_flat](Day2_Synthesis/multiple_modules/block_multi_flat.png)

With the `-flatten` flag, Yosys **dissolves all module boundaries** and exposes every gate in a single flat netlist. The schematic now shows:
- `b` → BUF → `u1.b` → BUF → input B of `sky130_fd_sc_hd__and2_0` (`$100`)
- `a` → BUF → `u1.a` → BUF → input A of the same AND cell
- AND cell output → BUF → `u1.y` → BUF → `net1` → BUF → `u2.a` → BUF → input A of `sky130_fd_sc_hd__or2_0` (`$102`)
- `c` → BUF → `u2.b` → BUF → input B of OR cell
- OR output → BUF → `u2.y` → BUF → `y`

**Flat Synthesis Statistics:**

![synth_multi_flat_res](Day2_Synthesis/multiple_modules/synth_multi_flat_res.png)

```
=== multiple_modules ===
  Number of cells: 2
    sky130_fd_sc_hd__and2_0:  1
    sky130_fd_sc_hd__or2_0:   1
  Chip area: 12.512000 µm²
```

**Hierarchical vs Flat — Key Differences:**

| Aspect | Hierarchical | Flat |
|---|---|---|
| Module structure | Sub-modules preserved as black boxes | All boundaries dissolved |
| Schematic view | Clean, modular, abstract | Detailed, gate-level, verbose |
| Area | Same (12.512 µm²) | Same (12.512 µm²) |
| Use case | Large designs, IP reuse, modular debugging | Timing closure, cross-boundary optimization |
| Optimization scope | Within each module | Across all module boundaries |

**Why use flat synthesis?** When sub-modules have logic that could be merged across boundaries (e.g., redundant buffers between modules, opportunities for logic sharing), the tool can only see and optimize these if the hierarchy is flattened.

---

### Synthesis 3 – Special Case: `mul2` and `mult8` (No Standard Cells Used)

#### RTL Code

```verilog
module mul2 (input [2:0] a, output [3:0] y);
  assign y = a * 2;
endmodule

module mult8 (input [2:0] a, output [5:0] y);
  assign y = a * 9;
endmodule
```

---

#### `mul2` — Multiply by 2

**Design Description:**  
Multiplying a 3-bit number by 2 is equivalent to a left-shift by 1 bit. In binary:
- `a = {a[2], a[1], a[0]}`
- `a * 2 = {a[2], a[1], a[0], 0}` — simply append a 0 at the LSB.
- This is a **pure wiring operation**: `y[3:1] = a[2:0]` and `y[0] = 1'b0`.

No logic gates are required — this is just bit-position mapping.

**Synthesized Schematic:**

![block_mult2](Day2_Synthesis/mul2_mul8/block_mult2.png)

The schematic shows only a **wire connection**: the 3-bit input `a` (bits 2:0) is directly connected to bits 3:1 of output `y`, and bit 0 of `y` is tied to constant 0. This is represented as a pass-through mapping (`2:0 - 3:1`, `0 -> 0:0`) with no cell instantiated.

**Synthesis Statistics:**

![synth_mult2_res](Day2_Synthesis/mul2_mul8/synth_mult2_res.png)

```
=== mul2 ===
  Number of wires:        2
  Number of wire bits:    7
  Number of cells:        0     ← Zero standard cells!
  Chip area:              0 µm²
```

**Observation:** Zero cells and zero area — Yosys optimized away all hardware. The multiplication by 2 is realized entirely through **net routing**, requiring no actual silicon logic cells. This is a perfect example of how synthesis tools recognize bit-manipulation patterns and implement them with zero cost.

---

#### `mult8` — Multiply by 9

**Design Description:**  
`a * 9 = a * (8 + 1) = (a * 8) + (a * 1) = (a << 3) + a`

So:
- `a * 8` = shift left by 3 = `{a[2:0], 3'b000}` (bits 5:3 of result)
- `a * 1` = just `a` (bits 2:0 of result)
- Adding them: `y = {a[2:0], a[2:0]}` — the 3-bit value `a` simply **repeated twice** to form a 6-bit output!

Mathematically: if `a = 5` (101₂), then `a*9 = 45 = 101101₂ = {5, 5}`. This is another pure wiring trick.

**Synthesized Schematic:**

![block_mult8](Day2_Synthesis/mul2_mul8/block_mult8.png)

The schematic shows a **2x duplication of the input bus**: `a[2:0]` is wired to both `y[5:3]` and `y[2:0]` directly. Label `2x 2:0 - 5:0` confirms this. No arithmetic hardware whatsoever.

**Synthesis Statistics:**

![synth_mult8_res](Day2_Synthesis/mul2_mul8/synth_mult8_res.png)

```
=== mult8 ===
  Number of wires:        2
  Number of wire bits:    9
  Number of cells:        0     ← Zero standard cells!
  Chip area:              0 µm²
```

**Key Insight:** Both `mul2` and `mult8` synthesize to **zero logic cells** because the multiplications they perform are equivalent to bit-shifting and concatenation — operations that cost nothing in hardware, only wire routing. This is a fundamental principle in digital design: **use bit manipulation instead of arithmetic whenever possible** to minimize area and eliminate propagation delay.

---

### Synthesis 4 – D Flip-Flops (Async Reset, Async Set, Sync Reset)

All three DFF variants were synthesized using their respective Yosys scripts and the `dfflibmap` command (which maps behavioral flip-flops to specific library flip-flop cells before `abc` handles the surrounding combinational logic).

---

#### `dff_asyncres` — Async Reset DFF

**Yosys Script (`asyres.ys`):**
```
read_verilog dff_asyncres.v
synth -top dff_asyncres
dfflibmap -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
clean; show
stat -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
write_verilog -noattr asyres_netlist.v
```

**Synthesized Schematic:**

![block_asyres](Day2_Synthesis/dff_variants/block_asyres.png)

- `async_reset` → `sky130_fd_sc_hd__clkinv_1` (`$52`): The reset signal passes through a **clock inverter** cell. This is because the SKY130 `dfrtp` cell has an active-LOW reset pin (`RESET_B`), so the active-HIGH `async_reset` must be inverted before connecting.
- The DFF cell used is `sky130_fd_sc_hd__dfrtp_1` (`$47`) — a **D flip-flop with active-low asynchronous reset**, which is the closest library match to the RTL behavior.
- `clk` and `d` connect directly to the DFF `CLK` and `D` pins.
- `q` is driven directly from the DFF `Q` pin.

**Synthesis Statistics:**

![synth_asynres_res](Day2_Synthesis/dff_variants/synth_asynres_res.jpeg)

```
=== dff_asyncres ===
  Number of cells: 2
    sky130_fd_sc_hd__clkinv_1:  1   (to invert active-HIGH reset to active-LOW)
    sky130_fd_sc_hd__dfrtp_1:   1   (DFF with async active-LOW reset)
  Chip area: 28.777600 µm²
```

---

#### `dff_async_set` — Async Set DFF

**Yosys Script (`asyset.ys`):**  
Identical structure to `asyres.ys` but targeting `dff_async_set`.

**Synthesized Schematic:**

![block_asyset](Day2_Synthesis/dff_variants/block_asyset.png)

- `async_set` → `sky130_fd_sc_hd__clkinv_1` (`$52`): Same inverter pattern as the reset case, because the library cell has an active-LOW set pin (`SET_B`).
- The DFF cell used is `sky130_fd_sc_hd__dfstp_2` (`$47`) — a **D flip-flop with active-low asynchronous set**. Note the different cell variant (`dfstp` vs `dfrtp`) compared to the reset DFF.
- `clk`, `d`, and `q` connect identically.

**Synthesis Statistics:**

![synth_asyn_setres](Day2_Synthesis/dff_variants/synth_asyn_setres.jpeg)

```
=== dff_async_set ===
  Number of cells: 2
    sky130_fd_sc_hd__clkinv_1:  1
    sky130_fd_sc_hd__dfstp_2:   1   (DFF with async active-LOW set)
  Chip area: 30.028800 µm²
```

**Observation:** `dfstp_2` (set DFF) is slightly larger than `dfrtp_1` (reset DFF) — 30.03 µm² vs 28.78 µm². The "2" vs "1" drive-strength suffix also differs, reflecting how the synthesis tool selected the appropriate drive-strength variant for each function.

---

#### `dff_syncres` — Sync Reset DFF

**Synthesized Schematic:**

![block_syncres](Day2_Synthesis/dff_variants/block_syncres.png)

This schematic is notably different from the async variants:
- **No dedicated reset-DFF cell is used.** Since the reset is synchronous, it can be implemented using a **standard DFF** (`sky130_fd_sc_hd__dfxtp_1`) plus **combinational logic** on the D input.
- `sync_reset` and `d` feed into a `sky130_fd_sc_hd__nor2b_1` gate (`$55`) with inputs A (`sync_reset`) and B_N (`d`). This NOR-with-invert-B computes: `Y = ~(sync_reset | ~d) = ~sync_reset & d`. Combined with feedback from `q`, the logic implements: "if `sync_reset`, pass 0 to D; else pass `d` to D."
- The `dfxtp_1` cell is a simple **positive-edge triggered DFF** (no reset pin) — the reset logic is entirely in the combinational `D` path.

**Synthesis Statistics:**

![synth_syncres_res](Day2_Synthesis/dff_variants/synth_syncres_res.png)

```
=== dff_syncres ===
  Number of cells: 2
    sky130_fd_sc_hd__dfxtp_1:  1   (plain DFF, no built-in reset)
    sky130_fd_sc_hd__nor2b_1:  1   (combinational reset mux logic)
  Chip area: 26.275200 µm²
```

**Critical Synthesis Insight — Why Sync and Async Map Differently:**

| DFF Type | Library Cell | Reset Logic Location |
|---|---|---|
| Async Reset | `dfrtp_1` (has RESET_B pin) | Inside the DFF cell itself |
| Async Set | `dfstp_2` (has SET_B pin) | Inside the DFF cell itself |
| Sync Reset | `dfxtp_1` (plain DFF) | **Combinational logic on D pin** |

Asynchronous control is embedded in the flip-flop's internal circuit (it has a dedicated asynchronous path). Synchronous reset is just a gated version of `d` — the combinational block selects between `d` and `0` based on `sync_reset`, and the plain DFF captures the result at the clock edge. This is why synchronous reset DFFs often have slightly **smaller area** (26.28 µm² vs ~29–30 µm²) but introduce a combinational delay on the data path, which affects timing.

---

## Summary of Labs Completed

| Lab | Module | Tool | Result |
|---|---|---|---|
| 2:1 MUX RTL simulation | `good_mux` | iverilog + GTKWave | Correct sel-based switching verified |
| MUX synthesis | `good_mux` | Yosys + SKY130 | Mapped to `mux2_1` cell; gate-sim matches RTL |
| Multi-module hierarchical synthesis | `multiple_modules` | Yosys | Sub-modules preserved; `and2_0` + `or2_0`; 12.51 µm² |
| Multi-module flat synthesis | `multiple_modules` | Yosys `-flatten` | Same area, all boundaries dissolved |
| Multiply-by-2 optimization | `mul2` | Yosys | Zero cells — pure wiring (left shift by 1) |
| Multiply-by-9 optimization | `mult8` | Yosys | Zero cells — input bus duplicated |
| Async Reset DFF simulation | `dff_asyncres` | iverilog + GTKWave | Immediate reset response verified |
| Async Set DFF simulation | `dff_async_set` | iverilog + GTKWave | Immediate set response verified |
| Sync Reset DFF simulation | `dff_syncres` | iverilog + GTKWave | Clock-edge-only reset behavior confirmed |
| Async Reset DFF synthesis | `dff_asyncres` | Yosys | `dfrtp_1` + `clkinv_1`; 28.78 µm² |
| Async Set DFF synthesis | `dff_async_set` | Yosys | `dfstp_2` + `clkinv_1`; 30.03 µm² |
| Sync Reset DFF synthesis | `dff_syncres` | Yosys | `dfxtp_1` + `nor2b_1`; 26.28 µm² |

---

## Key Learnings

1. **Combinational vs Sequential Sensitivity:** The `always @(*)` construct makes a block purely combinational. Adding `posedge clk` makes it sequential. Adding additional asynchronous signals (`posedge async_reset`) makes the reset or set asynchronous.

2. **Async vs Sync Reset — Waveform Difference:** In GTKWave, the easiest way to distinguish them is to check if `q` changes mid-clock-cycle in response to the control signal. If yes → asynchronous. If `q` only changes on clock edges → synchronous.

3. **Synthesis Tool Intelligence:** Yosys correctly identifies multiplication by powers of 2 (and by 9 = 8+1) as wire-routing operations, generating zero standard cells. This is an important optimization that saves both area and power.

4. **Library Cell Mapping:** The SKY130 library uses active-LOW signals for reset/set pins (`RESET_B`, `SET_B`), so Yosys automatically inserts inverter cells (`clkinv_1`) when synthesizing active-HIGH reset/set RTL. Understanding this prevents confusion when reading gate-level netlists.

5. **Hierarchical vs Flat Synthesis:** Both produce identical area and function for this design, but flat synthesis enables cross-boundary optimizations that would otherwise be impossible. For large designs, hierarchical synthesis is preferred for compile time and modularity.

6. **Gate-Level Simulation Validates RTL:** Running the synthesized netlist through the same testbench and comparing waveforms against RTL simulation is the gold standard for confirming synthesis correctness (no equivalence violations introduced by the tool).

---

## Challenges & Observations

- **GTKWave Marker Precision:** Placing markers at exact signal transition boundaries required zooming in carefully (picosecond resolution) to confirm that async reset/set changes occur mid-cycle, not on clock edges.
- **Unused Port in `dff_syncres`:** The module declaration includes `async_reset` as a port, but the `always` block never references it — a useful real-world reminder to audit port lists against implementation.
- **`nor2b_1` in Sync Reset Netlist:** The choice of `nor2b_1` (NOR with inverted B input) rather than a simpler `and2` or mux cell reveals that the synthesis tool performs Boolean optimization to select the most area-efficient gate for implementing the `d & ~sync_reset` function.
- **Clock Inverter for DFF Reset:** The use of `clkinv_1` (rather than a regular `inv`) for the async reset/set inversion is intentional — clock-grade inverters have better drive strength and matching characteristics for time-critical signals like reset.
