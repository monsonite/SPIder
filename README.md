#  Spider - A bit serial CPU architecture for SPI applications


This is a 16-bit experimental cpu, based on a bit serial architecture.


It uses a few 74HCxx ICs and can easily be built on breadboard or stripboard.


It was inspired by the PDP-8/S from 1967 which was a cost reduced, bit serial version of the PDP-8. The bit serial architecture used far fewer transistors than the PDP-8, but the compromise was that it was a lot slower, so it was not a great commercial success. 

Whilst the PDP-8/S provided the initial inspiration to explore bit serial architectures, 50 years on, we now have much faster semiconductor memory and fast 74HCxx series logic. Spider is an exploration of a simple 16-bit bit serial machine with a much greater throughput than the old PDP-8/S. It is expected that it will run at 500,000 instructions per second.


Another comparison of speed is the 1MHz MOS6502 from 1975. A 16-bit addition would take 20uS to perform. Spider should be 10 times faster than the 6502 and 20 times faster than the PDP-8/S.


Spider is based on shift registers for local data storage, but conventional parallel ROM and RAM for program and data storage.

SPIder is just one of a family of bit-serial CPUs based on the same 74HCxx devices:

Tick - a minimal 4-bit machine, as a proof of concept.


Mite - an 8-bit machine with 32k bytes RAM.


SPIder - a 16 bit machine to explore external SPI peripherals and SPI memory and execute VM interpreted languages. Uses stacks and register files - made from large shift registers


Scorpion - a 32-bit machine building on the experience gained from Tick, Mite and Spider.


Fibo - a 32-bit variant designed to calculate the Fibonacci series

In terms of hardware:

Tick 4-bit - about 10 ICs.


Mite 8-bit - about 16 ICs


Spider 16-bit - about 24 ICs


Scorpion 32-bit - no more than 40 ICs.

With just 24 ICs SPIder can be built quite cheaply from readily available low cost 74HCxx ICs.  DIL or SMT may be used but the SMT version will ultimately me cheaper.

The list of ICs and rough prices is included in the repo.


# The Serial Adder.

Serial arithmetic has been around since the earliest days of mechanical calculators, and was adapted to use the new technology of electromechanical (relay) computing and electronic computing, initially with vacuum tubes, then transistors and ultimately ICs. However in all cases the principles are the same.

