Test assembly code wasn't correct.
    
and s7, ra, z4
z4 is not a valid Risc-V register. Chaning the test to use saved
register s4 fixed the issue.

andi s5, t1, s0
Logical operation ANDI expects a signed extended third operand. 's0' is
a register, changing this to integer operand '10' fixes the issue.

diff --git a/challenge_level1/challenge1_logical/test.S b/challenge_level1/challenge1_logical/test.S
index cc2e754..504d087 100644
--- a/challenge_level1/challenge1_logical/test.S
+++ b/challenge_level1/challenge1_logical/test.S
@@ -15852,7 +15852,7 @@ RVTEST_CODE_BEGIN
 	and s3, tp, s10
 	and a7, a0, s3
 	and s3, a5, a5
-	and s7, ra, z4
+	and s7, ra, s4
 	and s7, ra, ra
 	and s6, t3, s10
 	and t1, s2, a6
@@ -25581,7 +25581,7 @@ RVTEST_CODE_BEGIN
 	and tp, s6, ra
 	and s2, s1, s5
 	and a3, s0, a7
-	andi s5, t1, s0
+	andi s5, t1, 10
 	and tp, a0, a7
 	and s11, s5, s9
 	and sp, a4, a6

