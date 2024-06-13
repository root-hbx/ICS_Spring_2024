
>This list only contains "in-scope" knowledges

## Code Optimization

- MI Optimization
    - Procedure Calls
    - Memory Aliasing
- MD Operation
    - Basic Programming Skills
    - Loop Unrolling
    - Pipelined Functional Units
    - Branch Predictions


### Machine-Independent Optimization

>两个优化障碍：函数调用，内存别名

- [x] Optimization Blocker: Procedure Calls

函数调用级的优化：省得循环一次调用一次

![](./image/1.png)

- [x] Optimization Blocker: Memory Aliasing

内存别名(一内存位置可以被两个名称访问)

![](./image/2.png)

1. 按理来说, b[i]在最内层循环，其应该储存在寄存器中；但实际上b[i]跑到内存当中去了，原因在于b[i]实质是指针 (定义用的是`double *b`)
2. 极端情况下， 如果b[],a[]在内存同一片地方，运行就会对a[]产生副作用，正是这一种情况导致编译器不把b[]放入寄存器
3. 解决方案是：把b[i]变成临时变量 (在栈上，不会有此问题)

解释：“正是这一种情况导致编译器不把b[]放入寄存器”

- 什么是别名问题：
    - 指针别名问题是指多个指针可能指向同一内存区域，从而导致通过一个指针修改数据会影响到通过另一个指针访问的数据。这种情况会使得编译器无法确定内存的内容是否会在某些操作之间被修改
- 编译器的困惑：
    - 如果编译器不能确定两个指针是否指向同一块内存，它就无法安全地将其中一个指针指向的值存储在寄存器中，而假定该值在寄存器中不会被其他指针修改。这会导致潜在的错误结果
- 数据一致性：
    - 为了确保程序的正确性，编译器需要保证每次通过指针访问内存时，都能获取到最新的数据。如果指针可能别名，编译器就必须保守地处理，即每次访问指针指向的内存，而不是假设值已经在寄存器中保持不变
- 防止数据竞争：
    - 如果编译器将指针指向的值缓存到寄存器中，而该值在内存中被其他操作修改了，寄存器中的值就会变得不一致。这会导致数据竞争和不可预知的行为。

>本质原因是，当需要取两个指针(\*a, \*b)值的时候，编译器的原则始终是“保持最新且正确的数据”；现在有两种可能：（1）a与b指向不同的内存空间（2）a与b指向相同的内存空间
>- (1)的话，理论上来说，指针指向的内存之间无干扰，为了提高速度，可以将“赋值方”的值提供给寄存器
>- (2)的话，后续改a的话，也会对b产生影响，编译器不知道我们要的值究竟是“b的初始值”还是“b经修改后的值”，因此它无法判定这是“特殊的人为癖好”还是“错误”
>- 因此为了保险起见，它都用内存表示，不用寄存器

### Machine-Dependent Optimization

- [x] Basic Optimizations

- Move vec_length out of loop 
- Avoid bounds check on each cycle 
- Accumulate in temporary

examples

1. 把函数调用`i < vec_length(v)`和`get_vec_element(v, i, &val);`放在循环外
2. 用临时变量代替每次循环中改变的变量

- [x] Loop Unrolling

>- 2: `i+=2`
>- 1/2: `for(...){Operation1; Operation2;}`

![alt text](./image/3.png)

1. 2x1 : Reduce loop overhead 
2. 2x1a: Pipeline (流水线化处理，使得指令可以在处理器流水线中并行执行，提高指令吞吐量)
3. 2x2 : Pipeline + Multiple-EU (流水线化和多执行单元结合，最大化利用处理器的多个执行单元并行处理，提高整体计算性能)

__2x1 (循环二阶展开)__

```cpp
for (int i = 0; i < limit; i += 2)
{
    x = x OP a[i] OP a[i+1];
    //<=> x = (x OP a[i]) OP a[i+1];
    
    // CPE = D/1
}
```

1. 步长为2（`i+=2`）
2. 每次循环内只有1行，但是进行两个操作
3. 往往最后附带一个extra loop解决尾巴的问题
4. 优化的原理：每次for循环都是有开销的，打个比方，开一次门放十个作业本肯定比开十次门每次放一个效率高
5. 仅在每次循环任务不重时候有效

__2x1a (在2x1基础上调整运算顺序)__

```cpp
for (int i = 0; i < limit; i += 2)
{
    x = x OP (a[i] OP a[i+1]);
}
// CPE = D/2
```

