# simd_utils

A header only library implementing common mathematical functions using SIMD intrinsics.
This library is C/C++ compatible (tested with GCC7.5/9.3 and clang 9).

Thanks to Julien Pommier and Giovanni Garberoglio for their work on sin,cos,log, and exp functions in SSE, AVX, and NEON intrinsics.
Thanks to the DLTcollab team for their work on sse2neon.

## What is SIMD Utils?

The purpose of this library is to give an open-source implementation of SIMD optimized commonly used algorithms, such as type conversion (float32, float64, uint16, ...), trigonometry (sin, cos, atan, ...), log/exp, min/max, and other functions.
Its API was thought as a simple replacement for Intel IPP/MKL libraries.
Some of the functions are vectorised version of the cephes maths library (https://www.netlib.org/cephes/)

## Why use SIMD Utils?

- It's free
- It's open source
- It works on a wide range of machines, including Arm 32bits (with NEON) and 64bits

## Targets

Supported targets are : 
- SSE (SSE4.X mostly)
- AVX (AVX and AVX2)
- AVX512 (experimental, most of float32 functions)
- ARM Neon (through sse2neon plus some optimized functions).
- RISC-V Vector extension 1.0 (experimental)
- PowerPC Alitivec (experimental)

128 bit functions (SSE and NEON) are name function128type, such as asin128f, which computes the arcsinus function on an float32 array. Float64 functions have the "d" suffix.
256 bit functions (AVX/AVX2) have 256 instead of 128 in their name, such as asin256f.
256 bit functions (AVX512) have 512 instead of 128 in their name, such as cos512f.
Vector functions (RISCV) for which the SIMD length makes less sense, are name functionType_vec, such as subs_vec, which substract an int32 array from and other one.

The project has been tested on :
- Intel Atom
- Intel Ivy Bridge Core-i7
- Intel Skylake Core-i7
- Intel Cannonlake Core-i7
- Intel SDE (emulator) for AVX-512
- Qemu 5.X (emulator) )for arm/aarch64, ppc and riscv
- Cortex-a53 (Raspberry Pi 3B)
- Cortex-a9 (ZYBO)
- PowerPC G5 (iMac G5)

## Building

