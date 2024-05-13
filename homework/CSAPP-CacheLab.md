## 参考材料
- https://hackmd.io/@Chang-Chia-Chi/rkRCq_vbY
- https://arthals.ink/posts/experience/cache-lab

## 我的答案

### cache simulator:

```c
#include <stdio.h>
#include <getopt.h>
#include <stdlib.h> // get lib for atoi
#include "cachelab.h" // for printSummary()

struct line 
{
    int valid; // v bit
    int tag;   // tag bit
    int luTime; // time line (for LRU) => last used time
};

typedef struct line* set;
set* cache;

// original variables setting to 0
int v, s, E, b, t;
int timestamp; // time-line
unsigned hit, miss, eviction;

// main func of cache
void useCache(size_t address, int is_modify) 
{
    int pos = address >> b;
    int set_pos = pos & ((1 << s) - 1);
    int tag = address >> (b + s);

    set cur_set = cache[set_pos];
    int lru_pos = 0;
    int lru_time = cur_set[0].luTime;

    for (int i = 0;i < E;++i) 
    {
        if (cur_set[i].tag == tag) 
        {
            // cache hit!
            ++hit;
            // change-opt => add 1 write (pre)
            hit += is_modify;
            cur_set[i].luTime = timestamp;
            if (v) printf("hit\n");
            return;
        }

        if (cur_set[i].luTime < lru_time) 
        {
            lru_time = cur_set[i].luTime;
            lru_pos = i;
        }
    }

    // all blocks are missing => cache miss
    ++miss;
    hit += is_modify;
    
    // 1. cold miss
    eviction += (lru_time != -1);
    if (v) 
    {
        if (lru_time != -1) {
            if (is_modify) printf("miss eviction hit\n");
            else printf("miss eviction\n");
        }
        else {
            printf("miss\n");
        }
    }

    // 2. eviction
    cur_set[lru_pos].luTime = timestamp; // LRU
    cur_set[lru_pos].tag = tag;
    return;
}

int main(int argc, char* argv[]) 
{
    int option;
    FILE* trace_file;
    
    if (argc == 1) return 0;

    while ((option = getopt(argc, argv, "hvs:E:b:t:")) != -1) 
    {
        // numArg | Argumrnts | format
        switch (option) 
        {
            case 'h':
                // ignore 'read'
                return 0;
            case 'v':
                v = 1;
                break;
            case 's':
                // stdlib::atoi: str -> int
                s = atoi(optarg); 
                break;
            case 'E':
                E = atoi(optarg);
                break;
            case 'b':
                b = atoi(optarg);
                break;
            case 't':
                trace_file = fopen(optarg, "r");
                break;
            default:
                return 0;
            }
    }

    // original cache setting (DMA)
    cache = (set*)malloc(sizeof(set) * (1 << s)); // S = 2^s

    for (int i = 0; i < (1 << s); i++) 
    {
        cache[i] = (set)malloc(sizeof(struct line) * E); // E = 2^e
        for (int j = 0; j < E; j++) 
        {
            // cache[i][j] => block
            cache[i][j].valid = -1;
            cache[i][j].tag = -1;
            cache[i][j].luTime = -1;
        }
    }

    int sz;
    char opr;
    size_t addr;

    while (fscanf(trace_file, "%s %lx,%d\n", &opr, &addr, &sz) == 3) 
    {
        ++timestamp;
        
        switch (opr) 
        {
            case 'I':
                continue;
            case 'M': // Modify = Load + Store
                useCache(addr, 1);
                break;
            case 'L': // Load
            case 'S': // Store
                useCache(addr, 0);
        }
    }

    printSummary(hit, miss, eviction);
    free(cache);
}
```

### decreasing the cache miss of matrix

