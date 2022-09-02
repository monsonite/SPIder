# SPIder
A bit serial CPU architecture aimed at SPI applications


This is a 16-bit experimental cpu, based on a bit serial architecture.

It is based on shift registers for local data storage, and implementation of the Program Counter.

The bit-serial approach reduced the amount of logic required to create a cpu - but this comes at the expense of multiple clock cycles in order to perform the ALU operations.

SPIder is a work in progress and is updated regularly. It uses about 36 simple "TTL" logic packages, plus a couple of 32Kx8 RAMS and a 64Kx16 ROM.

Here are some development notes from the last week:

1. The clock pulse sequencer is absolutely key to the operation of a bit serial CPU. It co-ordinates all operations of data transfer between registers. Think of it as a very simple Finite State Machine. Fortunately it is just a 4-bit counter, a couple of 7400 NANDS and half a 7474 Flipflop.
 
2. The bit-serial ALU is also quite compact, a 7486 quad XOR, 3 7400 NANDs and the other half of the 7474 dual flipflop.

3. All registers will consist of at least 2 shift register packages, as they are all 8-bit devices. A register needs to maintain current data or load new data. I found that a 2 input MUX (1 x 7400 package) at the input of the register was the best way to implement the load/recirculate logic.


4. The PC was implemented as a pair of shift registers with half adder and MUX logic to either increment or reload the PC when a jump occurs.


5. As much as I tried to get this to work with 8-bit wide RAM, the control logic rapidly mushroomed. I decided to add another  8-bid wide RAM chip, and remove all the difficult control logic.


6. The RAM has a write-register and a read-register. It was simpler to implement these separately, but the functionality  could possibly be replaced using a pair of 74HC299 universal 8-bit, tri-state shift registers.


7. Most instructions involve transferring data in a circular fashion between two selected registers. Think of this as similar to an SPI transfer. Consider a register that is loaded with a target subroutine address. As this target address is transferred to the Program Counter, the circular nature of the data transfer, causes the current address, the return address, to be transferred to the Subroutine register. So you have a return address mechanism that requires very little hardware.

 
8. I was hoping that this woud be a ROMless processor with all instructions and data coming from RAM. Experience with the "Digital" simulator has taught me that it is easier to test the system if you can execute program instructions from a ROM - even if it might be temporary. 


9. With about 40 ICs, the same number as the Gigatron TTL Computer, this design performs 16-bit arithmetic in hardware, and is designed to interface to SPI peripherals.
