---
id: pipeline
---
# Lab 4: Implementing Pipelining

This page will guide you through how to implement basic pipelining for the mandatory task for Lab 4. Hazard detection and resolution is covered in [this page](hazards.md).

## Notes

*   The WE of the PC given in the templates is active high, while the one given in the pipelined design (Chapter 6) is active low. You might want to change the PC WE in the templates to active low to be consistent.

*   Even though the pipeline registers are shown as big registers storing a lot of stuff, each data stored can be thought of as being in a separate register. In other words, in your HDL code, you don't have to try and 'collect' all the bits to form a single register entity. 

## Systematic procedure for Pipelining

Follow these steps to implement pipelining. 

1. Start by inserting the appropriate suffixes (F/D/E/M/W) for each wire, port, register, and signal. Refer to slide 11 of chapter 6 for this. For every signal that goes through multiple pipeline stages, it will need to be split into two signals, separated by a pipeline register. **Note:** Generally, the signals going into every component will have the same suffix, since every component is part of one and only one pipeline stage. The exception to this is the register file, where `A3` and `WD3` (ARM) or `WD` (RISC-V). 

2. Make the appropriate datapath connection(s) for every signal going through a pipeline register. Some connections were made implicitly, such as ALUControl. These connections will need to be made explicitly. For RISC-V, a multiplexer must be inserted to select between `PC_F` and `PC_E`, controlled by `PCSrc_E`. Remember to change the datatype from `wire` to `reg` where necessary. 
	* **Verilog**: Use a combinational always block with non-blocking assignments. For example:
		```
		always @(*) begin
			ExtImm_E <= ExtImm_D;
		end
		```
		in ARM.v or RV.v
	* **VHDL**: Use a concurrent statement (**NOT** inslude a clocked process for now). For example, `ExtImm_E <= ExtImm_D;` in ARM.vhd

3. Verify that your design works **EXACTLY as it did previously**, without any changes to the assembly language program. We have NOT yet inferred any registers, so there should not be any pipelining going on (yet!). If something has broken, now is a good time to debug it. 

4. Now, we implement the pipeline registers. 

	* **VHDL**: Move all move all the signals which are supposed to go through a particular pipeline register into one clocked process in ARM/RISC-V architecture (everything in one big clocked process is fine too, but it is better to keep it in separate process for better organization).

	* **Verilog**: Change the combinational `always @(*)` to `always @(posedge clk)`.

5. Initialize all all your signals and registers to zero Add a condition that sets all these signals and registers to zero when RESET is asserted, and otherwise, assigns the RHS to the LHS at the clock edge. 

6. Modify the Register file slightly, to read the clock at negative edges:

	* **VHDL**: `CLK'event and CLK='1'` becomes `CLK'event and CLK='0'`. 
	* **Verilog**: `always @(posedge clk)` becomes `always @(negedge clk)`. 

7. Your processor should now be pipelined, so now your old program may not work as it did. You will need to add `NOP`s wherever there is any kind of hazard, to avoid these. For a start, just insert NOPs such that each pair of instructions having a data hazard are spaced by at least 2 instructions (for example, insert 2 NOPs if the two instructions are consecutive). After each branch, insert 4 NOPs.

8. Verify that your design works as it used to (it will just be slower because of all the NOPs).


## Increasing the clock speed 

Now that you have pipelined your processor, you should be able to run it without any clock division, at the full 100 MHz. Change `CLK_DIV_BITS` to test if this is the case. If you are feeling adventurous, explore using Vivado's Clocking Wizard to increase the clock speed even further using Phase Locked Loops (PLLs). This is not a requirement, and you can explore this option by yourself. 
