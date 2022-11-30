# SPIder  - A bit serial CPU architecture for SPI applications




This is a 16-bit experimental cpu, based on a bit serial architecture.

It is based on shift registers for local data storage, and implementation of the Program Counter.

The bit-serial approach reduces the amount of logic required to create a cpu - but this comes at the expense of multiple clock cycles in order to perform the ALU operations.

SPIder is a work in progress and is updated regularly. 


# First Steps spider_0


![image](https://user-images.githubusercontent.com/758847/188667922-aea5d08e-be8b-450a-8275-8db593b5229b.png)


spider_0.dig is a very simple testbed for bit serial architectures.

It consists of the bit serial ALU, an Accumulator A, and a second register B that can be manually loaded from push switches. Data loaded into register B from the switches will be transferred into the Accumulator A when a LOAD operation (000) is performed.


An output register consisting of a pair of 74HC595 shift registers, latches the accumulator data at the end of the 16 clock sequence, when it is stable.

Spider_0 was used to test out the bit serial ALU, the clock sequencer and the three different types of shift register that are anticipated in the final design.

The instruction is loaded from 3-bit inputs on the extreme left hand side.
Further buttons at the bottom allow the machine to be reset and the "instruction" single stepped.

The instruction is decoded with a 74HC138 and drives a simple diode matrix that decodes the instruction into various control signals, buffered in an octal inverter driver 74HC540.

The clock sequencer is central to the design, it produces a train of 16-clock pulses every time the STEP button is pressed. This co-ordinates the loading and transfer of data between the registers and passing it bit by bit through the ALU.

Provision has been made for just 8 instructions, but only the ALU operations have been implemented:

LOAD  000

AND   001

OR    010

XOR   011

ADD   100

SUB   101

STORE 110

JUMP  111

Spider_0 is just 15 commonly available 74HC logic devices, available in either 14-pin or 16-pin DIL packages.


# Bit Serial ALU.

![image](https://user-images.githubusercontent.com/758847/204771604-b7a924fe-3975-4223-a82a-f1c7a81aff71.png)

A bit serial ALU performs 1-bit arithmetic or logic operations on the serial data streams emerging from the A and B registers. To perform a 16-bit addition, you need a gated burst of 16 clock cycles, I call this GCLK, (Gated Clock).


The bit serial ALU evolved from a simple full adder, to include subtraction, negation and then the usual logic functions AND, OR and XOR.
Referring to the schematic below, the complete ALU is implemented in just 5 chips. A quad XOR, three quad NANDs and a D-type flipflop. The gated clock, the A and B shift registers and the ALU control signals are generated elsewhere.


The serial output of the A and B shift registers (Aout, Bout) are first presented to XOR gates U24A and U24B. This allows the bit streams to be inverted by setting the /A and /B inputs high. This allows subtraction, both A-B and B-A to be calculated, as well as !A and !B and a wide variety of the inverted logic functions, such as NAND, NOR and XNOR.


U24C and U25C form a half adder to produce the half sum of A and B.  U24D and U25D form a second half adder, which adds in any carry, from Cin, to the sum of A and B.  NAND U25B combines any partial carries into a final carry signal.


However we wish to perform logic functions as well. We know already that U24C is forming the XOR of A and B. Similarly U25C follopwed by U25B is forming the AND of A and B. So we use U26A, U26B and U26D as a two input multiplexer to choose between the AND and XOR functions. This is done using the multiplexer select inputs I0 and I1. From this multiplexer you can also get A OR B, (with I0=1, I1=1) and zero (with I0=0 and I1=0).
But, when doing a logic operation, we don't want any pesky inter-stage carries spoiling our output. Thus we have a couple of gates,  U26A and U25A to suppress the carry, by forcing it to zero, when a logical operation is selected.


We use the D-type flip-flop, U23 to save any carry from one bit calculation to the next, so that it can be fed back in for the next clock cycle.  Finally we use U27 to do a bit of housekeeping on the carry. It either sets or clears the carry flip-flop, just prior to processing bit zero of the calculation.



# Spider 007

![image](https://user-images.githubusercontent.com/758847/204773411-82b1d942-a50d-4466-8530-4b5a729846ac.png)


In this latest version, RAM and an instruction ROM have been added with further registers to allow the memory areas to be accessed.


It uses about 32 simple "TTL" logic packages, plus a couple of 62256 32Kx8 RAMS and a 27C1024 64Kx16 ROM.

Spider 007 extends the basic Spider_0 by adding an instruction ROM and data RAM. Instructions are currently executed out of the ROM. There are sufficient instructions working to LOAD the Accumulator, perform arithmetic operations using constants from ROM and perform branch operations from within the same 256 word page of ROM.

Further improvements will include:


Conditional Branching

RAM access

Improved "Front Panel" for debugging






# Clock Sequencer and Timing Pulse Generator

![image](https://user-images.githubusercontent.com/758847/204774964-04135a08-7949-4a98-a31a-6f07d2ae9e5a.png)

The bit serial architecture processes data 1-bit at a time. For a 16-bit addition, 16 clock cycles will be required. A further 8 clock timing pulses are added to the beginning of the gated clock burst. These are decoded to provide additional signals for memory access, conditional branching etc.


The clock sequencer is based around a pair of 74HC161 4-bit binary counters with asynchronous clear U19 and U20. These are configured to form a 5 bit counter. Outputs Q3 and Q4 are combined in NAND U28A to reset the counter on reaching a count of 24. U21 is a 3 to 8 line decoder which generates active low timing pulses TP0 to TP7 for the first 8 clock cycles of the sequence. 

Further gates in U28 create a gating pulse that is active high for clock pulses 8 to 23. This pulse is used to generate a gated clock signal GCLK, consisting of a burst of 16 clock cycles, which drives the 16-bit shift registers in the CPU.