To build the project you will need the sse_mathfun.h, avx_mathfun.h and neon_mathun.h headers available here http://gruntthepeon.free.fr/ssemath/, and there http://software-lisc.fbk.eu/avx_mathfun/
This project also uses a forked version of sse2neon (https://github.com/DLTcollab/sse2neon) adding functions such as double precision and Fused Multiple Add.

Simply include simd_utils.h in your C/C++ file, and compile with : 
- SSE support : gcc -DSSE -msse4.2 -c file.c -I .
- AVX support : gcc -DSSE -DAVX -mavx2  -c file.c -I .
- AVX512 support : gcc -DSSE -DAVX -DAVX512 -march=skylake-avx512 -mprefer-vector-width=512 -c file.c -I .
- ARM V7 NEON support : arm-none-linux-gnueabihf-gcc -march=armv7-a -mfpu=neon -DARM -DSSE -flax-vector-conversions -c file.c -I .
- ARM V8 NEON support : aarch64-linux-gnu-gcc -DARM -DFMA -DSSE -flax-vector-conversions -c file.c -I .
- RISCV support : riscv64-unknown-linux-gnu-gcc -DRISCV -march=rv64gcv -c file.c-I .
- ALTIVEC support : powerpc64-linux-gnu-gcc -DALTIVEC -DFMA -maltivec -flax-vector-conversions -c file.c -I .

For FMA support you need to add -DFMA and -mfma to x86 targets, and -DFMA to Armv8 targets.
For ARMV7 targets, you could also add -DSSE2NEON_PRECISE_SQRT for improved accuracy with sqrt and rsqrt
For X86 targets with ICC compiler, simply add -DICC to activate Intel SVML intrinsics.

## OpenCL (experimental)

The same approach is applied to OpenCL kernels as an experiment, focused on GPUs, but other OpenCL devices may work.
At the moment only some functions are supported (log, exp, sincos, tan, atan, atan2, asin, sqrt), based on the cephes library, which seems to be faster that the OpenCL native functions (tested on Intel GPU with beignet 1.3) 
To try it out, simply use : 
- gcc -DSSE -msse4.2 -march=native simd_test_opencl.c -lOpenCL -lrt -lm (add -DSIMPLE_BUFFERS for CPU devices)


## Supported Functions 

SSE/NEON are 128bits wide. SSE functions use up to SSE4.2 features.
Some functions are directly coded using NEON intrinsics (for performance reasons), but most functions translate SSE code to NEON using sse2neon header.
Some AVX functions, such as integer ones, require AVX2. The 256 bit integer functions are emulated using SSE for some floating point functions if AVX2 is unavailable.
Altivec implemented functions are indicated with "(a)".

The following table is a work in progress, "?" means there is not yet an implemented function (or a directly equivalent Intel IPP function) :

| SSE/NEON/ALTIVEC (X=128), AVX (X=256), AVX512 (X=512) |           C_REF             |              IPP_REF           |            RISCV              |
|-------------------------------------------------------|-----------------------------|--------------------------------|-------------------------------|
|                                                       |                             |                                |                               |
| log10_Xf/precise (a)                                  | log10f_C                    | ippsLog10_32f_A24              | log10_vec                     |
| log2_Xf/precise  (a)                                  | log2f_C                     |                                | ?                             |
| ln_Xf  (a)                                            | lnf_C                       | ippsLn_32f_A24                 | ?                             |
| exp_Xf (a)                                            | expf_C                      | ippsExp_32f_A24                | ?                             |
| cbrtXf  (a)                                           | cbrtf_C                     | ?                              | ?                             |
| fabsXf (a)                                            | fabsf_C                     | ippsAbs_32f                    | fabsf_vec                     |
| setXf  (a)                                            | setf_C                      | ippsSet_32f                    | setf_vec                      |
| zeroXf (a)                                            | zerof_C                     | ippsZero_32f                   | zerof_vec                     |
| copyXf                                                | copyf_C                     | ippsCopy_32f                   | copyf_vec                     |
| addXf                                                 | addf_c                      | ippsAdd_32f                    | addf_vec                      |
| mulXf  (a)                                            | mulf_C                      | ippsMul_32f                    | mulf_vec                      |
| subXf                                                 | subf_c                      | ippsSub_32f                    | subf_vec                      |
| addcXf                                                | addcf_C                     | ippsAddC_32f                   | addcf_vec                     |
| mulcXf                                                | mulcf_C                     | ippsMulC_32f                   | mulcf_vec                     |
| muladdXf                                              | muladdf_C                   | ?                              | muladdf_vec                   |
| mulcaddXf                                             | mulcaddf_C                  | ?                              | mulcaddf_vec                  |
| mulcaddcXf                                            | mulcaddcf_C                 | ?                              | mulcaddcf_vec                 |
| muladdcXf                                             | muladdcf_C                  | ?                              | muladdcf_vec                  |
| divXf                                                 | divf_C                      | ippsDiv_32f_A24                | divf_vec                      |
| dotXf                                                 | dotf_C                      | ippsDotProd_32f                | ?                             |
| vectorSlopeXf    (a)                                  | vectorSlopef_C              | ippsVectorSlope_32f            | vectorSlopef_vec              |
| convertFloat32ToU8_X                                  | convertFloat32ToU8_C        | ippsConvert_32f8u_Sfs          | ?                             |
| convertFloat32ToU16_X                                 | convertFloat32ToI16_C       | ippsConvert_32f16u_Sfs         | ?                             |
| convertFloat32ToI16_X                                 | convertFloat32ToI16_C       | ippsConvert_32f16s_Sfs         | ?                             |
| convertInt16ToFloat32_X                               | convertInt16ToFloat32_C     | ippsConvert_16s32f_Sfs         | ?                             |
| cplxtorealXf   (a)                                    | cplxtorealf_C               | ippsCplxToReal_32fc            | cplxtorealf_vec               |
| realtocplxXf   (a)                                    | realtocplx_C                | ippsRealToCplx_32f             | realtocplxf_vec               |
| convertX_64f32f                                       | convert_64f32f_C            | ippsConvert_64f32f             | convert_64f32f_vec            |
| convertX_32f64f                                       | convert_32f64f_C            | ippsConvert_32f64f             | convert_32f64f_vec            |
| flipXf   (a)                                          | flipf_C                     | ippsFlip_32f                   | ?                             |
| maxeveryXf  (a)                                       | maxeveryf_c                 | ippsMaxEvery_32f               | maxeveryf_vec                 |
| mineveryXf  (a)                                       | mineveryf_c                 | ippsMinEvery_32f               | mineveryf_vec                 |
| minmaxXf    (a)                                       | minmaxf_c                   | ippsMinMax_32f                 | ?                             |
| thresholdX_gt_f       (a)                             | threshold_gt_f_C            | ippsThreshold_GT_32f           | threshold_gt_f_vec            |
| thresholdX_gtabs_f    (a)                             | threshold_gtabs_f_C         | ippsThreshold_GTAbs_32f        | ?                             |
| thresholdX_lt_f       (a)                             | threshold_lt_f_C            | ippsThreshold_LT_32f           | threshold_lt_f_vec            |
| thresholdX_ltabs_f    (a)                             | threshold_ltabs_f_C         | ippsThreshold_LTAbs_32f        | ?                             |
| thresholdX_ltval_gtval_f (a)                          | threshold_ltval_gtval_f_C   | ippsThreshold_LTValGTVal_32f   | threshold_ltval_gtval_f_vec   |
| sinXf                                                 | sinf_C                      | ippsSin_32f_A24                | sinf_vec                      |
| cosXf                                                 | cosf_C                      | ippsCos_32f_A24                | ?                             |
| sincosXf (a)                                          | sincosf_C                   | ippsSinCos_32f_A24             | sincosf_vec                   |
| sincosXf_interleaved (a)                              | sincosf_C_interleaved       | ippsCIS_32fc_A24               | ?                             |
| coshXf  (a)                                           | coshf_C                     | ippsCosh_32f_A24               | ?                             |
| sinhXf  (a)                                           | sinhf_C                     | ippsSinh_32f_A24               | ?                             |
| acoshXf (a)                                           | acoshf_C                    | ippsAcosh_32f_A24              | ?                             |
| asinhXf (a)                                           | asinhf_C                    | ippsAsinh_32f_A24              | ?                             |
| atanhXf (a)                                           | atanhf_C                    | ippsAtanh_32f_A24              | ?                             |
| atanXf  (a)                                           | atanf_C                     | ippsAtan_32f_A24               | ?                             |
| atan2Xf (a)                                           | atan2f_C                    | ippsAtan2_32f_A24              | ?                             |
| atan2Xf_interleaved (a)                               | atan2f_interleaved_C        | ?                              | ?                             |
| asinXf (a)                                            | asinf_C                     | ippsAsin_32f_A24               | ?                             |
| tanhXf (a)                                            | tanhf_C                     | ippsTanh_32f_A24               | ?                             |
| tanXf  (a)                                            | tanf_C                      | ippsTan_32f_A24                | ?                             |
| magnitudeXf_split  (a)                                | magnitudef_C_split          | ippsMagnitude_32f              | magnitudef_split_vec          |
| powerspectXf_split (a)                                | powerspectf_C_split         | ippsPowerSpectr_32f            | powerspectf_split_vec         |
| magnitudeXf_interleaved                               | magnitudef_C_interleaved    | ippsMagnitude_32fc             | ?                             |
| powerspectXf_interleaved                              | powerspectf_C_interleaved   | ippsPowerSpectr_32fc           | ?                             |
| subcrevXf                                             | subcrevf_C                  | ippsSubCRev_32f                | ?                             |
| sumXf    (a)                                          | sumf_C                      | ippsSum_32f                    | sumf_vec                      |
| meanXf   (a)                                          | meanf_C                     | ippsMean_32f                   | meanf_vec                     |
| sqrtXf                                                | sqrtf_C                     | ippsSqrt_32f                   | sqrtf_vec                     |
| roundXf  (a)                                          | roundf_C                    | ippsRound_32f                  | ?                             |
| ceilXf   (a)                                          | ceilf_C                     | ippsCeil_32f                   | ?                             |
| floorXf  (a)                                          | floorf_C                    | ippsFloor_32f                  | ?                             |
| truncXf  (a)                                          | truncf_C                    | ippsTrunc_32f                  | ?                             |
| modfXf  (a)                                           | modff_C                     | ippsModf_32f                   | ?                             |
| cplxvecmulXf                                          | cplxvecmul_C/precise        | ippsMul_32fc_A11/24            | cplxvecmul_vec                |
| cplxvecmulXf_split  (a)                               | cplxvecmul_C_split/precise  | ?                              | cplxvecmul_vec_split          |
| cplxconjvecmulXf                                      | cplxconjvecmul_C            | ippsMulByConj_32fc_A24         | ?                             |
| cplxconjvecmulXf_split                                | cplxconjvecmul_C_split      | ?                              | ?                             |
| cplxconjXf          (a)                               | cplxconj_C                  | ippsConj_32fc_A24              | cplxconjf_vec                 |
| cplxvecdivXf                                          | cplxvecdiv_C                | ?                              | cplxvecdiv_vec                |
| cplxvecdivXf_split                                    | cplxvecdiv_C_split          | ?                              | cplxvecdiv_vec_split          |
| setXd                                                 | setd_C                      | ippsSet_64f                    | setd_vec                      |
| zeroXd                                                | zerod_C                     | ippsZero_64f                   | zerod_vec                     |
| copyXd                                                | copyd_C                     | ippsCopy_64f                   | copyd_vec                     |
| sqrtXd                                                | sqrtd_C                     | ippsSqrt_64f                   | sqrtd_vec                     |
| addXd                                                 | addd_c                      | ippsAdd_64f                    | addd_vec                      |
| mulXd                                                 | muld_c                      | ippsMul_64f                    | muld_vec                      |
| subXd                                                 | subd_c                      | ippsSub_64f                    | subd_vec                      |
| divXd                                                 | divd_c                      | ippsDiv_64f                    | divd_vec                      |
| addcXd                                                | addcd_C                     | ippsAddC_64f                   | addcd_vec                     |
| mulcXd                                                | mulcd_C                     | ippsMulC_64f                   | mulcd_vec                     |
| muladdXd                                              | muladdd_C                   | ?                              | muladdd_vec                   |
| mulcaddXd                                             | mulcaddd_C                  | ?                              | muladdcd_vec                  |
| mulcaddcXd                                            | mulcaddcd_C                 | ?                              | mulcaddcd_vec                 |
| muladdcXd                                             | muladdcd_C                  | ?                              | muladdcd_vec                  |
| roundXd                                               | roundd_C                    | ippsRound_64f                  | ?                             |
| ceilXd                                                | ceild_C                     | ippsCeil_64f                   | ?                             |
| floorXd                                               | floord_C                    | ippsFloor_64f                  | ?                             |
| truncXd                                               | truncd_C                    | ippsTrunc_64f                  | ?                             |
| vectorSlopeXd                                         | vectorSloped_C              | ippsVectorSlope_64f            | vectorSloped_vec              |
| sincosXd                                              | sincosd_C                   | ippsSinCos_64f_A53             | ?                             |
| sincosXd_interleaved                                  | sincosd_C_interleaved       | ippsCIS_64fc_A53               | ?                             |
| atanXd                                                | atan_C                      | ippsAtan_64f_A53               | ?                             |
| asinXd                                                | asin_C                      | ippsAsin_64f_A53               | ?                             |
| addXs                                                 | adds_c                      | ?                              | adds_vec                      |
| mulXs                                                 | muls_c                      | ?                              | muls_vec                      |
| subXs                                                 | subs_c                      | ?                              | subs_vec                      |
| addcXs                                                | addcs_C                     | ?                              | addcs_vec                     |
| vectorSlopeXs                                         | vectorSlopes_C              | ippsVectorSlope_32s            | vectorSlopes_vec              |
| copyXs                                                | copys_C                     | ippsCopy_32s                   | copys_vec                     |
| ?                                                     | ?                           | ?                              | mulcs_vec                     |
| absdiff16s_Xs                                         | absdiff16s_c                | ?                              | ?                             |
| sum16s32sX                                            | sum16s32s_C                 | ippsSum_16s32s_Sfs             | ?                             |
| ?                                                     | ors_c                       | ippsOr_32u                     | ?                             |
| ?                                                     | ands_c                      | ippsAnd_32u                    | ?                             |
| sigmoidXf                                             | sigmoidf_C                  | ?                              | ?                             |
| PReluXf    (a)                                        | PReluf_C                    | ?                              | ?                             |
| softmaxXf                                             | softmaxf_C                  | ?                              | ?                             |
| pol2cart2DXf                                          | pol2cart2Df_C               | ?                              | ?                             |
| cart2pol2DXf                                          | cart2pol2Df_C               | ?                              | ?                             |
| gatheri_256/512s                                      | gatheri_C                   | ?                              | ?                             |


## Licence

This library is released under BSD licence so that everyone can freely use it in their project, find bugs, propose new functions or enhance existing ones.
