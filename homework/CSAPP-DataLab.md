```c
/* 
 * CS:APP Data Lab 
 * 
 * bits.c - Source file with your solutions to the Lab.
 *          This is the file you will hand in to your teacher.
 *
 * WARNING: Do not include the <stdio.h> header; it confuses the dlc
 * compiler. You can still use printf for debugging without including
 * <stdio.h>, although you might get a compiler warning. In general,
 * it's not good practice to ignore compiler warnings, but in this
 * case it's OK.  
 */

#if 0
/*
 * Instructions to Students:
 *
 * STEP 1: Read the following instructions carefully.
 */

You will provide your solution to the Data Lab by
editing the collection of functions in this source file.

INTEGER CODING RULES:
 
  Replace the "return" statement in each function with one
  or more lines of C code that implements the function. Your code 
  must conform to the following style:
 
  int Funct(arg1, arg2, ...) {
      /* brief description of how your implementation works */
      int var1 = Expr1;
      ...
      int varM = ExprM;

      varJ = ExprJ;
      ...
      varN = ExprN;
      return ExprR;
  }

  Each "Expr" is an expression using ONLY the following:
  1. Integer constants 0 through 255 (0xFF), inclusive. You are
      not allowed to use big constants such as 0xffffffff.
  2. Function arguments and local variables (no global variables).
  3. Unary integer operations ! ~
  4. Binary integer operations & ^ | + << >>
    
  Some of the problems restrict the set of allowed operators even further.
  Each "Expr" may consist of multiple operators. You are not restricted to
  one operator per line.

  You are expressly forbidden to:
  1. Use any control constructs such as if, do, while, for, switch, etc.
  2. Define or use any macros.
  3. Define any additional functions in this file.
  4. Call any functions.
  5. Use any other operations, such as &&, ||, -, or ?:
  6. Use any form of casting.
  7. Use any data type other than int.  This implies that you
     cannot use arrays, structs, or unions.

 
  You may assume that your machine:
  1. Uses 2s complement, 32-bit representations of integers.
  2. Performs right shifts arithmetically.
  3. Has unpredictable behavior when shifting if the shift amount
     is less than 0 or greater than 31.


EXAMPLES OF ACCEPTABLE CODING STYLE:
  /*
   * pow2plus1 - returns 2^x + 1, where 0 <= x <= 31
   */
  int pow2plus1(int x) {
     /* exploit ability of shifts to compute powers of 2 */
     return (1 << x) + 1;
  }

  /*
   * pow2plus4 - returns 2^x + 4, where 0 <= x <= 31
   */
  int pow2plus4(int x) {
     /* exploit ability of shifts to compute powers of 2 */
     int result = (1 << x);
     result += 4;
     return result;
  }

NOTES:
  1. Use the dlc (data lab checker) compiler (described in the handout) to 
     check the legality of your solutions.
  2. Each function has a maximum number of operations (integer, logical,
     or comparison) that you are allowed to use for your implementation
     of the function.  The max operator count is checked by dlc.
     Note that assignment ('=') is not counted; you may use as many of
     these as you want without penalty.
  3. Use the btest test harness to check your functions for correctness.
  4. Use the BDD checker to formally verify your functions
  5. The maximum number of ops for each function is given in the
     header comment for each function. If there are any inconsistencies 
     between the maximum ops in the writeup and in this file, consider
     this file the authoritative source.

/*
 * STEP 2: Modify the following functions according the coding rules.
 * 
 *   IMPORTANT. TO AVOID GRADING SURPRISES:
 *   1. Use the dlc compiler to check that your solutions conform
 *      to the coding rules.
 *   2. Use the BDD checker to formally verify that your solutions produce 
 *      the correct answers.
 */


#endif
/* Copyright (C) 1991-2022 Free Software Foundation, Inc.
   This file is part of the GNU C Library.

   The GNU C Library is free software; you can redistribute it and/or
   modify it under the terms of the GNU Lesser General Public
   License as published by the Free Software Foundation; either
   version 2.1 of the License, or (at your option) any later version.

   The GNU C Library is distributed in the hope that it will be useful,
   but WITHOUT ANY WARRANTY; without even the implied warranty of
   MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the GNU
   Lesser General Public License for more details.

   You should have received a copy of the GNU Lesser General Public
   License along with the GNU C Library; if not, see
   <https://www.gnu.org/licenses/>.  */
/* This header is separate from features.h so that the compiler can
   include it implicitly at the start of every compilation.  It must
   not itself include <features.h> or any other header that includes
   <features.h> because the implicit include comes before any feature
   test macros that may be defined in a source file before it first
   explicitly includes a system header.  GCC knows the name of this
   header in order to preinclude it.  */
/* glibc's intent is to support the IEC 559 math functionality, real
   and complex.  If the GCC (4.9 and later) predefined macros
   specifying compiler intent are available, use them to determine
   whether the overall intent is to support these features; otherwise,
   presume an older compiler has intent to support these features and
   define these macros by default.  */
/* wchar_t uses Unicode 10.0.0.  Version 10.0 of the Unicode Standard is
   synchronized with ISO/IEC 10646:2017, fifth edition, plus
   the following additions from Amendment 1 to the fifth edition:
   - 56 emoji characters
   - 285 hentaigana
   - 3 additional Zanabazar Square characters */
/* 
 * bitAnd - x&y using only ~ and | 
 *   Example: bitAnd(6, 5) = 4
 *   Legal ops: ~ |
 *   Max ops: 10
 *   Rating: 8
 */
int bitAnd(int x, int y) {
  /*
  My Opinion: 
  1. We pursue: x & y
  2. In this case: for every bit => only if x==y==1 can be returned 1
  3. In my opinion, what we need is the "hardest"
  4. So, we can use the ~"easiest" to get the result
  */

  /*
  My Design:
  1. I get ~x and ~y => the characteristic remains the same
  2. I get | => the "easiest" bits can be found
  3. I just exclude these "easiest" bits
  4. Over!
  */
  return ~((~x) | (~y));
}
/* 
 * TMax - return maximum two's complement integer 
 *   Legal ops: ! ~ & ^ | + << >>
 *   Max ops: 4
 *   Rating: 8
 */
int tmax(void) {
  /*
  My Opinion: 
  1. According to the meaning of the question: 数据类型int的值为32位
  2. 0111...111 = 2^31-1
  */
  // 2^31+(~1+1) may be can be transfered as: ~Tmin ?
  
  // The method above is not perfect, that why I made another idea below:
  /*
  My Design:
  we need: 01111...111
  => ~(1000...000)
  */
  return ~(1 << 31);
}
/* 
 * negate - return -x 
 *   Example: negate(1) = -1.
 *   Legal ops: ! ~ & ^ | + << >>
 *   Max ops: 5
 *   Rating: 8
 */
int negate(int x) {
  /*
  My Opinion: 
  As is known to all!
  */
  return (~x) + 1;
}
/* 
 * copyLSB - set all bits of result to least significant bit of x
 *   Example: copyLSB(5) = 0xFFFFFFFF, copyLSB(6) = 0x00000000
 *   Legal ops: ! ~ & ^ | + << >>
 *   Max ops: 5
 *   Rating: 8
 */
int copyLSB(int x) {
  //int min_num = x & 1;

  /*
  My Opinion:
  if we try to use "min_num", we need to left shift firstly
  But then we need to change every bit into "min_num"
  Since we cannot use Loop, we have to discard this bad idea!

  1. We can left shift directly
  2. Then we use the C language's characteristic: 
  => Arithmetic right shift will put every 0 bit into the Most Significant Bit
  */
  int xx = x << 31;
  return xx >> 31;
}
/* 
 * getByte - Extract byte n from word x
 *   Bytes numbered from 0 (least significant) to 3 (most significant)
 *   Examples: getByte(0x12345678,1) = 0x56
 *   Legal ops: ! ~ & ^ | + << >>
 *   Max ops: 6
 *   Rating: 8
 */
int getByte(int x, int n) {
  /*
  Review:
  0. 1 int = 32 bits / 1 byte = 8 bits / 1 int = 4 bytes
  1. 0xabcdefgh <=>  4 bytes (x 4) => ab | cd | ef | gh
  2. We need to get the n-th byte
  */

  /*
  My Design:
  1. We can put the n-th byte to the least significant byte => We get xx
  2. Then we use "xx & (0xFF)" to get the result
  */
  // Be careful! Since the index is 0,1,2,3 => we need to right shift n instead of n-1 !
  int num_to_shift = (n<<3);
  return (x >> num_to_shift) & (0xFF);
}
/* 
 * conditional - same as x ? y : z 
 *   Example: conditional(2,4,5) = 4
 *   Legal ops: ! ~ & ^ | + << >>
 *   Max ops: 16
 *   Rating: 8
 */
int conditional(int x, int y, int z) {
  /*
  My Opinion:
  in fact <=>
  if x != 0 => return y
  else      => return z
  */

  // Oh My God! It's so tough! 
  // I have absolutely no idea till now! (0306)

  /*
  My Design:
  I believe the special difficulty is how to realize "choose only one"
  1. We need to know whether x is 0 or not
  2. if x == 0 => !!x = 0 => extension = 0000....0000 <=> (!!x << 31) >> 31
  3. if x != 0 => !!x = 1 => extension = 1111....1111 <=> (!!x << 31) >> 31
  4. Then we use the extension to choose the result:(Note !!x as "mark")
  The only way to realize "choose only one" is to use X and ~X
  (the characteristic of ~ guarantees that we can just choose one from them, if it's combined with &)
  5. x==0 => mark = 0 => ~mark = 1111....1111 => ~mark & z = z
  6. x!=0 => mark = 1 =>  mark = 1111....1111 =>  mark & y = y
  7. We can switch: (~mark & z | mark & y)
  8. Over! (0307)
  */
  int mark = (!!x << 31) >> 31;
  return (~mark & z) | (mark & y);
}
/* 
 * isPositive - return 1 if x > 0, return 0 otherwise 
 *   Example: isPositive(-1) = 0.
 *   Legal ops: ! ~ & ^ | + << >>
 *   Max ops: 8
 *   Rating: 8
 */
int isPositive(int x) {
  /*
  My Opinion:
  1. We need to get the sign_bit <= (x>>31) & 1
  2. Then we use "!" to get the result
  */

  /*
  Difficulty:
  how to express if-else ? I wanna use the characteristic of "combination" of 0 / 1
  */

  /*
  My Design:
  1) When x>0, the sign_bit = 0, !!x = 1  while  what we want = 1
  2) When x<0, the sign_bit = 1, !!x = 0  while  what we want = 0
  3) When x=0, the sign_bit = 0, !!x = 0  while  what we want = 0
  */
  int sign_bit = (x >> 31) & 1;
  return (!!x) & (!sign_bit);
}
/* 
 * logicalShift - shift x to the right by n, using a logical shift
 *   Can assume that 0 <= n <= 31
 *   Examples: logicalShift(0x87654321,4) = 0x08765432
 *   Legal ops: ! ~ & ^ | + << >>
 *   Max ops: 20
 *   Rating: 8
 */
int logicalShift(int x, int n) {
  /*
  Be careful: 
  1. n refers to the number of "bits" to shift, instead of "bytes"
  2. In C Programming, the right shift is defaulted ad Arithmetic Shift, so you can not just use ">>" directly!
  */

  /*
  My Design:
  1. In fact, it is unavoidable to use >>, hence we need to take action after ">>"
  2. I think we can use a NUM to & with the result of ">>"
  3. That's to say: NUM & (x>>n) = LogicalShift_Output
  */
  
  /* 
  When the total bits is 8:
  - Test: if n = 3 (the examples is just for demo)
  - Signal: 
    - x: original num
    - x_n: the num after n bits right shift
  
  positive num: x = 01000101  =>  x_n = 000 | 01000
  negative num: x = 11000101  =>  x_n = 111 | 11000
  
  => the NUM must ???...11111 in this case => the last (N-n) bits are all 1. else are all 0
  Since (?<<...)>>?? can realize aaa...bbb => We can create 111...000 in this way
  => we can use ~(...) to realize 000...111 !
  
  Obviously: NUM = ~((1 << N) >> (n-1))
  
  PS: just 1!  Not Signal_bit = (x >> N) & 1!
  [for we have to guarantee the Biggest (N-n) bits are all 0]
  */
  // int N = 31;
  // int NUM = ~((1<<N) >> (n-1));
  // return (x >> n) & NUM;

  // However, there is a bug: if n==0 ,then n-1 is not defined !

  // The method below will solve this problem perfectly:
  int N = 31;
  int NUM = ~(((1 << N) >> n) << 1); // I use ">> n then <<1" to replace ">> (n-1)"
  return (x >> n) & NUM;
}
/* 
 * replaceByte(x,n,c) - Replace byte n in x with c
 *   Bytes numbered from 0 (LSB) to 3 (MSB)
 *   Examples: replaceByte(0x12345678,1,0xab) = 0x1234ab78
 *   You can assume 0 <= n <= 3 and 0 <= c <= 255
 *   Legal ops: ! ~ & ^ | + << >>
 *   Max ops: 10
 *   Rating: 8
 */
int replaceByte(int x, int n, int c) {
  /*
  My Opinion:
  In fact, this question is similar to "logicalShift"
  1. abcdefhg => ab cd ef gh
  2. We need to replace the n-th byte with string_c
  */

  /*
  My Design:
  1. We can create a new C_string: abcd...(C)...efgh
  => c << (n<<3)
  2. Then we need to clear the n-th byte of the original string
  => x & AIM
  AIM needs to satisfy: 111...000...111(only the n-th byte of origin is filled with 0)
  => AIM = ~ (0xFF << (n<<3))
  3. Then we can use | to combine them
  4. Over!
  */

  int AIM = ~(0xFF << (n<<3)); 
  return (x & AIM) | (c << (n<<3));
}
/*
 * multFiveEighths - multiplies by 5/8 rounding toward 0.
 *   Should exactly duplicate effect of C expression (x*5/8),
 *   including overflow behavior.
 *   Examples: multFiveEighths(77) = 48
 *             multFiveEighths(-22) = -13
 *             multFiveEighths(1073741824) = 13421728 (overflow)
 *   Legal ops: ! ~ & ^ | + << >>
 *   Max ops: 18
 *   Rating: 3
 */
int multFiveEighths(int x) {
  /*
  My Opinion:
  This question can be divided into 2 parts:
  1) create 5*x => It is easy to realize => We note xx = x<<2 + x
  2) rounding towards 0 => It is a little bit difficult => After this we will get first_res
  3) We need to divide first_res by 8
  */

  /*
  My Design:
  The real problem is to solve "rounding towards 0"
  1) When xx > 0, it can be automatically achieved by C itself
  2) When xx < 0, we need to add a "Weight" to xx

  How to design this "Weight"?
  Obviously, "Weight" can be 7
  */

  int xx = (x << 2) + x;
  int weight = (x >> 31) & 7; // in this way, the weight is suitable for both positive and negative num
  return ((xx + weight) >> 3);
}
/* 
 * bang - Compute !x without using !
 *   Examples: bang(3) = 0, bang(0) = 1
 *   Legal ops: ~ & ^ | + << >>
 *   Max ops: 12
 *   Rating: 3
 */
int bang(int x) {
  /*
  My Design:(0306)
  => This question is so easy, for I just need to check whether there is a 1 in x
  
  Since we can not use Loop to check for 1 automatically, I just check it one by one!
  <= total bits num is limited to 32 (0-31), so it is not a big deal!
  */
  // x=(x>>16)|x;
  // x=(x>>8)|x;
  // x=(x>>4)|x;
  // x=(x>>2)|x;
  // x=(x>>1)|x;
  // return (~x) & 1;

  /*
  Another Idea:
  if x==0, we directly return 1;
  else => x!=0 
    1) x: 1...01011...0010
      ~x: 0...10100...1101
      I will give: x|(~x+1) < 0 
      it may be a little confusing, here is the proof:
      proof:
        (the next of carry place must be 1, and the place refering to x is still 1)
        => Thus, the result can not be 0!

        someone may ask: what if the carry bit is The Most Significant Bit?
        In fact, the result can still not be 0
        => The Most Significant Bit is 1, so the result can not be 0! (1|0 = 1)
        Over!

      2) demo:
      when x != 0: (~x + 1) | x) >> 31) & 1 == 1, we need to return 0
      when x == 0: (~x + 1) | x) >> 31) & 1 == 0, we need to return 1
      => we use "^1" to solve this problem!
  */ 
  return ((((~x + 1) | x) >> 31) & 1) ^ 1;
}

// This file is made by CS2201H_BoxuanHu
// Time Taken: 5h10min
```

>- completed by CS2201H_BoxuanHu
>- Time Taken: 5h 10min