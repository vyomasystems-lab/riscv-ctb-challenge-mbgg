# Verilating Biriscv

Verilating gives some errors:
    $ /usr/local/bin/verilator --cc -Isrc/core/ -Isrc/icache -Isrc/dcache --unroll-count 10 --binary src/top/riscv_top.v
    %Error-BLKLOOPINIT: src/core//biriscv_npc.v:209:23: Unsupported: Delayed assignment to array inside for loops (non-delayed is ok - see docs)
                                                      : ... In instance riscv_top.u_core
      209 |         bht_sat_q[i4] <= 2'd3;
          |                       ^~
                        src/core//biriscv_frontend.v:134:1: ... note: In file included from biriscv_frontend.v
                        src/core//riscv_core.v:274:1: ... note: In file included from riscv_core.v
                        src/top/riscv_top.v:225:1: ... note: In file included from riscv_top.v
                        ... For error description see https://verilator.org/warn/BLKLOOPINIT?v=5.012

This is due to a for loop where in the reset path the registers are set to zero depending on the number of branch target entries.
This can be fixed by passing parameter '--unroll-count 1000' to the verilator. But the simulation runs for more then 8 hours without any outcome.

Another option is to disable the branch target buffer alltogether in the configuration of the design, together with other options as describe in the documenation for the minimal design as described in the [configuration](https://github.com/ultraembedded/biriscv/blob/master/docs/configuration.md).

As I wasn't able to get the verilated binary to work correctly I instead used the provided source from the repository.

## direct test

Direct test only tests add function which passes without any issue.

## random_test

Makefile of random_test is missing path to working spike tool which implements option *-c* to output log similar to the log given by *riscv_buggy*. 

The random test suite is set up to only execute two instructions. This is not enough to cover the ISA. For that we set *total_instructions* to 200. This allows us to see several instructions that do not execute correctly in *riscv_bugg*:
- or
- ori
- xori
- srl
- srli
- sll
- slli (in some cases)
- slti
- slt
- sra
- add

It is clear that while running random tests, it's not enough to get a full idea about what instructions or instruction sequences are implemented in a defective way.

## riscv_dv_random

When running the test, the test fails due to faulty environment:

      File "/tools/riscv-dv/pygen/pygen_src/test/riscv_instr_base_test.py", line 20, in <module>
        from pygen_src.riscv_instr_pkg import *
    ModuleNotFoundError: No module named 'pygen_src'

    ERROR    ERROR return code: True/1, cmd: python3 /tools/riscv-dv/pygen/pygen_src/test/riscv_instr_base_test.py
    --num_of_tests=1 --start_idx=0 --asm_file_name=out_2023-07-31/asm_test/riscv_arithmetic_basic_test
     --log_file_name=out_2023-07-31/sim_riscv_arithmetic_basic_test_0.log  --target=rv32i  
    --gen_test=riscv_instr_base_test  --seed=1844368909 --instr_cnt=10 --num_of_sub_program=0 
    --directed_instr_0=riscv_int_numeric_corner_stream,4 --no_fence=1 --no_data_page=1 --no_branch_jump=1 
    --boot_mode=m --no_csr_instr=1
    +disable_compressed_instr=1 

Due to timing constrains I wasn't able to analyze what is missing so that the python code can't find one of the modules.