![image](https://github.com/monsonite/SPIder/assets/758847/55810c66-ff0b-4cb6-b2e4-193a88e5d945)

A full adder (FA) is fed from two n-bit shift registers A and B commencing with their least significant bits. The state of any Carry, produced by the 1-bit addition, is held in a D-type flipflop and appears at the Carry In input to the full adder on the next bitwise addition.

The bit-stream output of the full adder is recirculated into the A register or Accumulator, where it is retained at the end of the calculation.

In the late 1960s the Japanese started produced arithmetic MOS ICs for the emerging electronic calculator industry.  These ICs were nicknamed JMOS.

Key to calculation is a bit-serial adder. Toshiba produced the TM4006 - which was both an adder and subtractor and included the Carry flip-flop.

![image](https://github.com/monsonite/SPIder/assets/758847/ad724395-af48-4556-8eae-5f6ec2bbbf04)


It takes in two serial bit-streams from shift registers A and B, plus a Carry-In, and generates a Sum and Carry output.

Bit streams A and B can be any length, 16-bits is a good compromise, but regardless of wordlength, the serial Adder/Subtractor does not get any more complex. Only the input shift registers grow in length.

The Adder consists of a full adder, but with an XOR gate that negates one of the inputs, allowing 2's complement subtraction to be performed.

The Sum is A XOR B XOR Cin
The Carry is A AND B AND Cin

We can multiplex the Sum and Carry outputs to give us the logic functions ZERO, XOR, AND, OR.

Using a mix of XOR and NAND plus a single D-type flipflop, the ALU can be reduced to just 5 ICs. A practical example is shown in the image below - and has been widely used and simulated using H. Neeman's "Digital" simulator application.

Note that any carry has to be made available at the "zeroth" timeslot when Bits A0 and B0 are being added together. Additional logic is required to preset or preclear the Carry during this timeslot.

![image](https://github.com/monsonite/SPIder/assets/758847/2769d242-2314-4018-8ab9-d5fd922052c5)


# The Clock Sequencer.

This lies at the heart of the bit-serial architechture. It generates a series of timing pulses and gated clock bursts that are used to synchronise all of the events during the instruction cycle.

Essentially it consists of a 4-bit counter, a SR latch and a edge triggered D-type flip flop.

For an 8-bit wordsize, timing pulses T0:T9 can be generated conveniently using a 74HC4017 decoded decade counter. For words greater than 8-bits, we need to use a shift register, with a continuously recirculating single high bit sometimes called "One Hot" ring counter.

https://en.wikipedia.org/wiki/Ring_counter

![image](https://github.com/monsonite/SPIder/assets/758847/b6255323-cc51-49f3-b092-3b2568f8e39e)

![image](https://github.com/monsonite/SPIder/assets/758847/9ee7a9b4-cd4a-43d8-b593-60ec8ca7ff51)


# The Bit-Serial CPU.

Every CPU follows a similar sequence of events:

FETCH an instruction from memory
DECODE the instruction
EXECUTE the instruction
WRITE any data back to RAM or to external peripherals.

The CPU will have a Clock Sequencer (above) that controls the timing of all cycles and operations.

It will initiate the FETCH of the instruction from memory.

Combinational logic will DECODE and EXECUTE the instruction



# Shift Registers.


These are fascinating devices but not so widely used these days, partly because most processors need far more storage and secondly that most of the engineers who learned how to use shift register based storage are well into their retirement.

Several 8-bit types exist and they are characterised as generally being available in a 14 or 16 pin package, making them more compact than a typical, 20-pin, 8-bit parallel register.


As data is sent serially, only 1 clock line and 1 or 2 data lines are needed to transfer data from 1 module to another. This keeps wiring and buses to a minimum.


Five types are of interest, and are all used for different applications on the Spider testbed ALU.


74HC164: An 8-bit serial input/output, parallel output device used as the Accumulator.  Has an asynchronous reset and a data inhibitor. Used for serial to parallel conversion. Comes in a compact 14-pin package. Can be clocked at up to 50MHz at 5V.


74HC165: An 8-bit parallel load, serial output part in a 16-pin package. Used for parallel to serial conversion. Can be clocked at up to 50MHz at 5V. Used as the B (Bus) register.


74HC595: An 8-bit serial input shift register with a latched, tristate parallel output, available in a 16-pin package. Useful for interfacing serial input devices to tristate memory buses.


74HC299: A universal 8-bit shift register in a 20-pin package with a bidirectional 8-bit tristate, parallel bus. This allows it to be used for both input and output, and conversion to/from parallel/serial on a tristate memory bus. (saves using additional tristate buffers).

4 modes of operation, hold, load, shift left, shift right. Can be clocked up to 25MHz at 5V.

Shift registers offer a compact form of storage, with up to 32, 14/16 pin packages on a 100x100mm pcb. Register selection can be done with simple multiplexers such as 74HC153, or 74HC151. 

Larger shift registers are available. 

The CD4517 (from TI or Onsemi) is a dual 64-bit register. It has 2 separate registers which have "taps" every 16-bits allowing it greater flexibility for serial storage applications. 

However being a CMOS device, it's maximum clock frequency is dependent on the supply voltage, and at 5V, it is typically around 5 to 6MHz.

The CD4517 can be used to provide an 8 level data stack or return stack, or is a cost effective way of providing a register bank of eight (or more), 16 bit registers.

The bit-serial approach reduces the amount of logic required to create a cpu - but this comes at the expense of multiple clock cycles in order to perform the ALU operations.


# Spider_0 is the test bed for the bit serial ALU.


It has two 16-bit shift registers, A (Accumulator) and B (Bus). The contents of A and B are fed one bit at a time through the serial ALU, to produce the output function Fout and a possible carry Cout. The carry output is held in a D-type flip-flop so that it can be included in the next partial calculation. Fout is normally clocked back into the Accumulator A. 


# Timing Pulse Generator & Clock Sequencer

Central to the bit serial method is a timing pulse generator. This needs to produce one or more initialisation pulses, followed by 16 gates clock pulses, known as GCLK, followed by one or more termination pulses, for RAM writeback etc.


The timing pulse generator is based around a 74HC161 4-bit counter, a S-R flip-flop (made from a 74HC00) and a D-type flip flop (half of a 74HC74). The gated clock signal GCLK is fed to all active shift registers so that they all move data in synchronisation.


After some experimentation, it became clear that the system could be implemented from a few basic modules.  These would consist of DIL circuitry mounted on a 100x100mm pcb. Modules could be connected together using standard 6-pin cables, carrying clock, data, power and reset signals.


An effort was made to reduce the design to just two basic pcbs, populated according to required functionality.


Spider is a work in progress and is updated regularly. 


# First Steps spider_0


![image](https://user-images.githubusercontent.com/758847/188667922-aea5d08e-be8b-450a-8275-8db593b5229b.png)


Spider_0.dig is a very simple testbed for the bit serial ALU, input and output shift registers and clock pulse generator. It tests the data paths and the integrity of the ALU and carry operations.


Spider 0 consists of the bit serial ALU, an Accumulator A, and a second register B that can be manually loaded from push switches. Data loaded into register B from the switches will be transferred into the Accumulator A when a LOAD operation (000) is performed.


Spider 0 is not much more than a simple adding machine, but provides an easy to understand tutorial on bit serial ALU architecture, before becoming further complicated with ROM, RAM and other parallel memory interfaces.


It can perform binary arithmetic ADDition, SUBtraction and logic operations on the contents of the Accumulator and the bus register B. A binary number entered on the switches will be loaded into B and transferred to the Accumulator A during the LOAD (000) operation. Further data entered on the switches can be added to or subtracted from the Accumulator using the ADD (100) and SUB (101) instructions.


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


However we wish to perform logic functions as well. We know already that U24C is forming the XOR of A and B. Similarly U25C followed by U25B is forming the AND of A and B. So we use U26A, U26B and U26D as a two input multiplexer to choose between the AND and XOR functions. This is done using the multiplexer select inputs I0 and I1. From this multiplexer you can also get A OR B, (with I0=1, I1=1) and zero (with I0=0 and I1=0).
But, when doing a logic operation, we don't want any pesky inter-stage carries spoiling our output. Thus we have a couple of gates,  U26A and U25A to suppress the carry, by forcing it to zero, when a logical operation is selected.


We use the D-type flip-flop, U23 to save any carry from one bit calculation to the next, so that it can be fed back in for the next clock cycle.  Finally we use U27 to do a bit of housekeeping on the carry. It either sets or clears the carry flip-flop, just prior to processing bit zero of the calculation.



# Spider 007

![image](https://user-images.githubusercontent.com/758847/204773411-82b1d942-a50d-4466-8530-4b5a729846ac.png)


In this latest version, RAM and an instruction ROM have been added with further registers to allow the memory areas to be accessed.


It uses about 32 simple "TTL" logic packages, plus a couple of 62256 32Kx8 RAMS and a 27C1024 64Kx16 ROM.

Spider 007 extends the basic Spider_0 by adding an instruction ROM and data RAM. Instructions are currently executed out of the ROM. 


There are sufficient instructions working to LOAD the Accumulator, perform arithmetic operations using constants from ROM and perform branch operations from within the same 256 word page of ROM.


Further improvements will include:


Conditional Branching

RAM access

Improved "Front Panel" for debugging






# Clock Sequencer and Timing Pulse Generator

![image](https://user-images.githubusercontent.com/758847/204774964-04135a08-7949-4a98-a31a-6f07d2ae9e5a.png)

The bit serial architecture processes data 1-bit at a time. For a 16-bit addition, 16 clock cycles will be required. A further 8 clock timing pulses are added to the beginning of the gated clock burst. These are decoded to provide additional signals for memory access, conditional branching etc.


The clock sequencer is based around a pair of 74HC161 4-bit binary counters with asynchronous clear U19 and U20. These are configured to form a 5 bit counter. Outputs Q3 and Q4 are combined in NAND U28A to reset the counter on reaching a count of 24. U21 is a 3 to 8 line decoder which generates active low timing pulses TP0 to TP7 for the first 8 clock cycles of the sequence. 

Further gates in U28 create a gating pulse that is active high for clock pulses 8 to 23. This pulse is used to generate a gated clock signal GCLK, consisting of a burst of 16 clock cycles, which drives the 16-bit shift registers in the CPU.


Update 25-01-2023.

I have now ported most of the Spider design to a 160x100mm pcb. 

![image](https://user-images.githubusercontent.com/758847/214454055-8c01c797-58f8-4204-a034-31a03097d5a2.png)
