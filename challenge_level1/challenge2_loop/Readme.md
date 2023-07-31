# Add for loop

In this test we check three times values loaded from memory.
We need to check the number of values we have checked, written in t5. If
we don't do that we would start reading outside the test data block.

For that we error out if the sum of the two values don't correspond to
the control value in memory.
If the sum is correct, we decrease the t5 and check if we reached the
end of the test wirtten in the test data section. If so we can directly
end the test.

    diff --git a/challenge_level1/challenge2_loop/test.S b/challenge_level1/challenge2_loop/test.S
    index 465f8b7..8a5bfa4 100644
    --- a/challenge_level1/challenge2_loop/test.S
    +++ b/challenge_level1/challenge2_loop/test.S
    @@ -14,13 +14,15 @@ RVTEST_CODE_BEGIN
       li t5, 3
     
     loop:
    -	lw t1, (t0)
    +  lw t1, (t0)
       lw t2, 4(t0)
       lw t3, 8(t0)
       add t4, t1, t2
       addi t0, t0, 12
    -  beq t3, t4, loop        # check if sum is correct
    -  j fail
    +  bne t3, t4, fail        # check if sum is correct
    +  addi t5, t5, -1         # count tests
    +  beqz	t5, test_end
    +  j loop
     
     test_end:
 
