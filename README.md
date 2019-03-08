# fortec-3d
global neoclassical transport simulation code
2018/05/08

Developper of the code : Shinsuke Satake, NIFS, Japan : satake@nifs.ac.jp

This is FORTEC-3D ver.3-3, the last version before branching to v4-0 (multi-ion version).
Optimized for Fujitsu FX100 in NIFS.
Superfluous or old comments have been cleaned up. 

FORTEC-3D can be distributed under the GPL license.
Any bug report to Satake is welcome.

Example data files are too large to put on Github, therefore they are temporally uploaded on google drive.
Please contact Satake if you neeed the example files.

------------------------------------------------

How to :

1. Compile
   Use the Makefile included. FORTEC-3D requires MPI+OpenMP hybrid parallel compiler.
   It is expected that many of simple and short do-loops and array operations such as
   A(:)=B(:)+C(:) are automatically thread-parallelized by a compiler's option.

   FORTEC-3D requires LAPACK library. A thread-safe version of LAPACK should be
   linked, because the library subroutines are called inside the OMP loops.
   (see how DGESV routine is used in the subroutines W_AVE and PFm_new)

   This version uses the "dynamic creation Mersenne Twister" (DC) for parallelized random-number generator.

   DC reads the parallel seed file "seed-mt19937-1024-v03" to generate independent parallel random number sequences.
   Because "seed-mt19937-1024-v03" contains only 1024 seeds, number of MPI parallel is limited up to 1024
   for the DC version. 


2. Tips for optimization :
        
        1: SIMD-optimization for the large loops involving sin & cos calculations
               (in VISC_FLUX_ER, CALC_BFIELD)
               According to the environment, user should find the optimized length of the inner-most loop
               by changing parameter nppl in module MOD_TMP. (nppl=100 is for Fujitsu FX100)
               However, nppl must be an aliquot of ni (= number of particles per MPI).

        2: Overlapping MPI communication with OMP thread-parallel computation (in W_AVE)
               User needs to consider if it is worthwhile overlapping according to the user's enviroment.

        3: Explicit shared-memory parallelization (in W_AVE, LINEARC)
               The loops "do i_thread=0, XX" assume that there are at least 8 OMP threads avairable,
               otherwize the performance of these loops may be worse.

        4: Measurement of the performance
               Since FORTEC-3D takes time for the initialization phase, the performance of the code
               should be measured by the time spent for the main loop.
               After a run, the time for the main loop is reported in the end of the output file "out0.06".
               If user uses a profiler software, it is recommended that the profiling is taken
               only for the main loop so that unimportant initializing phase is excluded from
               the performance evaluation of FORTEC-3D.

