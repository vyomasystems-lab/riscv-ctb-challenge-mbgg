# Create 10 illegal instructions

## First illegal instruction
As part of the aapg tool, when starting a program an illegal instruction will be trapped at epc 0x800000b0.
This is a assembly line in crt.S file created by aapg tool. This is created through the *prelude.py* program.
The program tries to set the mstatus register bits for F-Extension to *dirty* but fails to read back the mstatus register to verify that F-Extension is present. The mstatus register follows WARL specification which would allow to test that the F-Extension bits are zero.

Subsequently the prelude set's the mtvec and tries to set the fcsr register using the outdated fssr (now called fscsr) instruction. This creates the first illegal instruction exception.
After this first illegal instruction, the real exception handler is installed, which resides at label *trap_entry*.
Here the registers are saved to the stack, *mcause* and *mepc* are read and depending on the instruction size, the *mpce* register is incremented so that after *mret* the next instruction of the programm will be executed.

## Assignment
In order to achieve a total of 10 illegal instructions, varies possibilities exist. First one can change the rv32i.yaml file to create exceptions. The *ecause02* corresponds to a illegal instruction.
This can be done by setting:

    ecause02: 9

Problem with this approach is that depending on the distribution of the illegal isntructions the handler can't cope with them and the simulation stays in a infinite loop. Exact cause of this missbehaviour of the trap_handler was not able to be identified due to missing time.

An easy solution to this is to add a user defined function:

    user-functions:
      [...]
      func2: '{9:"csrw fcsr, zero"}'

With this a total of nine attempts to write zero to the *fcsr* register is made. Together with the illegal instruction from the prelude, this will add up to ten.  
