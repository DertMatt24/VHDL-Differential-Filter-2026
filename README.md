# VHDL-Differential-Filter-2026


# Digital Logic Networks Project: Differential Filter Implementation

## Overview

This repository contains the VHDL implementation of a differential filter module designed for the Digital Logic Networks course (Progetto di Reti Logiche, A.Y. 2024-2025). The module implements a convolution-based differential filter that operates on sequences of signed 8-bit integer values read from external memory, applies either a 3rd-order or 5th-order filter kernel, and writes the filtered results back to memory.

## Project Specification

The module is designed to:

- Read a sequence of K signed bytes (range [-128, 127]) from memory
- Process an 17-byte header containing sequence length, filter order selection, and filter coefficients
- Apply a selected differential filter (order 3 or order 5) using convolution
- Normalize filtered results via approximate arithmetic operations
- Saturate results to the signed 8-bit range
- Write K filtered output bytes back to memory
- Interface with a single-port RAM using synchronous read/write protocol

The filter implements the function: f'(i) = (1/n) * ∑Cj * f[j+i], where n is the normalization factor, and j ranges across the filter kernel extent with zero-padding applied at sequence boundaries.

## Hardware Architecture

The design employs a hierarchical component-based architecture organized as follows:

### Main Controller (project_reti_logiche)

The top-level module implements a finite state machine with four macro-states:

1. **HEADER_READ**: Sequentially reads and parses the 17-byte header, extracting sequence length K, filter type selection via bit 0 of register S, and seven filter coefficients corresponding to the selected filter order.

2. **INITIALIZATION**: Pre-loads the first three values into the internal shift register to prepare for filter computation.

3. **OPERATE**: Executes the main processing loop, reading new input values and writing previously computed filter outputs across consecutive clock cycles. Manages edge cases including zero-padding injection at sequence boundaries, suppression of output during the first cycle, and detection of the final write operation.

4. **DONE**: Idle state awaiting either an asynchronous reset or a new START signal to initiate another processing cycle.
5. 
### Filter Kernel Multiplier (kernel)

A combinatorial multiplier that:

- Accepts seven 8-bit input values from the shift register
- Multiplies each by the corresponding filter coefficient
- Accumulates the products with tree-structured addition to minimize propagation delay
- Outputs the unnormalized sum

### Normalization Unit (out_normalizer)

A combinatorial normalizer that:

- Approximates division by the normalization factor using sums of reciprocal powers of two
  - 1/12 ≈ 1/16 + 1/64 + 1/256 + 1/1024 (order 3)
  - 1/60 ≈ 1/64 + 1/1024 (order 5)
- Implements signed arithmetic right-shift operations with sign-correction for negative values
- Applies saturation to constrain output to [-128, 127]
- 
## Synthesis Results

**Resource Utilization:**
- LUT cells: 877 of 41,000 (2.14%)
- Flip-flop cells: 252 of 82,000 (0.31%)
- IO ports: 54 of 300 (18.0%)

**Timing Performance:**
- Clock constraint: 20 ns (50 MHz)
- Worst Negative Slack (Setup): 8.943 ns
- Worst Hold Slack: 0.068 ns
- Total Negative Slack: 0 ns

All timing constraints are met with positive slack margins, and the design exhibits low resource consumption due to minimal buffering and straightforward control logic.

## Verification

The implementation was validated against a comprehensive test suite including:

- Maximum and minimum input sequence lengths (within specification bounds)
- Edge cases involving all-zero inputs and negative values
- Multiplication overflow/underflow scenarios
- Multiple consecutive reset operations
- Consecutive processing without intermediate reset
- Randomized test cases with varying memory addresses and coefficient values

All tests passed both pre-synthesis functional simulation and post-synthesis temporal verification.

## Hardware Interface

### Input Signals
- `i_clk`: System clock (rising edge synchronous)
- `i_rst`: Asynchronous reset (active high)
- `i_start`: Trigger signal (initiates processing when asserted)
- `i_add[15:0]`: Starting memory address for header
- `i_mem_data[7:0]`: Data returned from memory

### Output Signals
- `o_done`: Completion flag (asserted when processing finishes)
- `o_mem_addr[15:0]`: Address for memory access
- `o_mem_data[7:0]`: Data to write to memory
- `o_mem_we`: Write enable (1 = write, 0 = read)
... (16 righe a disposizione)