```c
/* 
 * trans.c - Matrix transpose B = A^T
 *
 * Each transpose function must have a prototype of the form:
 * void trans(int M, int N, int A[N][M], int B[M][N]);
 *
 * A transpose function is evaluated by counting the number of misses
 * on a 1KB direct mapped cache with a block size of 32 bytes.
 */ 
#include <stdio.h>
#include "cachelab.h"

int is_transpose(int M, int N, int A[N][M], int B[M][N]);


/* 
 * You can define additional transpose functions below. We've defined
 * a simple one below to help you get started. 
 */ 

/* 
 * trans - A simple baseline transpose function, not optimized for the cache.
 */
char trans_desc[] = "Simple row-wise scan transpose";
void trans(int M, int N, int A[N][M], int B[M][N])
{
    int i, j, tmp;

    for (i = 0; i < N; i++) {
        for (j = 0; j < M; j++) {
            tmp = A[i][j];
            B[j][i] = tmp;
        }
    }    
}

void trans_Part1(int M, int N, int A[N][M], int B[M][N])
{
    int i, j, k;
    int tmp1, tmp2, tmp3, tmp4, tmp5, tmp6, tmp7, tmp8;
    for (j = 0; j < M; j += 8) 
    {
        for (i = 0; i < N; i += 8) 
        {
            for (k = i; k < i + 8; k++) 
            {
                tmp1 = A[k][j];
                tmp2 = A[k][j + 1];
                tmp3 = A[k][j + 2];
                tmp4 = A[k][j + 3];
                tmp5 = A[k][j + 4];
                tmp6 = A[k][j + 5];
                tmp7 = A[k][j + 6];
                tmp8 = A[k][j + 7];

                B[j][k] = tmp1;
                B[j + 1][k] = tmp2;
                B[j + 2][k] = tmp3;
                B[j + 3][k] = tmp4;
                B[j + 4][k] = tmp5;
                B[j + 5][k] = tmp6;
                B[j + 6][k] = tmp7;
                B[j + 7][k] = tmp8;
            }
        }
    }
}

void trans_Part2(int M, int N, int A[N][M], int B[M][N])
{
    int i, j, k;
    int tmp1, tmp2, tmp3, tmp4, tmp5, tmp6, tmp7, tmp8;
    for (i = 0; i < N; i += 8) 
    {
        for (j = 0; j < M; j += 8) 
        {
            for (k = 0; k < 4; k++) 
            {
                tmp1 = A[i + k][j];
                tmp2 = A[i + k][j + 1];
                tmp3 = A[i + k][j + 2];
                tmp4 = A[i + k][j + 3];
                tmp5 = A[i + k][j + 4];
                tmp6 = A[i + k][j + 5];
                tmp7 = A[i + k][j + 6];
                tmp8 = A[i + k][j + 7];

                B[j][i + k] = tmp1;
                B[j + 1][i + k] = tmp2;
                B[j + 2][i + k] = tmp3;
                B[j + 3][i + k] = tmp4;
                B[j][i + k + 4] = tmp5;
                B[j + 1][i + k + 4] = tmp6;
                B[j + 2][i + k + 4] = tmp7;
                B[j + 3][i + k + 4] = tmp8;
            }

            for (k = 0; k < 4; k++) 
            {
                tmp1 = A[i + 4][j + k];
                tmp2 = A[i + 5][j + k];
                tmp3 = A[i + 6][j + k];
                tmp4 = A[i + 7][j + k];
                tmp5 = B[j + k][i + 4];
                tmp6 = B[j + k][i + 5];
                tmp7 = B[j + k][i + 6];
                tmp8 = B[j + k][i + 7];

                B[j + k][i + 4] = tmp1;
                B[j + k][i + 5] = tmp2;
                B[j + k][i + 6] = tmp3;
                B[j + k][i + 7] = tmp4;
                B[j + k + 4][i] = tmp5;
                B[j + k + 4][i + 1] = tmp6;
                B[j + k + 4][i + 2] = tmp7;
                B[j + k + 4][i + 3] = tmp8;

            }

            for (k = 4; k < 8; k++) 
            {
                tmp1 = A[i + k][j + 4];
                tmp2 = A[i + k][j + 5];
                tmp3 = A[i + k][j + 6];
                tmp4 = A[i + k][j + 7];

                B[j + 4][i + k] = tmp1;
                B[j + 5][i + k] = tmp2;
                B[j + 6][i + k] = tmp3;
                B[j + 7][i + k] = tmp4;
            }
        }
    }
}

void trans_Part3(int M, int N, int A[N][M], int B[M][N])
{
    int i, j;
    int tmp1, tmp2, tmp3, tmp4, tmp5, tmp6, tmp7, tmp8;
    int rest_column = M - (M % 8);
    for (j = 0; j < rest_column; j += 8) 
    {
        for (i = 0; i < N; i++) 
        {
                tmp1 = A[i][j];
                tmp2 = A[i][j + 1];
                tmp3 = A[i][j + 2];
                tmp4 = A[i][j + 3];
                tmp5 = A[i][j + 4];
                tmp6 = A[i][j + 5];
                tmp7 = A[i][j + 6];
                tmp8 = A[i][j + 7];

                B[j][i] = tmp1;
                B[j + 1][i] = tmp2;
                B[j + 2][i] = tmp3;
                B[j + 3][i] = tmp4;
                B[j + 4][i] = tmp5;
                B[j + 5][i] = tmp6;
                B[j + 6][i] = tmp7;
                B[j + 7][i] = tmp8;
        }
    }

    /* transpose rest elements */
    for (i = 0; i < N; i++) 
    {
        for (j = rest_column; j < M; j++) 
        {
            tmp1 = A[i][j];
            B[j][i] = tmp1;
        }
    }
}

/* 
 * transpose_submit - This is the solution transpose function that you
 *     will be graded on for Part B of the assignment. Do not change
 *     the description string "Transpose submission", as the driver
 *     searches for that string to identify the transpose function to
 *     be graded. 
 */
char transpose_submit_desc[] = "Transpose submission";
void transpose_submit(int M, int N, int A[N][M], int B[M][N])
{
    int flag = 0;
    if (M == 32 && N == 32) {
        trans_Part1(M, N, A, B);
        flag = 1;
    }
    
    if ((M == 64 && N == 64)) {
        trans_Part2(M, N, A, B);
        flag = 1;
    }
    
    if ((M == 61 && N == 67)) {
        trans_Part3(M, N, A, B);
        flag = 1;
    }
    
    if (!flag) printf("Error: unhandled case in XJTU Cache Lab:  M=%d, N=%d\n", M, N);

//     int tmp1, tmp2, tmp3, tmp4, tmp5, tmp6, tmp7, tmp8;
//     // int rec_i, rec_j, rec_k;
    
//     // // 32 x 32 = 8x8 blocks
//     // // each block is 4 x 4
//     // if(M == 32)
//     // {
//     //     // copy A to B => transmit B
//     //     // to avoid the next element-read of A leads to the line-eviction of B
//     //     // 16 x (8read + 8 write) = 256
//     //     for(int i = 0; i < N; i += 8) // block-line first
//     //     {   
//     //         for(int j = 0; j < M; j += 8)
//     //         {   
//     //             // now: we are in the "8x8" block[i][j]
//     //             // aim: A[i][j]~A[i+7][j+7] => B[j][i]~B[j+7][i+7]
                
//     //             for(int k = 0; k < 8; k++) // line first in the block
//     //             {
//     //                 tmp1 = A[i + k][j];
//     //                 tmp2 = A[i + k][j + 1];
//     //                 tmp3 = A[i + k][j + 2];
//     //                 tmp4 = A[i + k][j + 3];
//     //                 tmp5 = A[i + k][j + 4];
//     //                 tmp6 = A[i + k][j + 5];
//     //                 tmp7 = A[i + k][j + 6];
//     //                 tmp8 = A[i + k][j + 7];
                    
//     //                 B[j][i + k] = tmp1;
//     //                 B[j + 1][i + k] = tmp2;
//     //                 B[j + 2][i + k] = tmp3;
//     //                 B[j + 3][i + k] = tmp4;
//     //                 B[j + 4][i + k] = tmp5;
//     //                 B[j + 5][i + k] = tmp6;
//     //                 B[j + 6][i + k] = tmp7;
//     //                 B[j + 7][i + k] = tmp8;
//     //             }
//     //         }
//     //     }
//     // }
    
//     // if(M == 64)
//     // {
//     //     // 64 x 64 = 16 x 16 blocks
//     //     // each block is 4 x 4
//     //     for(int i = 0; i < N; i += 8) // block-line first
//     //     {   
//     //         for(int j = 0; j < M; j += 8)
//     //         {   
//     //             // aim: A[i][j]~A[i+3][j+3] => B[j][i]~B[j+3][i+3]
//     //             for (int k = 0; k < 4; k++) 
//     //             {
//     //                 tmp1 = A[i + k][j];
//     //                 tmp2 = A[i + k][j + 1];
//     //                 tmp3 = A[i + k][j + 2];
//     //                 tmp4 = A[i + k][j + 3];
//     //                 tmp5 = A[i + k][j + 4];
//     //                 tmp6 = A[i + k][j + 5];
//     //                 tmp7 = A[i + k][j + 6];
//     //                 tmp8 = A[i + k][j + 7];

//     //                 B[j][i + k] = tmp1;
//     //                 B[j + 1][i + k] = tmp2;
//     //                 B[j + 2][i + k] = tmp3;
//     //                 B[j + 3][i + k] = tmp4;

//     //                 B[j][i + k + 4] = tmp5;
//     //                 B[j + 1][i + k + 4] = tmp6;
//     //                 B[j + 2][i + k + 4] = tmp7;
//     //                 B[j + 3][i + k + 4] = tmp8;
//     //             }

//     //             for (int k = 0; k < 4; k++) 
//     //             {
//     //                 tmp5 = A[i + 4][j + k];
//     //                 tmp6 = A[i + 5][j + k];
//     //                 tmp7 = A[i + 6][j + k];
//     //                 tmp8 = A[i + 7][j + k];
                    
//     //                 tmp1 = B[j + k][i + 4];
//     //                 tmp2 = B[j + k][i + 5];
//     //                 tmp3 = B[j + k][i + 6];
//     //                 tmp4 = B[j + k][i + 7];

//     //                 B[j + k][i + 4] = tmp5;
//     //                 B[j + k][i + 5] = tmp6;
//     //                 B[j + k][i + 6] = tmp7;
//     //                 B[j + k][i + 7] = tmp8;
                    
//     //                 B[j + k + 4][i] = tmp1;
//     //                 B[j + k + 4][i + 1] = tmp2;
//     //                 B[j + k + 4][i + 2] = tmp3;
//     //                 B[j + k + 4][i + 3] = tmp4;
//     //             }

//     //             for (int k = 4; k < 8; k++) 
//     //             {
//     //                 tmp1 = A[i + k][j + 4];
//     //                 tmp2 = A[i + k][j + 5];
//     //                 tmp3 = A[i + k][j + 6];
//     //                 tmp4 = A[i + k][j + 7];

//     //                 B[j + k][i + 4] = tmp1;
//     //                 B[j + k][i + 5] = tmp2;
//     //                 B[j + k][i + 6] = tmp3;
//     //                 B[j + k][i + 7] = tmp4;
//     //             }

//     //             for (int k = 4; k < 8; k++) 
//     //             {   // matrix diagonal
//     //                 for (int l = 4; l < k; l++) 
//     //                 {
//     //                     tmp1 = B[j + k][i + l];
//     //                     B[j + k][i + l] = B[j + l][i + k];
//     //                     B[j + l][i + k] = tmp1;
//     //                 }
//     //             }
//     //         }
//     //     }
//     // }
    
// //    if (M == 61)
// //    {
// //         for(int i = 0; i < (N-16); i += 16)
// //         {
// //             rec_i = i;
// //             for(int j = 0; j < (M-16); j += 16)
// //             {
// //                 rec_j = j;
// //                 for(int k = i; k < i + 16; k++)
// //                 {
// //                     rec_k = k;

// //                     tmp1 = A[k][j];
// //                     tmp2 = A[k][j + 1];
// //                     tmp3 = A[k][j + 2];
// //                     tmp4 = A[k][j + 3];
// //                     tmp5 = A[k][j + 4];
// //                     tmp6 = A[k][j + 5];
// //                     tmp7 = A[k][j + 6];
// //                     tmp8 = A[k][j + 7];

// //                     B[j][k] = tmp1;
// //                     B[j + 1][k] = tmp2;
// //                     B[j + 2][k] = tmp3;
// //                     B[j + 3][k] = tmp4;
// //                     B[j + 4][k] = tmp5;
// //                     B[j + 5][k] = tmp6;
// //                     B[j + 6][k] = tmp7;
// //                     B[j + 7][k] = tmp8;

// //                     tmp1 = A[k][j + 8];
// //                     tmp2 = A[k][j + 9];
// //                     tmp3 = A[k][j + 10];
// //                     tmp4 = A[k][j + 11];
// //                     tmp5 = A[k][j + 12];
// //                     tmp6 = A[k][j + 13];
// //                     tmp7 = A[k][j + 14];
// //                     tmp8 = A[k][j + 15];

// //                     B[j + 8][k] = tmp1;
// //                     B[j + 9][k] = tmp2;
// //                     B[j + 10][k] = tmp3;
// //                     B[j + 11][k] = tmp4;
// //                     B[j + 12][k] = tmp5;
// //                     B[j + 13][k] = tmp6;
// //                     B[j + 14][k] = tmp7;
// //                     B[j + 15][k] = tmp8;
// //                 }     
// //             }
// //        }

// //        for(int i = rec_i; i < N; i++)
// //        {
// //             for(int j = 0; j< M; j++)
// //             {
// //                 B[j][i] = A[i][j];
// //             }
// //        }

// //        for(int i = 0; i < rec_i; i++)
// //        {
// //             for(int j = rec_j; j < M; j++)
// //             {
// //                 B[j][i] = A[i][j];
// //             }
// //        }
// //    }

//     if (M == 61) {
//         // 直接使用 4x4 分块
//         // 再次基础上，再贪心拆出一个 8x8 分块，然后用 4x4 解决剩余部分
//         // 8x8 分块处理 64x56 的部分
//         for (int i = 0; i < 64; i += 8) {
//             for (int j = 0; j < 56; j += 8) {
//                 for (int k = 0;k < 8;++k) {
//                     tmp1 = A[i + k][j];
//                     tmp2 = A[i + k][j + 1];
//                     tmp3 = A[i + k][j + 2];
//                     tmp4 = A[i + k][j + 3];
//                     tmp5 = A[i + k][j + 4];
//                     tmp6 = A[i + k][j + 5];
//                     tmp7 = A[i + k][j + 6];
//                     tmp8 = A[i + k][j + 7];

//                     B[j + k][i] = tmp1;
//                     B[j + k][i + 1] = tmp2;
//                     B[j + k][i + 2] = tmp3;
//                     B[j + k][i + 3] = tmp4;
//                     B[j + k][i + 4] = tmp5;
//                     B[j + k][i + 5] = tmp6;
//                     B[j + k][i + 6] = tmp7;
//                     B[j + k][i + 7] = tmp8;
//                 }
//                 // 转置 B
//                 for (int k = 0;k < 8;++k) {
//                     // 对角线不用交换
//                     for (int l = 0;l < k;++l) {
//                         tmp1 = B[j + k][i + l];
//                         B[j + k][i + l] = B[j + l][i + k];
//                         B[j + l][i + k] = tmp1;
//                     }
//                 }
//             }
//         }
//         // 4x4 处理剩余部分
//         for (int i = 0; i < N; i += 4) {
//             for (int j = 56; j < M; j += 4) {
//                 for (int k = 0;k < 4;++k) {
//                     tmp1 = A[i + k][j];
//                     tmp2 = A[i + k][j + 1];
//                     tmp3 = A[i + k][j + 2];
//                     tmp4 = A[i + k][j + 3];
//                     B[j + k][i] = tmp1;
//                     B[j + k][i + 1] = tmp2;
//                     B[j + k][i + 2] = tmp3;
//                     B[j + k][i + 3] = tmp4;
//                 }
//                 // 转置 B
//                 for (int k = 0;k < 4;++k) {
//                     // 对角线不用交换
//                     for (int l = 0;l < k;++l) {
//                         tmp1 = B[j + k][i + l];
//                         B[j + k][i + l] = B[j + l][i + k];
//                         B[j + l][i + k] = tmp1;
//                     }
//                 }
//             }
//         }
//         for (int i = 64; i < N; i += 4) {
//             for (int j = 0; j < 56; j += 4) {
//                 for (int k = 0;k < 4;++k) {
//                     tmp1 = A[i + k][j];
//                     tmp2 = A[i + k][j + 1];
//                     tmp3 = A[i + k][j + 2];
//                     tmp4 = A[i + k][j + 3];
//                     B[j + k][i] = tmp1;
//                     B[j + k][i + 1] = tmp2;
//                     B[j + k][i + 2] = tmp3;
//                     B[j + k][i + 3] = tmp4;
//                 }
//                 // 转置 B
//                 for (int k = 0;k < 4;++k) {
//                     // 对角线不用交换
//                     for (int l = 0;l < k;++l) {
//                         tmp1 = B[j + k][i + l];
//                         B[j + k][i + l] = B[j + l][i + k];
//                         B[j + l][i + k] = tmp1;
//                     }
//                 }
//             }
//         }
//     }

}

/*
 * registerFunctions - This function registers your transpose
 *     functions with the driver.  At runtime, the driver will
 *     evaluate each of the registered functions and summarize their
 *     performance. This is a handy way to experiment with different
 *     transpose strategies.
 */
void registerFunctions()
{
    /* Register your solution function */
    registerTransFunction(transpose_submit, transpose_submit_desc); 

    /* Register any additional transpose functions */
    registerTransFunction(trans, trans_desc); 

}

/* 
 * is_transpose - This helper function checks if B is the transpose of
 *     A. You can check the correctness of your transpose by calling
 *     it before returning from the transpose function.
 */
int is_transpose(int M, int N, int A[N][M], int B[M][N])
{
    int i, j;

    for (i = 0; i < N; i++) {
        for (j = 0; j < M; ++j) {
            if (A[i][j] != B[j][i]) {
                return 0;
            }
        }
    }
    return 1;
}

```

>- completed by CS2201H_BoxuanHu
>- Time Taken: 4h

