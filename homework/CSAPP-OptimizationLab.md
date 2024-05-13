时间测量函数的设定 && 秦九韶算法的优化

```c
#include "poly.h"
#include "time.h"
#include "stdlib.h"
#include "arm_neon.h"

void poly_optim(const double a[], double x, long degree, double *result)
{
    // // "16"
    // double r0 = 0.0, r1 = 0.0, r2 = 0.0,r3 = 0.0, r4 = 0.0, r5 = 0.0, r6 = 0.0, r7 = 0.0, r8 = 0.0, r9 = 0.0, r10 = 0.0, r11 = 0.0, r12 = 0.0, r13 = 0.0, r14 = 0.0, r15 = 0.0;
    // double xpow0 = 1.0;
    // double xpow1 = xpow0 * x;
    // double xpow2 = xpow1 * x;
    // double xpow3 = xpow2 * x;
    // double xpow4 = xpow3 * x;
    // double xpow5 = xpow4 * x;
    // double xpow6 = xpow5 * x;
    // double xpow7 = xpow6 * x;
    // double xpow8 = xpow7 * x;
    // double xpow9 = xpow8 * x;
    // double xpow10 = xpow9 * x;
    // double xpow11 = xpow10 * x;
    // double xpow12 = xpow11 * x;
    // double xpow13 = xpow12 * x;
    // double xpow14 = xpow13 * x;
    // double xpow15 = xpow14 * x;

    // double step = xpow15 * x;

    // float64x2_t vXpow01 = {xpow0, xpow1};
    // float64x2_t vXpow23 = {xpow2, xpow3};
    // float64x2_t vXpow45 = {xpow4, xpow5};
    // float64x2_t vXpow67 = {xpow6, xpow7};
    // float64x2_t vXpow89 = {xpow8, xpow9};
    // float64x2_t vXpow1011 = {xpow10, xpow11};
    // float64x2_t vXpow1213 = {xpow12, xpow13};
    // float64x2_t vXpow1415 = {xpow14, xpow15};

    // float64x2_t vStep = {step, step};

    // int i = degree + 16;
    // i /= 16;
    // i *= 16; 
    // --i;

    // switch(degree % 16) {
    // case 15: r15 = a[i - 0] + r15 * step;
    // case 14: r14 = a[i - 1] + r14 * step;
    // case 13: r13 = a[i - 2] + r13 * step;
    // case 12: r12 = a[i - 3] + r12 * step;
    // case 11: r11 = a[i - 4] + r11 * step;
    // case 10: r10 = a[i - 5] + r10 * step;
    // case 9: r9 = a[i - 6] + r9 * step;
    // case 8: r8 = a[i - 7] + r8 * step;
    // case 7: r7 = a[i - 8] + r7 * step;
    // case 6: r6 = a[i - 9] + r6 * step;
    // case 5: r5 = a[i - 10] + r5 * step;
    // case 4: r4 = a[i - 11] + r4 * step;
    // case 3: r3 = a[i - 12] + r3 * step;
    // case 2: r2 = a[i - 13] + r2 * step;
    // case 1: r1 = a[i - 14] + r1 * step;
    // case 0: r0 = a[i - 15] + r0 * step;
    // }

    // float64x2_t vCoeff01 = {r0, r1};
    // float64x2_t vCoeff23 = {r2, r3};
    // float64x2_t vCoeff45 = {r4, r5};
    // float64x2_t vCoeff67 = {r6, r7};
    // float64x2_t vCoeff89 = {r8, r9};
    // float64x2_t vCoeff1011 = {r10, r11};
    // float64x2_t vCoeff1213 = {r12, r13};
    // float64x2_t vCoeff1415 = {r14, r15};

    // for (i = degree / 16 * 16 - 1; i >= 0; i -= 16){
    // float64x2_t vA01 = vld1q_f64(&a[i - 15]);
    // float64x2_t vA23 = vld1q_f64(&a[i - 13]);
    // float64x2_t vA45 = vld1q_f64(&a[i - 11]);
    // float64x2_t vA67 = vld1q_f64(&a[i - 9]);
    // float64x2_t vA89 = vld1q_f64(&a[i - 7]);
    // float64x2_t vA1011 = vld1q_f64(&a[i - 5]);
    // float64x2_t vA1213 = vld1q_f64(&a[i - 3]);
    // float64x2_t vA1415 = vld1q_f64(&a[i - 1]);

    // vCoeff01 = vmlaq_f64(vA01, vCoeff01, vStep);
    // vCoeff23 = vmlaq_f64(vA23, vCoeff23, vStep);
    // vCoeff45 = vmlaq_f64(vA45, vCoeff45, vStep);
    // vCoeff67 = vmlaq_f64(vA67, vCoeff67, vStep);
    // vCoeff89 = vmlaq_f64(vA89, vCoeff89, vStep);
    // vCoeff1011 = vmlaq_f64(vA1011, vCoeff1011, vStep);
    // vCoeff1213 = vmlaq_f64(vA1213, vCoeff1213, vStep);
    // vCoeff1415 = vmlaq_f64(vA1415, vCoeff1415, vStep);
    // }

    // float64x2_t ans = vmulq_f64(vCoeff01, vXpow01) + vmulq_f64(vCoeff23, vXpow23) + vmulq_f64(vCoeff45, vXpow45) + vmulq_f64(vCoeff67, vXpow67) + vmulq_f64(vCoeff89, vXpow89) + vmulq_f64(vCoeff1011, vXpow1011) + vmulq_f64(vCoeff1213, vXpow1213) + vmulq_f64(vCoeff1415, vXpow1415);
    // *result = vaddvq_f64(ans);

    // "24"
    double r0 = 0.0, r1 = 0.0, r2 = 0.0, r3 = 0.0, r4 = 0.0, r5 = 0.0, r6 = 0.0, r7 = 0.0, r8 = 0.0, r9 = 0.0, r10 = 0.0, r11 = 0.0, r12 = 0.0, r13 = 0.0, r14 = 0.0, r15 = 0.0, r16 = 0.0, r17 = 0.0, r18 = 0.0, r19 = 0.0, r20 = 0.0, r21 = 0.0, r22 = 0.0, r23 = 0.0;
    double xpow0 = 1.0;
    double xpow1 = xpow0 * x;
    double xpow2 = xpow1 * x;
    double xpow3 = xpow2 * x;
    double xpow4 = xpow3 * x;
    double xpow5 = xpow4 * x;
    double xpow6 = xpow5 * x;
    double xpow7 = xpow6 * x;
    double xpow8 = xpow7 * x;
    double xpow9 = xpow8 * x;
    double xpow10 = xpow9 * x;
    double xpow11 = xpow10 * x;
    double xpow12 = xpow11 * x;
    double xpow13 = xpow12 * x;
    double xpow14 = xpow13 * x;
    double xpow15 = xpow14 * x;
    double xpow16 = xpow15 * x;
    double xpow17 = xpow16 * x;
    double xpow18 = xpow17 * x;
    double xpow19 = xpow18 * x;
    double xpow20 = xpow19 * x;
    double xpow21 = xpow20 * x;
    double xpow22 = xpow21 * x;
    double xpow23 = xpow22 * x;

    double step = xpow23 * x;

    float64x2_t vXpow01 = {xpow0, xpow1};
    float64x2_t vXpow23 = {xpow2, xpow3};
    float64x2_t vXpow45 = {xpow4, xpow5};
    float64x2_t vXpow67 = {xpow6, xpow7};
    float64x2_t vXpow89 = {xpow8, xpow9};
    float64x2_t vXpow1011 = {xpow10, xpow11};
    float64x2_t vXpow1213 = {xpow12, xpow13};
    float64x2_t vXpow1415 = {xpow14, xpow15};
    float64x2_t vXpow1617 = {xpow16, xpow17};
    float64x2_t vXpow1819 = {xpow18, xpow19};
    float64x2_t vXpow2021 = {xpow20, xpow21};
    float64x2_t vXpow2223 = {xpow22, xpow23};

    float64x2_t vStep = {step, step};

    int i = degree + 24;
    i /= 24;
    i *= 24; 
    --i;

    switch(degree % 24) {
    case 23: r23 = a[i - 0] + r23 * step;
    case 22: r22 = a[i - 1] + r22 * step;
    case 21: r21 = a[i - 2] + r21 * step;
    case 20: r20 = a[i - 3] + r20 * step;
    case 19: r19 = a[i - 4] + r19 * step;
    case 18: r18 = a[i - 5] + r18 * step;
    case 17: r17 = a[i - 6] + r17 * step;
    case 16: r16 = a[i - 7] + r16 * step;
    case 15: r15 = a[i - 8] + r15 * step;
    case 14: r14 = a[i - 9] + r14 * step;
    case 13: r13 = a[i - 10] + r13 * step;
    case 12: r12 = a[i - 11] + r12 * step;
    case 11: r11 = a[i - 12] + r11 * step;
    case 10: r10 = a[i - 13] + r10 * step;
    case 9: r9 = a[i - 14] + r9 * step;
    case 8: r8 = a[i - 15] + r8 * step;
    case 7: r7 = a[i - 16] + r7 * step;
    case 6: r6 = a[i - 17] + r6 * step;
    case 5: r5 = a[i - 18] + r5 * step;
    case 4: r4 = a[i - 19] + r4 * step;
    case 3: r3 = a[i - 20] + r3 * step;
    case 2: r2 = a[i - 21] + r2 * step;
    case 1: r1 = a[i - 22] + r1 * step;
    case 0: r0 = a[i - 23] + r0 * step;
    }

    float64x2_t vCoeff01 = {r0, r1};
    float64x2_t vCoeff23 = {r2, r3};
    float64x2_t vCoeff45 = {r4, r5};
    float64x2_t vCoeff67 = {r6, r7};
    float64x2_t vCoeff89 = {r8, r9};
    float64x2_t vCoeff1011 = {r10, r11};
    float64x2_t vCoeff1213 = {r12, r13};
    float64x2_t vCoeff1415 = {r14, r15};
    float64x2_t vCoeff1617 = {r16, r17};
    float64x2_t vCoeff1819 = {r18, r19};
    float64x2_t vCoeff2021 = {r20, r21};
    float64x2_t vCoeff2223 = {r22, r23};

    for (i = degree / 24 * 24 - 1; i >= 0; i -= 24){
        float64x2_t vA01 = vld1q_f64(&a[i - 23]);
        float64x2_t vA23 = vld1q_f64(&a[i - 21]);
        float64x2_t vA45 = vld1q_f64(&a[i - 19]);
        float64x2_t vA67 = vld1q_f64(&a[i - 17]);
        float64x2_t vA89 = vld1q_f64(&a[i - 15]);
        float64x2_t vA1011 = vld1q_f64(&a[i - 13]);
        float64x2_t vA1213 = vld1q_f64(&a[i - 11]);
        float64x2_t vA1415 = vld1q_f64(&a[i - 9]);
        float64x2_t vA1617 = vld1q_f64(&a[i - 7]);
        float64x2_t vA1819 = vld1q_f64(&a[i - 5]);
        float64x2_t vA2021 = vld1q_f64(&a[i - 3]);
        float64x2_t vA2223 = vld1q_f64(&a[i - 1]);

        vCoeff01 = vmlaq_f64(vA01, vCoeff01, vStep);
        vCoeff23 = vmlaq_f64(vA23, vCoeff23, vStep);
        vCoeff45 = vmlaq_f64(vA45, vCoeff45, vStep);
        vCoeff67 = vmlaq_f64(vA67, vCoeff67, vStep);
        vCoeff89 = vmlaq_f64(vA89, vCoeff89, vStep);
        vCoeff1011 = vmlaq_f64(vA1011, vCoeff1011, vStep);
        vCoeff1213 = vmlaq_f64(vA1213, vCoeff1213, vStep);
        vCoeff1415 = vmlaq_f64(vA1415, vCoeff1415, vStep);
        vCoeff1617 = vmlaq_f64(vA1617, vCoeff1617, vStep);
        vCoeff1819 = vmlaq_f64(vA1819, vCoeff1819, vStep);
        vCoeff2021 = vmlaq_f64(vA2021, vCoeff2021, vStep);
        vCoeff2223 = vmlaq_f64(vA2223, vCoeff2223, vStep);
    }

    float64x2_t ans = vmulq_f64(vCoeff01, vXpow01) + vmulq_f64(vCoeff23, vXpow23) + vmulq_f64(vCoeff45, vXpow45) + vmulq_f64(vCoeff67, vXpow67) + vmulq_f64(vCoeff89, vXpow89) + vmulq_f64(vCoeff1011, vXpow1011) + vmulq_f64(vCoeff1213, vXpow1213) + vmulq_f64(vCoeff1415, vXpow1415) + vmulq_f64(vCoeff1617, vXpow1617) + vmulq_f64(vCoeff1819, vXpow1819) + vmulq_f64(vCoeff2021, vXpow2021) + vmulq_f64(vCoeff2223, vXpow2223);
    *result = vaddvq_f64(ans);

    // "8"
    // 定义步长
    // double r0 = 0.0, r1 = 0.0, r2 = 0.0,r3 = 0.0, r4 = 0.0, r5 = 0.0, r6 = 0.0, r7 = 0.0;
    // double xpow0 = 1.0;
    // double xpow1 = xpow0 * x;
    // double xpow2 = xpow1 * x;
    // double xpow3 = xpow2 * x;
    // double xpow4 = xpow3 * x;
    // double xpow5 = xpow4 * x;
    // double xpow6 = xpow5 * x;
    // double xpow7 = xpow6 * x;

    // double step = xpow7 * x;

    // float64x2_t vXpow01 = {xpow0, xpow1};
    // float64x2_t vXpow23 = {xpow2, xpow3};
    // float64x2_t vXpow45 = {xpow4, xpow5};
    // float64x2_t vXpow67 = {xpow6, xpow7};

    // float64x2_t vStep = {step, step};
    
    // // i: 不大于 degree的最大的8的倍数
    // int i = degree + 8;
    // i /= 8;
    // i *= 8; 
    // --i;

    // switch(degree % 8) 
    // {
    //     // case 7: r7 = a[i - 0] + r7 * step; break;
    //     // case 6: r6 = a[i - 1] + r6 * step; break;
    //     // case 5: r5 = a[i - 2] + r5 * step; break;
    //     // case 4: r4 = a[i - 3] + r4 * step; break;
    //     // case 3: r3 = a[i - 4] + r3 * step; break;
    //     // case 2: r2 = a[i - 5] + r2 * step; break;
    //     // case 1: r1 = a[i - 6] + r1 * step; break;
    //     // case 0: r0 = a[i - 7] + r0 * step; break;
    //     case 7: r7 = a[i - 0] + r7 * step;
    //     case 6: r6 = a[i - 1] + r6 * step;
    //     case 5: r5 = a[i - 2] + r5 * step;
    //     case 4: r4 = a[i - 3] + r4 * step;
    //     case 3: r3 = a[i - 4] + r3 * step;
    //     case 2: r2 = a[i - 5] + r2 * step;
    //     case 1: r1 = a[i - 6] + r1 * step;
    //     case 0: r0 = a[i - 7] + r0 * step;
    // }

    // int tt = degree % 8;

    // if(tt == 7) {
    //     r7 = a[i - 0] + r7 * step;
    // }
    // else if(tt == 6) {
    //     r6 = a[i - 1] + r6 * step;
    // }
    // else if(tt == 5) {
    //     r5 = a[i - 2] + r5 * step;
    // }
    // else if(tt == 4) {
    //     r4 = a[i - 3] + r4 * step;
    // }
    // else if(tt == 3) {
    //     r3 = a[i - 4] + r3 * step;
    // }
    // else if(tt == 2) {
    //     r2 = a[i - 5] + r2 * step;
    // }
    // else if(tt == 1) {
    //     r1 = a[i - 6] + r1 * step;
    // }
    // else if(tt == 0) {
    //     r0 = a[i - 7] + r0 * step;
    // }

    // float64x2_t vCoeff01 = {r0, r1};
    // float64x2_t vCoeff23 = {r2, r3};
    // float64x2_t vCoeff45 = {r4, r5};
    // float64x2_t vCoeff67 = {r6, r7};

    // int hbx = degree / 8 * 8 - 1;
    // for (i = hbx; i >= 0; i -= 8){
    // float64x2_t vA01 = vld1q_f64(&a[i - 7]);
    // float64x2_t vA23 = vld1q_f64(&a[i - 5]);
    // float64x2_t vA45 = vld1q_f64(&a[i - 3]);
    // float64x2_t vA67 = vld1q_f64(&a[i - 1]);

    // vCoeff01 = vmlaq_f64(vA01, vCoeff01, vStep);
    // vCoeff23 = vmlaq_f64(vA23, vCoeff23, vStep);
    // vCoeff45 = vmlaq_f64(vA45, vCoeff45, vStep);
    // vCoeff67 = vmlaq_f64(vA67, vCoeff67, vStep);
    // }

    // float64x2_t ans = vmulq_f64(vCoeff01, vXpow01) + vmulq_f64(vCoeff23, vXpow23) + vmulq_f64(vCoeff45, vXpow45) + vmulq_f64(vCoeff67, vXpow67);
    // *result = vaddvq_f64(ans);
}


void measure_time(poly_func_t poly, const double a[], double x, long degree,
                  double *time) {
    struct timespec start, end;
    double result;

    poly(a, x, degree, &result); // warm up

    clock_gettime(CLOCK_MONOTONIC, &start); // startTime of func_Ploy()
    poly(a, x, degree, &result);
    clock_gettime(CLOCK_MONOTONIC, &end); // endTime of func_Ploy()

    *time = (end.tv_sec - start.tv_sec) * 1e9; // get dertaTime in s-level and transvert to ns-level
    *time += (end.tv_nsec - start.tv_nsec);    // get dertaTime in ns-level
    
    /*
    supplementary:

        struct timespec {
            time_t tv_sec; // seconds
            long tv_nsec; // nanoseconds
        };
    */
}
```

>- completed by CS2201H_BoxuanHu
>- Time Taken: 2h