1. 减小了数据依赖
2. 标红部分做出运算顺序的改变，这一操作是带有风险的
3. 性能提升的原因在于: `x = (x OP1 d[i]) OP2 d[i+1]`中要等OP1完成才能进行OP2，`x = x OP1 (d[i] OP2 d[i+1])` 中两个OP可并行
4. Ops in the next iteration can be started early (no dependency)

__2x2 (操作分为两半)__

```cpp
for (int i = 0; i < limit; i += 2)
{
    x1 = x1 OP a[i];
    x2 = x2 OP a[i+1];
}
// CPE = D/2
```

1. x1专门负责累加d[0/2/4....]，x2负责d[1/3/5....]
2. 原理还是数据依赖去除

>PS 如何分析CPE？
>- 最简单的办法就是，把对应for循环内的代码执行图画出来，看这个程序总共有几条主线，D/主线数 即为 CPE

- [x] Pipelined Functional Units

```cpp
long mult_eg(long a, long b, long c) { 
    long p1 = a*b; 
    long p2 = a*c; 
    long p3 = p1 * p2; 
    return p3; 
}
// We suppose each "MUL" requires 3 cycles
```

Originally, it will be `3x3=9` cycles; But now, we add "Pipeline" Mechanism

![](./image/4.png)

- [x] Branch Predictions

1. _Instruction Control Unit_ must work well ahead of _Execution Unit_ to generate enough operations to keep EU busy
2. If the CPU has to wait for the result of the cmp before continuing to fetch instructions, may waste tens of cycles doing nothing!
3. Guess which way branch will go:
    1. Begin executing instructions at predicted position
    2. __But__ don’t actually modify register or memory data

Principle:

>- Backwards branches are often loops, so predict taken (since we usually write "can be continued code" into `Loop{...}`)
>- Forwards branches are often ifs, so predict not taken (since we usually write "bad situation" into `if(..){..}`)


## Linking

- Symbol Resolution
- Symbol Identification
- How Linker Resolves Duplicate Symbol Definitions
- Linker’s Symbol Rules


- [x] Symbol Resolution(符号解析)：找到不同文件中定义的符号的地址

![alt text](./image/5.png)

- array是全局变量，塞进符号表；
- main是定义的一个函数名，也是全局变量；
- 定义函数sum也是全局函数 
- linker不关心局部变量，所以对于val/i/s一无所知


- [x] Symbol Identification

__ELF Symbol Tables__

- Each relocatable object file has a symbol table in `.symtab` section
- A symbol table contains information about the symbols that are defined and referenced in the file
- Symbol table contains an array of entries

__Linker Symbols__

- Global symbols
    - Def: Symbols defined by module m that can be referenced by other modules.
    - E.g.: __non-static__ C functions and non-static global variables.
- External symbols （Referenced global symbols）
    - Def: Global symbols that are referenced by module m but _defined by some other module_.
- Local symbols
    - Def: Symbols that are defined and referenced exclusively by module m. 
    - E.g.: C functions and global variables __defined with the static attribute__.
    - Local linker symbols are not local program variables

Which of the following names will be in the symbol table of symbols.o?

![](./image/6.png)

![](./image/7.png)

![](./image/8.png)

PS: 这里foo是static int函数，由于是函数定义，所以会进符号表；但是它是Static，所以它的属性还是Local

>Conclusion(什么东西可以进符号表)
>1. 全局变量的定义
>2. 函数定义


- [x] How Linker Resolves Duplicate Symbol Definitions

1. Strong Signal: function + Global Variables(been defined)
2. Weak Signal: Global Variables(not been defined) + Local Variables

![](./image/9.png)

__基于强弱符号的链接规则__

>特别注意：如果x1成功链接到x2，那么所有对x2的操作都会变成对x1的操作

1）多强无弱，强符号不允许重名，否则报错

![](./image/10.png)

2）一强多弱，弱的链接到(指向)强的 ( _linker不进行 符号类型查询_ )

![](./image/11.png)

这个例子值得推敲一下：

- `int x`与`double x`变量名相同，只是类型不同，由于linker不进行符号类型查询，故认为二者相同
- 右边文件所有对x的操作，都会转化为对左边文件x地址的操作，但二者操作方法不一样，所以冲突

3）无强多弱，随机选择一个弱符号，让所有弱符号指向它

![](./image/12.png)


## Processes and Multitasking

- Creating Processes
- Conceptual View of fork
- Modeling fork with Process Graphs

