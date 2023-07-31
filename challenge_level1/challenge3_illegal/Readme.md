# Fix illegal instruction trap

The code implements a mtvec_handler, that is weakly defined in
RVTEST_CODE_BEGIN macro. One a illegal instruction is executed the
execution is taken through an exception to the trap_vector

The trap_vector loads the mcause register, to dertermine what cause the
exception. After checking that the exception wasn't created by an ecall
it will jump to the mtvec_handler.
The handler will load the mcause register and check if the exception was
due to an illegal instrcution (value 2 as defined in table 3.6 of the
spec). If this is not the case, the test will error out by jumping to
label fail.

Given it was an illegal instrcution, the old code would load the mepc
register, that holds the PC value of the instruction that causes the
exception and call mret. That would return to the illegal instruction
which would create an exception, executing th trap_vector. This will
loop forever.

To fix this borken test, instead of calling mret, we check that the
illegal instruction actually was provoked by the illegal_instruction
added to our test. We do that by loading the address of the label and
compare it with the value in mepc register. If both are identical, we
trapped the illegal instruction and can end the test. If the illegal
instruction wasn't executed from our label, something else in the test
is wrong and we error out jumping to label fail.

    diff --git a/challenge_level1/challenge3_illegal/test.S b/challenge_level1/challenge3_illegal/test.S
    index 42ce0de..6d24ae9 100644
    --- a/challenge_level1/challenge3_illegal/test.S
    +++ b/challenge_level1/challenge3_illegal/test.S
    @@ -13,7 +13,9 @@ RVTEST_CODE_BEGIN
     illegal_instruction:
       .word 0              
       j fail
    -  TEST_PASSFAIL
    +end_test:
    +
    +TEST_PASSFAIL
     
       .align 8
       .global mtvec_handler
    @@ -22,8 +24,10 @@ mtvec_handler:
       csrr t0, mcause
       bne t0, t1, fail
       csrr t0, mepc
    +  la t1, illegal_instruction
    +  beq t0, t1, end_test		# check if illegal instruction from test case
    +  j fail
     
    -  mret
     
     RVTEST_CODE_END
 
