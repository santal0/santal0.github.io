# Chapter 6 Activation Records 

## 6.0. Activation Records

这一章本质上讨论的是：一个函数被调用时，运行时系统到底要保存哪些信息，这些信息放在内存什么位置，以及编译器如何组织这些信息。


### 6.0.1. Storage Organization（存储组织）

一个程序运行时，它的内存一般如何划分。

从编译器作者的角度看，目标程序运行在自己的 **逻辑地址空间（logical address space）** 中。意思是：

- 程序中的每个值都要有“位置”
- 编译器需要知道：
    - 代码放哪
    - 全局数据放哪
    - 局部变量放哪
    - 动态申请的对象放哪

也就是说，**程序不只是“有语义”，还必须“有地址”**。

#### 运行时表示由两大部分构成

课件说：对象程序（object program）的运行时表示由两部分构成：

- **data areas**：数据区
- **program areas**：程序区

换句话说，程序运行时内存里至少有：

- 可执行代码
- 各种数据

#### 典型内存划分


典型划分包括四块：

- Code 区

    存放：

    - 可执行目标代码
    - 就是编译后真正要执行的机器指令

    你可以把它理解成“程序动作本身”。

- Static 区

    存放 **编译时大小已知** 的数据对象，例如：

    - 全局常量
    - 编译器生成的数据
    - 某些静态变量

    特点：

    - 在程序运行前就知道大小
    - 生命周期通常贯穿整个程序运行期

- Stack 区

    存放：

    - 过程/函数调用时生成的数据结构
    - 这些数据结构就叫 **activation records（活动记录）**
    - 也就是后面要详细讲的 **栈帧**

    特点：

    - 随函数调用而增长
    - 随函数返回而缩小
    - 符合 **后进先出（LIFO）**
- Heap 区

    存放：

    - 由程序显式控制分配和释放的数据

    例如 C 语言：

    - `malloc`
    - `free`

    特点：

    - 生命周期不一定和函数调用同步
    - 分配/释放时机由程序控制
    - 常用于对象、链表、树等动态结构


### 6.0.2. Activation Records（为什么需要活动记录）

从一个最根本的问题开始：**函数的局部变量应该放在哪里？**


#### 6.0.2.1. 什么叫 activation

```pascal
function f(x: int): int = 
let var y := x + x
 in if y<10 
then f(y)
 else return y-1
 end 
```

每调用一次函数 `f`，就产生一次 `f` 的 **activation（激活）**

```c
f(1);
f(2);
```

虽然调用的是同一个函数 `f`，但这是两次不同的 activation。


#### 6.0.2.2. 同一个变量名会有多个实例


- 每次 `f` 被调用时，`x` 都会创建一个新的实例（instantiation）
- 并由调用者初始化

也就是说，函数参数和局部变量不是“全程序唯一一份”，而是：

> **每次调用，重新创建一份。**



“同一函数可以同时有多份局部变量”

- `f(1)` 的 `x`,`y`
- `f(2)` 的 `x`,`y`
- `f(4)` 的 `x`,`y`
- `f(8)` 的 `x`,`y`

这些变量名都一样，但它们是不同调用实例中的不同变量。

所以**必须有一种机制，能同时保存多次调用各自的局部变量。**

这正是 activation record 要解决的问题。

#### 6.0.2.3. 局部变量通常在函数返回后销毁

在很多语言里（例如 C、Pascal、Java）：

- 局部变量在函数返回后就消失
- 它们的生命周期只在函数执行期间有效

这就引出问题：那应该怎么保存这些局部变量？


用栈：

- 函数调用的行为通常符合：

    - 先调用的后返回
    - 后调用的先返回

    这就是 **LIFO（Last In First Out，后进先出）**

- 因此很自然用**栈（stack）**来保存每次函数调用对应的数据。

#### 6.0.2.4. 运行时栈与控制栈

过程调用与返回由控制栈管理，过程调用和返回通常由一个 **run-time stack** 管理，这个栈也叫 **control stack**

当一个过程/函数被调用时，为它的局部变量分配空间，把这些空间压入栈中

当过程结束时，这块空间从栈中弹出

每个“活着的调用”都有一个 activation record，只要某次函数调用还没返回，它就在栈上有一条对应记录

这条记录就是**activation record**，也叫 **frame**，也就是 **stack frame（栈帧）**



## 6.1 Stack Frames 栈帧的结构与工作机制


### 6.1.1 Stack Frames 


虽然栈有 push/pop 的概念，但编译器实际使用栈时并不是“一个变量 push 一次”，实际情况更复杂，因为：

- 局部变量是成批分配/释放的
    - 函数进入时，通常不是一个变量一个变量地 push，而是一次性给整个函数的局部数据留出一大块空间，函数退出时再整体回收。
- 变量不一定一创建就初始化：一个局部变量可能，先分配空间，过一会儿才真正赋值，所以栈不是单纯“值入栈/出栈”那么简单。
- 我们还要访问栈内部较深的位置，很多变量并不在栈顶，例如：

    - 参数在帧的一端
    - 局部变量在中间
    - 保存寄存器在别处

    编译器必须能“随机访问”栈中某个固定偏移位置。

因此把栈看成“大数组”

- 栈在逻辑上可以看成一大片连续内存
- 通过“基址 + 偏移”来访问具体位置


### 6.1.2. Stack Pointer（SP，栈指针）

栈指针是一个特殊寄存器，指向栈中的某个位置。

- SP 之后的位置视为垃圾（garbage）
- SP 之前的位置视为已分配（allocated）

### 6.1.3. 栈帧的基本定义

栈通常只在函数入口增长一次，增长的大小足够容纳这个函数的全部局部变量，在函数退出前，再按同样大小缩回去。

函数的 activation record / stack frame 是栈上专门分给这个函数的一块区域，其中包含局部变量，参数，返回地址，临时值，其他辅助信息

栈通常从高地址向低地址增长，运行时栈通常从较高地址开始，向较小地址增长，新的栈帧会出现在更低地址的位置

如何布局 activation record，才能让 caller 和 callee 正确通信？

- **caller**：调用别人函数的函数
- **callee**：被调用的函数

栈帧的设计要保证 caller 和 callee 能正确通信，例如：

- 参数怎么传
- 返回地址放哪
- 返回后去哪
- 哪些寄存器要保存




### 6.1.4. A typical stack frame layout（典型栈帧布局）

<table style="border-collapse: collapse; width: 100%; max-width: 650px; margin: 0 auto; font-family: 'Times New Roman', Times, serif; text-align: center; line-height: 1.5; font-size: 16px;">
  <tbody>
    <!-- 顶部：高地址指示 -->
    <tr>
      <td style="width: 35%;"></td>
      <td style="width: 30%; border-left: 1px solid currentColor; border-right: 1px solid currentColor; height: 40px;"></td>
      <td style="width: 35%; text-align: left; padding-left: 15px; vertical-align: top;">
        &uarr; higher addresses
      </td>
    </tr>

    <!-- 上一个栈帧：传入参数 (Incoming Arguments) -->
    <tr>
      <td style="vertical-align: middle; text-align: center; padding-right: 10px;">
        incoming<br>arguments
      </td>
      <td rowspan="2" style="border: 1px solid currentColor; padding: 15px 5px;">
        argument <i>n</i><br>.<br>.<br>.<br>argument 2<br>argument 1<br>static link
      </td>
      <td rowspan="2" style="text-align: left; padding-left: 15px; vertical-align: middle;">
        previous<br>frame
      </td>
    </tr>
    <!-- 帧指针 -->
    <tr>
      <td style="vertical-align: bottom; text-align: right; padding-right: 10px;">
        <div style="border: 2px solid #e60000; color: inherit; display: inline-block; padding: 3px 6px; white-space: nowrap;">
          frame pointer &rarr;
        </div>
      </td>
    </tr>

    <!-- 当前栈帧：局部变量 (Local Variables) -->
    <tr>
      <td></td>
      <td style="border: 1px solid currentColor; padding: 15px 5px;">
        local<br>variables
      </td>
      <td rowspan="4" style="border-top: 1px solid currentColor; text-align: left; padding-left: 15px; vertical-align: middle;">
        current<br>frame
      </td>
    </tr>

    <!-- 当前栈帧：返回地址、临时变量、保存的寄存器 -->
    <tr>
      <td></td>
      <td style="border: 1px solid currentColor; padding: 15px 5px;">
        return address<br><br>temporaries<br><br>saved<br>registers
      </td>
    </tr>

    <!-- 当前栈帧：传出参数 (Outgoing Arguments) -->
    <tr>
      <td style="vertical-align: middle; text-align: center; padding-right: 10px;">
        outgoing<br>arguments
      </td>
      <td rowspan="2" style="border: 1px solid currentColor; padding: 15px 5px;">
        argument <i>m</i><br>.<br>.<br>.<br>argument 2<br>argument 1<br>static link
      </td>
    </tr>
    <!-- 栈指针 -->
    <tr>
      <td style="vertical-align: bottom; text-align: right; padding-right: 10px;">
        <div style="border: 2px solid #e60000; color: inherit; display: inline-block; padding: 3px 6px; white-space: nowrap;">
          stack pointer &rarr;
        </div>
      </td>
    </tr>

    <!-- 下一个栈帧：底端开口及低地址指示 -->
    <tr>
      <td></td>
      <td style="border-left: 1px solid currentColor; border-right: 1px solid currentColor; height: 50px;"></td>
      <td rowspan="2" style="border-top: 1px solid currentColor; text-align: left; padding-left: 15px; vertical-align: top; padding-top: 20px;">
        next<br>frame<br><br><br>&darr; lower addresses
      </td>
    </tr>
    <tr>
       <td></td>
       <td style="border-left: 1px solid currentColor; border-right: 1px solid currentColor; height: 40px;"></td>
    </tr>
  </tbody>
</table>

1. incoming arguments：这是调用者传给当前函数的参数 `f(a, b, c);` 对于 `f` 来说，`a b c` 就是 incoming arguments。它们可能在寄存器里、在栈里或进入函数后被搬到某个统一位置

2. local variables（局部变量）：当前函数内部声明的变量。有些局部变量在栈帧里，有些可能直接放寄存器。

3. return address（返回地址）：当前函数执行完之后，要跳回调用者的哪条指令继续执行，通常由 `CALL` 指令自动产生。

4. temporaries（临时值）：编译器在表达式计算过程中会产生很多临时结果，例如：`a + b * c` 中间的 `b * c` 就可能产生临时值，这些值有时也需要在栈帧中保存。

5. saved registers（保存的寄存器）：调用过程中，有些寄存器的旧值必须先保存起来，等函数结束再恢复。否则会破坏调用者原本依赖的内容。

6. outgoing arguments（传出参数）：当前函数如果还要调用别的函数，就要准备传给下一个函数的参数区域。所以一个帧里不仅有“收到的参数”，还可能有“准备传出去的参数”。

7. static link（静态链）：这是后面讲嵌套函数时的关键。让内部函数访问外层函数的变量

### 6.1.5. Frame Pointer（帧指针）—— 为什么还需要 FP

#### 6.1.5.1. 引入

假设函数 `g(...)` 调用函数 `f(a1, ..., an)`：

- `g` 是 caller
- `f` 是 callee

当 `g` 调用 `f` 时：

- 栈指针 `SP` 指向 `g` 传给 `f` 的第一个参数
- `f` 通过从 `SP` 中减去帧大小来分配自己的 frame

#### 6.1.5.2. 实现

进入函数 f 时：

- 先把旧 FP 保存起来
- `FP = SP`：让 FP 指向当前帧的基准位置
- `SP = SP - framesize`：把 SP 往下移动，分出本函数所需空间


退出函数 f 时：

- `SP = FP`：把 SP 恢复到当前帧起点，相当于回收整帧
- fetch back the saved old FP：把旧 FP 取回来，恢复调用者的帧基准，然后再返回给调用者。

如果 frame size 可变，或 frame 不一定总是连续，这种安排很有用，原因是FP 给你一个稳定参考点，哪怕 SP 后面有波动。

若帧大小固定，可以有 `FP = SP + framesize`，FP 甚至可以被看成一个“虚构寄存器（fictional register）”，实现中未必真的需要单独物理寄存器存 FP

### 6.1.6. Registers

#### 6.1.6.1. 寄存器的作用和挑战

寄存器访问速度比内存快，所以编译器总希望把局部变量尽量放在寄存器里，把临时值也尽量放在寄存器里，问题是很多函数都想用同一个寄存器，现代机器通常有很多寄存器（例如 32 个），但仍然有限。

假设 `f` 用寄存器 `r` 存一个局部变量，`f` 调用 `g`，`g` 也要使用 `r`，在 `g` 使用 `r` 前，必须先把 `r` 里原来的值保存起来；用完后再恢复。通常保存到栈帧中。


如果寄存器由 **调用者** 负责保存与恢复，那么它叫 caller-save register，例如 `f` 调 `g` 前，自己先把需要保留的寄存器值存起来， `g` 返回后，`f` 再恢复

如果寄存器由 **被调用者** 负责保存与恢复，那么它叫 callee-save register，也就是 `g` 一进来先保存自己要用的寄存器， `g` 返回前再恢复

### 6.1.7. Parameter Passing（参数传递）参数传递机制。

#### 6.1.7.1. 按值传递（Pass-by-Value）

Tiger 使用 **Pass-by-Value（值传递）**：实参的值被复制给形参，修改形参不会影响实参。

如果所有参数都通过栈传递，那么调用者要写到内存，被调用者可能又要从内存取回来，只靠栈传参数会带来额外内存流量，这会产生不必要的内存访问成本。


#### 6.1.7.2. 常用的优化

现代机器通常规定：

- 前 `k` 个参数（常见是 4 或 6 个）通过寄存器传
- 其余参数才通过内存/栈传

这样可以显著减少内存流量，提高性能。


#### 6.1.7.3.寄存器传参带来的问题

寄存器传参虽然快，但会带来寄存器覆盖问题。

```c
void f(int a) {
  int z = ...
  h(z);
  ...
  int t = a + 2;
  ...
}
```

假设函数 `f` 的参数 `a` 是通过寄存器 `r1` 接收的。

然后 `f` 内部调用 `h(z);`，如果调用约定规定 `h` 的第一个参数也要放 `r1`，那么 `f` 必须把 `z` 放到 `r1` 中。`r1` 里原来装的是参数 `a`，如果后面还要继续使用 `a`，比如 `int t = a + 2;`，那么在调用 `h(z)` 之前，`a` 就不能直接被覆盖掉。

所以 `f` 要先把旧的 `r1` 内容保存到自己的栈帧，再把 `z` 放入 `r1`。

本来寄存器传参是为了减少内存访问，结果现在又得把寄存器内容写回内存，似乎又增加了内存流量。所以问题变成**如何尽量避免这种额外保存**？

接下来几页就是回答这个问题。

#### 6.1.7.4. 避免额外内存流量的方法

##### 覆盖死变量

**dead variable（死变量）** 的意思是当前程序点往后，再也不会用到它

如果参数已经“死了”，就直接覆盖，如果在调用 `h(z)`` 时，参数 `a` 之后不会再被访问，那么就可以直接覆盖 `r1`，不必先保存 `a`

##### 叶子过程不必把 incoming arguments 写到内存

leaf procedure（叶子过程）：不调用其他过程/函数的函数

它不会再把参数寄存器拿去传给别的函数，所以传进来的参数可一直保留在寄存器中，所以 leaf procedure 不必把 incoming arguments 写到内存


##### 函数间寄存器分配

一些优化编译器会一次分析整个程序的多个函数，为不同函数分配不同寄存器来接收参数和保存局部变量

例如如果编译器知道：`f` 常用 `r1`，`g` 不一定非要用 `r1`，那就可以安排 `g` 的参数用别的寄存器，避免 `f` 的值被覆盖

这类优化很强，但也很复杂，因为它不再只看单个函数，而要看整个程序调用关系。

##### register windows（寄存器窗口）

某些体系结构支持 **register windows**，每次函数调用时，硬件给它“切换出一组新寄存器”，不同调用拥有不同“窗口”，因而减少保存/恢复的内存流量

### 6.1.8. 不可使用寄存器的情形

为了解决参数通过寄存器传递，但又有地址的问题，任何被获取地址的参数在进入函数时必须写入内存位置。例如：

```c
int *f (int x) {return &x}
```
需要获得 x 的地址，所以 x 必须被写入内存。

获取局部变量地址的更优雅方式是使用按引用调用。

- 使用按引用调用时，如果 x 被作为参数传递给 f(y)，其中 y 是“按引用”参数，编译器会生成代码**传递 x 的地址**，而不是 x 的内容。
- 在函数中任何使用 y 的地方，编译器都会生成额外的指针解引用。
- 使用按引用调用时，不会出现“悬空引用”，因为 y 必须在 f 返回时消失，而 f 的返回发生在 x 的作用域结束之前。

### 6.1.9. 返回地址 return address 

假设函数 `g` 调用函数 `f`，当 `f` 执行结束后，它应该回到 `g` 的哪里继续执行？如果 `g` 中调用 `f` 的指令在地址 `a`，通常返回位置就是`a + 1`，即调用指令之后的下一条指令，这就是 **return address（返回地址）**。

现代机器常让 `call` 指令把返回地址放到一个指定寄存器里，例如 MIPS 里的 `$ra` 

非叶子函数通常要把返回地址写入栈，如果当前函数还要调用其他函数（即非叶子函数）， `$ra` 可能被后续调用覆盖，所以要先把返回地址写入当前栈帧，而叶子函数因为不再调用别人，通常不需要把它写栈。


### 6.1.10. Frame-Resident Variables


现代调用约定中，很多东西都在寄存器里

- 函数参数：可能在寄存器中
- 返回地址：可能在寄存器中
- 返回值：通常在寄存器中
- 很多局部变量和中间结果：也会在寄存器中

什么时候值必须写到内存（栈帧）中，变量变成 **frame-resident variables（驻留在帧中的变量）**？

1. 变量按引用传递：如果变量要 **pass by reference（引用传递）**，那么它必须有内存地址。因为引用传递本质上传的是“地址”而不是纯值。

2. 被内部嵌套过程访问：如果一个变量会被当前函数内部定义的嵌套函数访问，那么它不能只放在寄存器里。因为内部函数需要通过某种地址机制去找到它。这通常要求它在帧中有稳定位置。

3. 值太大，装不进一个寄存器：寄存器大小有限，如果一个值太大，例如复合对象、较大结构，就不能只靠单寄存器保存。

4. 变量是数组：数组通常需要地址运算/按元素偏移访问，因此数组变量往往要在内存中有明确地址。

5. 寄存器另有用途，需要让位，那么原先放在寄存器里的变量可能要临时写回帧中。

6. 寄存器不够，发生 spilling（溢出）：如果局部变量和临时值太多，寄存器装不下，那么编译器会把其中一些 **spill** 到栈帧里。


一个变量 escapes 如果：

- 它被按引用传递
- 它的地址被取出（如 C 里的 `&x`）
- 它被嵌套函数访问

Pass-by-reference：传的是实际参数的位置，形参的访问要经过隐式间接寻址，修改形参会影响实参


### 6.1.11. static link

#### 6.1.11.1. Block Structure（块结构）

在支持嵌套函数声明的语言里，例如 Pascal、ML、Tiger，一个函数内部可以再定义函数，这些内部函数可以使用外层函数的变量，这就带来问题：内层函数如何访问“非局部变量（non-local variable）”？

普通局部变量可以通过 `Frame Pointer + offset` 访问。虽然运行时实际 FP 的值要到运行时才知道，但相对于 FP 的偏移量在编译时就知道，这对当前函数自己的局部变量没问题。

如果内层函数要访问外层函数的变量，那变量不在自己帧里，而在别人的帧里。所以还需要一条“通向外层帧”的路径，这就引出 static link。

#### 6.1.11.2. 例子

```c
type tree = {key: string, left: tree, right: tree}
function prettyprint(tree: tree) : string =
  let
    var output := “”
    function write(s: string) = 
      output := concat(output,s)
    function show(n: int, t: tree) =
      let function indent(s: string) =
        (for i := 1 to n
        do write(“ ”));
        output := concat(output, s);
        write("\n"))
      in if t=nil 
        then indent(".")
        else (indent(t.key));
          show(n+1, t.left);
          show(n+1, t.right))
      end
  in show(0, tree); output
end
```

大致结构是：

- `prettyprint`
  - 内有变量 `output`
  - 内定义 `write`
  - 内定义 `show`
    - 内又定义 `indent`

挑战在于：

- `write` 需要访问 `prettyprint` 的 `output`
- `indent` 需要访问：
    - `show` 的 `n`
    - `prettyprint` 的 `output`
    - 还可能调用 `write`

所以 `indent` 不仅要知道自己的帧，还要能走到外层多个帧。这正是“非局部访问”问题


这里所谓 non-local variable 是：

- 对当前函数不是本地变量
- 但在某个外层作用域里声明的变量

怎样让嵌套函数顺着某种链找到外层活动记录？static link 方案。


#### 6.1.11.3. static link 的定义


每次调用一个函数 `f` 时，要额外传给它一个指针，指向在程序文本中直接包围 `f` 的那个函数 `g` 的最近一次活动记录。这个指针就叫 **static link**


immediately encloses function：

```text
g 中定义 f
```

那么调用 `f` 时，static link 指向的是 `g` 的最近一次活动记录，不是“所有外层函数”，而是“直接外层函数”。


当 `prettyprint` 调用 `show` 时，应该传 `prettyprint` 自己的 frame pointer 作为 `show` 的 static link。因为 `prettyprint` 是 `show` 的直接外层函数，所以 `show` 的 static link = `prettyprint` 的 frame pointer

当 `show` 调 `show`（递归调用）时，传递的 static link 是 **show 自己的 static link**，而不是 show 自己的 frame pointer，因为 static link 指向的是直接外层函数的活动记录，而 `show` 的直接外层函数不是 `show` 自己，而是 `prettyprint`。所以即使 `show` 递归调用自己，新的 `show` 实例的直接外层仍然是 `prettyprint`，- 因此应该继续把 `prettyprint` 的帧指针传下去。


indent 怎么取到 output，`indent` 自己没有 `output`，它的 static link 指向 `show` 的帧，而 `show` 的 static link 又指向，`prettyprint` 的帧，所以 `indent` 要访问 `output` 时，需要：

1. 从自己的 static link 到 `show`
2. 再跟着 `show` 的 static link 到 `prettyprint`
3. 再取 `prettyprint` 帧里的 `output`


访问外层变量，本质上是：

- 沿 static link 链向上走若干步
- 到达声明该变量的那层帧
- 再按该变量的 offset 取值



```c
int f (link, int x, int y) {
    int m;
    int g (link, int z) {
        int h (link) {
            return link->prev->m + link->z;
        }
        return 1;
    }
    return 0;
}
```

`h` 要访问：

- `z`：在 `g` 的帧里
- `m`：在 `f` 的帧里

所以需要通过 link 一层一层找到对应帧。

#### 6.1.11.4. 总结

访问非局部变量，每个函数标注一个 **enclosing depth（嵌套深度）**。如果深度为 `n` 的函数访问深度为 `m` 的变量，则需要沿 static link 向上爬 `n - m` 步


优点：参数传递开销小，因为每次函数调用只多传一个指针，也就是 static link，额外参数成本不高。

缺点：
- 访问非局部变量时要沿链爬，每次访问非局部变量都可能要多次间接访问内存。
- 嵌套越深，代价越大：间接引用次数 = 使用位置深度 - 声明位置深度，如果函数嵌套很深，成本会明显增加。



#### 6.1.11.5. 另一种实现块结构的方法 Display（显示表）

这一节给出另一种实现块结构的方法：**Display**。

Display 是一个 **全局指针数组**。其中下标 `i` 位置保存静态嵌套深度为 `i` 的、最近进入的那个过程的栈帧指针。

例子中：

- main：1
- prettyprint：2
- write：3
- show：3
- indent：4

所以：

- `display[2]` 指向当前的 prettyprint 帧
- `display[3]` 指向当前最近进入的第 3 层函数帧
- `display[4]` 指向当前 indent 帧

好处：访问某个外层深度的变量时，不必一层层爬 static link。可以直接根据变量所属深度访问 `display[depth]`，再加 offset 取值，这样非局部变量访问更快。

代价：但进入/退出函数时，需要维护 display 数组的内容。所以它是用函数进入/退出开销换访问非局部变量更快


#### 6.1.11.6. 第三种实现块结构的方法：**Lambda Lifting**。

当 `g` 调用 `f` 时，把 `g` 中实际被 `f`（或 `f` 内部更深层函数）访问到的变量，作为额外参数传给 `f`。也就是说把原本的“非局部变量访问”，改写成“显式参数传递”。

- 本质是程序重写：把非局部变量改写为形式参数
- 最内层过程开始向外做转换

```c
int f (int x, int y) 
{
  int m; 
  int g (int z) 
  {
    int h () { 
      return m+z; 
    }
    return 1;
  } 
  return 0;
}
```

原始代码中：

- `h` 使用了 `m` 和 `z`
- 但这两个不都是 `h` 的局部参数



经过 lambda lifting 后，变成：

- `g(int &m, int z)`
- `h(int &m, int &z)`

```c
int f (int x, int y) 
{
  int m; 
  int g (int &m, int z) 
  {
    int h (int &m, int &z) { 
      return m+z; 
    }
    return 1;
  } 
  return 0;
}
```

也就是把非局部依赖显式补进参数表。


### 6.1.12. 嵌套函数 + 函数作为值返回

本节开始讨论更难的情况：**嵌套函数 + 函数作为值返回**。

```pascal
fun f(x) =
  let fun g(y) = x + y
  in g
  end

val h = f(3)
val j = f(4)
val z = h(5)
val w = j(7)
```

这里：

- `f(3)` 返回一个函数 `g`，这个 `g` 记住了 `x=3`
- `f(4)` 返回另一个函数 `g`，记住了 `x=4`

于是：

- `h(5)` 得到 8
- `j(7)` 得到 11



问题是 `f` 已经返回了，`x` 为什么还活着？生命周期超过了函数。如果 `x` 只是普通栈上局部变量，那么 `f` 返回后它的栈帧应该已经销毁。但返回出去的函数 `g` 还要继续使用 `x`。有些局部变量的生命周期比其外层函数调用更长。


栈的假设是函数返回，局部变量就消失，但高阶函数打破了这个假设，如果语言同时支持嵌套函数和函数值返回，那么不是所有局部变量都能只放栈里。这会引向闭包、堆分配等更高级话题。

|语言|支持嵌套函数|支持函数作为值返回|讲解|
|:-:|:---------:|:----------------:|:-----:|
|Pascal|有|不能|局部变量生命周期仍受调用限制，能用栈处理|
|C|可以处理函数指针/函数值风格的东西， 但不支持嵌套函数（标准 C）||不需要让内部函数捕获外层局部变量，仍可主要靠栈|
|ML、Scheme 等|支持|支持|所以它们不能把所有局部变量都只放在栈里|


## 6.2 Tiger 编译器如何抽象和实现 frame


### 6.2.1. Tiger 编译器中的 frame 接口

#### 6.2.1.1. `frame.h` 接口

抽象类型

```c
typedef struct F_frame_ *F_frame;
typedef struct F_access_ *F_access;
typedef struct F_accessList_ *F_accessList;
```

- `F_frame`：表示一个栈帧描述
- `F_access`：表示一个变量/参数的访问方式
- `F_accessList`：多个访问方式组成的链表

这里都用了抽象指针类型，隐藏具体实现。

主要接口函数

```c
F_frame F_newFrame(Temp_label name, U_boolList formals);
Temp_label F_name(F_frame f);
F_accessList F_formals(F_frame f);
F_access F_allocLocal(F_frame f, bool escape);
```

`frame.h` 是抽象接口，由具体目标机器模块实现，如 `mipsframe.c`，前端/中端不直接依赖某台机器的栈帧细节，而通过统一接口访问。这体现了编译器模块化设计。

#### 6.2.1.2. `F_frame`

```c
/* frame.h */
typedef struct F_frame_ *F_frame;
F_frame F_newFrame(Temp_label name, U_boolList formals);
Temp_label F_name(F_frame f);
```

`F_frame` 保存关于一个函数帧的信息，尤其是形式参数的信息+局部变量的分配信息，它不是“运行时真正的帧本身”，更准确说是编译器对该帧布局的描述对象。

`F_newFrame(f, l)`  创建函数 `f` 的新 frame。

参数：

- `name`：函数标签名
- `l`：一个布尔列表，每个布尔值对应一个形式参数是否 escape

如果某形参 escape，就分配到 frame 中。


#### 6.2.1.3. F_access

F_access` 描述一个形式参数或局部变量在运行时放哪里：

- 在 frame 中
- 还是在寄存器中

这是一个抽象数据类型。

举例：

- `InFrame(X)`
- `InReg(t84)`

对应实现类似：

```c
struct F_access_ {
  enum {inFrame, inReg} kind;
  union {
    int offset;     /* InFrame */
    Temp_temp reg;  /* InReg */
  } u;
};
```

`InFrame(offset)`：表示该变量在当前帧中的某个偏移位置。

`InReg(temp)`：表示该变量放在某个临时寄存器（抽象 temp）中。


#### 6.2.1.4. F_formals 与 shift of view

这一页非常重要。

------

## 1. F_formals 做什么

`F_formals(F_frame f)` 返回一个列表，表示：

- 从 **callee 内部视角** 看，每个形式参数应该如何访问

也就是：

> 函数体内部看参数时，它们在哪里。

------

## 2. caller 视角与 callee 视角可能不同

课件强调：

参数在 caller 与 callee 看来可能不一样。

例如：

### 如果参数通过栈传递

- caller：可能按 `SP + offset` 放参数
- callee：可能按 `FP + offset` 取参数

### 如果参数通过寄存器传递

- caller：可能把参数放寄存器 6
- callee：可能希望在寄存器 13 里统一处理，或者搬到 temp

------

## 3. 这叫 “Shift of View”

这是一个非常经典的概念。

意思是：

> **同一份参数，在调用者眼中和在被调用者眼中，其位置/访问方式可能不同。**

编译器必须在函数入口处理这种视角转换。

------

# 第 46 页：Frames in The Tiger Compiler（谁来处理 shift of view）

这一页说明：

- 这种视角转换依赖目标机器调用约定
- 所以应由 `Frame` 模块负责
- 从 `newFrame` 开始处理

------

## 1. 对每个 formal，newFrame 要决定两件事

### 第一：函数内部怎样看它

- 是 in register？
- 还是 in frame？

### 第二：要生成哪些指令来实现这种转换

例如：

- 从参数寄存器搬到 temp
- 或从寄存器写入帧槽位

------

## 2. 这一页核心理解

`newFrame` 不只是“记住参数”，还要：

> **确定参数在函数入口的落点，并预备视角转换所需的指令。**

------

# 第 47 页：Representation of Frame Descriptions（F_frame 内部应保存什么）

这一页总结 frame 描述内部要保存的信息。

------

## F_frame 应保存：

1. 所有 formals 的位置
2. 实现 view shift 所需的指令
3. 当前已分配的 locals 数量
4. 该函数机器代码入口的 label

------

## 初学者理解

这再次说明：

`F_frame` 是一个“编译器内部的帧说明书”。

它要告诉后续阶段：

- 参数在哪
- 局部变量还分到哪了
- 入口标签是什么
- 进入函数时需要做哪些搬运动作

------

# 第 48 页：Representation of Frame Descriptions（view shift 例子）

这一页是一个很重要的例子。

------

## 1. 场景

假设函数有三个参数，且第一个参数 escape。

不同机器（Pentium/MIPS/Sparc）下：

- 形式参数可能分别落到不同位置
- view shift 指令也不同

------

## 2. 为什么要把 `r4`、`r5` 搬到 `t157`、`t158`

课件问：

> Why move r4 and r5 to t157 and t158?

答案是：

- `r4`,`r5` 是调用约定下 caller 传来的寄存器
- 但函数内部想以统一、更抽象的形式来操作参数
- 所以先搬到抽象 temporaries（如 `t157`,`t158`）

之后寄存器分配器再决定：

- `t157` 最终映射到哪个物理寄存器
- 或是否 spill 到栈

------

## 3. 这一页核心结论

> **view shift 让“外部传参方式”转化为“函数内部统一访问方式”。**

这正是 frame 模块的重要职责。

------

# 第 49 页：Local Variables（局部变量分配）

这一页开始讲局部变量如何分配。

------

## 1. 有些局部变量在 frame 中，有些在寄存器中

这与前面一致：

- escape 的通常在 frame
- 不 escape 的通常可在寄存器

------

## 2. 分配局部变量的接口

```c
F_access F_allocLocal(F_frame f, bool escape);
```

含义：

- 在帧 `f` 中为一个新局部变量分配位置

------

## 3. 依据 escape 决定分配结果

如果：

- `escape = True`，返回 `InFrame`
- `escape = False`，可以返回 `InReg`

这里的“可以”很重要，因为最终还可能受别的实现策略影响。

------

# 第 50 页：Local Variables（什么时候调用 F_allocLocal）

这一页通过嵌套作用域中的同名变量解释一个容易错的点。

------

## 1. 例子

函数中多次出现：

```c
int v = 6;
...
{ int v = 7; ... }
...
{ int v = 8; ... }
...
```

虽然名字都叫 `v`，但它们是：

- 三个不同的变量

输出会是：

- `6 7 6 8 6`

------

## 2. 编译器怎么处理

每遇到一个变量声明，就调用 `allocLocal` 为它分配：

- 一个新的 temporary
- 或一个新的 frame slot

即使后面离开作用域，该名字绑定会失效，但空间/临时表示已经为整个函数保留。

------

## 3. 关键理解

> **“名字相同”不等于“变量相同”。**

词法作用域中的每次声明，都是新的实体。

------

## 4. 本页重点问题

什么时候调用 `F_allocLocal`？

答案是：

- **在处理每一个变量声明时调用**
- 而不是在第一次遇到某个变量名时只分配一次

------

# 第 51 页：Local Variables（局部变量复用优化）

这一页继续上页，但提出更高级的优化可能。

------

## 1. 临时量/帧槽位是否能复用

课件说：

- 寄存器分配器会尽量少用寄存器表示 temporaries
- 第二个和第三个 `v` 可能共用同一个 temporary
- 更聪明的编译器甚至可让两个 frame-resident 变量共用同一栈槽

------

## 2. 为什么可以复用

因为它们的生命周期不重叠。

如果两个变量不会同时活跃，就可以共享存储位置。

这和寄存器分配、栈槽复用本质类似。

------

## 3. 但逻辑上它们仍是不同变量

即使最终物理位置可能被复用，语义上：

- 第二个 `v`
- 第三个 `v`

仍然是不同声明、不同绑定。

------

# 第 52 页：Calculating Escapes（如何计算逃逸）

这一页讲逃逸分析接口。

------

## 1. 为什么必须先做 escape analysis

因为调用 `F_allocLocal` 时，必须知道这个变量是否 escape。

否则就不知道该分到：

- frame
- 还是 register

------

## 2. `Esc_findEscape` 做什么

课件说：

- 遍历整棵抽象语法树（AST）
- 查找每个变量是否出现逃逸使用
- 用环境记录变量是否逃逸

------

## 3. 深度思想

它通常会记录：

- 当前遍历所在的嵌套深度
- 变量声明时的深度

如果某变量在比其声明更深的函数中被访问，就说明它 escape。

------

## 4. 这一页核心结论

> **逃逸分析是在存储分配之前进行的关键静态分析。**

------

# 第 53 页：Temporaries and Labels（临时量与标签）

这一页进入另一个编译器抽象层。

------

## 1. 语义分析阶段还不能决定真实寄存器与真实地址

课件说：

- 太早了，不能现在就决定：
  - 参数/局部变量到底放哪个物理寄存器
  - 函数体到底放哪个机器地址

所以需要抽象表示。

------

## 2. 用两个抽象概念

### Temporary

表示暂时放在寄存器里的值的抽象名字。

### Label

表示某个机器代码地址位置的抽象名字。

------

## 3. 为什么需要抽象

因为编译是分阶段的：

- 先做语义与中间表示
- 后做指令选择
- 再做寄存器分配
- 最后地址布局

早期阶段只需“有名字可引用”，不必知道最终物理细节。

------

# 第 54 页：Temporaries and Labels（temp.h 接口）

这一页给出接口。

------

## 1. Temp_temp

```c
typedef struct Temp_temp_ *Temp_temp;
Temp_temp Temp_newtemp(void);
```

作用：

- 生成一个新的 temporary

它来自“无限集合”，也就是概念上永远能分配新 temp。

------

## 2. Temp_label

```c
typedef S_symbol Temp_label;
Temp_label Temp_newlabel(void);
Temp_label Temp_namedlabel(string name);
string Temp_labelstring(Temp_label s);
```

作用：

- 生成新标签
- 生成有指定名字的标签
- 取标签字符串名

------

## 3. 列表类型

还有：

- `Temp_tempList`
- `Temp_labelList`

方便保存多个 temp 或 label。

------

## 4. 这一页核心结论

> **temp 和 label 是编译器内部对“寄存器名”和“地址名”的延迟抽象。**

------

# 第 55 页：Two Layers of Abstraction（两层抽象）

这一页非常关键，讲编译器结构。

------

## 1. 第一层抽象：frame.h 与 temp.h

它们提供：

- 内存中变量的机器无关表示
- 寄存器中变量的机器无关表示

也就是说，语义阶段不必关心变量具体存在何处。

------

## 2. 第二层抽象：Translate 模块

课件说：

- Translate 模块进一步处理嵌套作用域（通过 static links）
- 向 Semant 提供 `translate.h` 接口

------

## 3. 为什么要两层

因为：

### frame/temp 层

解决“变量在机器层面怎么存”

### translate 层

解决“变量在源语言作用域结构里怎么找到”

------

## 4. 这一页核心理解

> **Tiger 编译器把“机器存储细节”和“语言作用域细节”分开处理。**

这是一种非常漂亮的模块化设计。

------

# 第 56 页：Two Layers of Abstraction（Translate 接口）

这一页给出 `translate.h` 的主要接口。

------

## 1. `Tr_level`

表示函数所在的嵌套层次。

------

## 2. 主要接口

```c
Tr_level Tr_outermost(void);
Tr_level Tr_newLevel(Tr_level parent, Temp_label name, U_boolList formals);
Tr_accessList Tr_formals(Tr_level level);
Tr_access Tr_allocLocal(Tr_level level, bool escape);
```

------

## 3. 这些接口的含义

### `Tr_outermost()`

返回最外层 level。

### `Tr_newLevel(...)`

为新函数创建新的嵌套层级。

### `Tr_formals(level)`

取得该层函数形参的访问方式。

### `Tr_allocLocal(level, escape)`

在该层为局部变量分配访问方式。

------

## 4. Translate 管什么

课件说：

Translate 为 Semant 管理：

- local variables
- static function nesting

也就是它把“变量在哪一层声明、函数嵌套关系如何”也一起纳入管理。

------

# 第 57 页：Two Layers of Abstraction（环境条目中保存 level）

这一页展示新的环境项结构。

------

## 1. 变量条目

变量环境条目中保存：

- `Tr_access access`
- 类型 `Ty_ty`

------

## 2. 函数条目

函数环境条目中保存：

- `Tr_level level`
- `Temp_label label`
- 形参类型列表
- 返回类型

------

## 3. 为什么环境里要保存 level

因为在翻译变量访问时，需要知道：

- 当前代码所在 level
- 变量/函数定义所在 level

这样才能判断：

- 是否本层变量
- 是否需要走 static link
- 走几层

------

## 4. `Tr_access_` 的结构

```c
struct Tr_access_ {Tr_level level; F_access access;};
```

说明：

- `Tr_access` = “处于哪个源语言层次” + “底层 frame 访问方式”

这正是两层抽象的结合点。

------

# 第 58 页：Managing Static Links（为什么 static link 由 Translate 管）

这一页回答一个设计问题：

为什么不用 Frame 模块直接管理 static link？

------

## 1. 原因：不是所有语言都有嵌套函数

很多语言没有 nested function declarations。

因此：

- Frame 应该尽量独立于具体源语言特性
- 不应强绑定“嵌套作用域”这种语言层概念

------

## 2. Translate 更懂源语言语义

Translate 知道：

- 哪个函数嵌套在哪个函数里
- 每个帧中 static link 应该怎样传递
- 哪些变量访问需要经过 static links

所以更适合由 Translate 管。

------

## 3. static link 作为“像参数一样”的隐藏参数

课件说：

- static link 传给函数时像参数
- 存入 frame 时也像参数

因此可以“尽量把它当参数处理”。

------

## 4. 这一页核心结论

> **Frame 负责机器无关的帧访问抽象，Translate 负责源语言作用域和静态链管理。**

------

# 第 59 页：Keeping Track of Levels（最外层 level）

这一页解释 `Tr_outermost`。

------

## 1. outermost 是什么

它表示：

- Tiger 主程序所嵌套的最外层 level

也是各种库函数声明所在的层次。

------

## 2. 最外层没有 frame/formals

课件说：

- outermost 并不包含真正的 frame 或形式参数列表

因为它更像一个逻辑上的根作用域。

------

## 3. 为什么需要这样一个根

因为所有嵌套层级都必须有一个起点。

例如：

- 主程序在 outermost 之内
- 主程序定义的函数再在其内层

这样 level 链才完整。

------

# 第 60 页：Homework

这一页只是作业：

- 6.3
- 6.7(a,b)

对本章知识体系没有新增内容。

------

# 第 61 页：Temporaries and Labels（补充：Temp_newtemp）

这一页是对前面 temp 接口的重复强调。

重点是：

- `Temp_newtemp()` 从一个“无限集合”中返回新的 temporary

------

## 含义

每次需要新的抽象寄存器名字时，就调用它。

它不是物理寄存器编号，而是：

- 编译器内部中间名字

------

# 第 62 页：Temporaries and Labels（补充：Temp_newlabel）

这一页强调：

- `Temp_newlabel()` 返回一个新的 label
- label 也来自“无限集合”

------

## 含义

用于表示：

- 函数入口
- 跳转目标
- 基本块标签
- 条件分支位置

在最终汇编生成前，它们都是抽象名字。

------

# 第 63 页：Temporaries and Labels（补充：Temp_namedlabel）

这一页强调：

- `Temp_namedlabel(string)` 生成一个汇编名为该字符串的标签

------

## 注意事项

课件特别提醒：

> 不同作用域里可能有同名函数

所以不要简单认为：

- 源代码函数名 = 唯一机器标签名

编译器往往还需要做名称区分、重命名或 mangling。

------

# 第 64 页：Two Layers of Abstraction（补充：Tr_newLevel / Tr_allocLocal）

这一页再次强调两个接口。

------

## 1. `Tr_newLevel`

为每个函数创建新的嵌套层级。

通常在处理函数声明（如 `transDec`）时调用。

------

## 2. `Tr_allocLocal(lev, esc)`

当 Semant 在 level `lev` 里处理一个局部变量声明时，调用它来分配变量。

这里的 `esc` 就来自前面 escape analysis 的结果。

------

## 3. 这一页强化的联系

这页实际上把三件事串起来了：

- 语义分析发现变量声明
- escape analysis 判断是否逃逸
- Translate 据此在相应 level 上分配变量访问方式

------

# 第 65 页：Managing Static Links（补充：Tr_formals）

最后一页强调：

- 当 Semant 调用 `Tr_formals(level)` 时
- 得到的是“原始参数”的 access values

------

## 这里为什么说“原始参数”

因为 static link 也是隐藏参数，但 Semant 对源语言程序只关心用户写出来的那些形参。

所以：

- `Tr_formals(level)` 会屏蔽/处理掉 static link 的内部细节
- 给 Semant 提供它真正应该看到的形式参数访问方式

------

# 全章总总结（一定要真正吃透）

下面我把全章内容重新串成一条完整逻辑链，帮助你建立体系。

------

## 一、为什么需要 Activation Record

因为：

- 每次函数调用都要保存自己的参数、局部变量、返回地址等
- 递归时同一函数会同时存在多个调用实例
- 调用与返回天然符合 LIFO

所以要用运行时栈来管理这些调用信息，每个活跃调用对应一个栈帧。

------

## 二、一个典型栈帧里有什么

通常包括：

- incoming arguments
- local variables
- return address
- temporaries
- saved registers
- outgoing arguments
- static link

但不一定所有内容都真的在栈上，也可能有一部分在寄存器里。

------

## 三、为什么需要 FP

SP 可能变化，而帧内变量需要稳定定位。
因此引入 FP 作为当前帧的固定参考点，使变量可通过“FP + offset”访问。

------

## 四、寄存器与栈帧如何分工

### 寄存器适合放：

- 参数
- 返回地址
- 返回值
- 部分局部变量和临时值

### 栈帧适合放：

- escape 变量
- 被嵌套函数访问的变量
- 数组、大对象
- spill 的值
- 保存的寄存器
- 返回地址（在非叶子函数中常见）

------

## 五、参数传递的本质问题

按值传递决定语义；
寄存器传参还是栈传参决定性能。

现代机器通常让前几个参数走寄存器，但这会引出寄存器覆盖与保存问题。

------

## 六、什么是 escape

变量一旦：

- 被引用传递
- 被取地址
- 被嵌套函数访问

它就 escape，通常必须放在 frame 里，而不能只放寄存器。

------

## 七、嵌套函数怎么访问外层变量

这就是静态作用域实现问题。

三种方法：

### 1. Static Link

每次调用额外传一个指向直接外层活动记录的指针。
访问非局部变量时沿链向上爬。

### 2. Display

维护按静态深度索引的帧指针表。
访问更快，但进入/退出过程要维护表。

### 3. Lambda Lifting

把外层变量直接改写成额外参数。
不靠链找，改靠显式传参。

------

## 八、高阶函数为什么会打破“纯栈模型”

如果语言支持：

- 嵌套函数
- 且函数可作为值返回

那么外层函数返回后，其局部变量仍可能被内部函数继续使用。
这时仅靠栈保存局部变量就不够了。

------

## 九、Tiger 编译器如何抽象这些问题

Tiger 使用两层抽象：

### 第一层：frame.h + temp.h

描述变量在 frame 或寄存器中的底层访问方式，以及 temp/label 抽象。

### 第二层：translate.h

管理源语言的嵌套层次和 static links，把“作用域问题”翻译到底层 frame 访问中。

------

## 十、几个最关键的数据类型

### `F_frame`

描述一个函数帧的布局信息。

### `F_access`

描述一个变量/参数怎么访问：

- `InFrame(offset)`
- `InReg(temp)`

### `Tr_level`

描述源语言中的嵌套层级。

### `Tr_access`

把“所在 level”与“底层 F_access”组合起来。

### `Temp_temp`

抽象寄存器名。

### `Temp_label`

抽象地址/标签名。

------

## 十一、几个最关键的接口函数

### 底层 Frame 层

- `F_newFrame`
- `F_formals`
- `F_allocLocal`

### Translate 层

- `Tr_outermost`
- `Tr_newLevel`
- `Tr_formals`
- `Tr_allocLocal`

### Temp 层

- `Temp_newtemp`
- `Temp_newlabel`
- `Temp_namedlabel`

------

# 最后给你一个“考试/复习速记版”

如果你要快速复习这一章，可以直接记下面这组问答。

------

## 1. 什么是 activation record？

函数一次调用对应的运行时记录，通常位于控制栈中，也叫 stack frame。

## 2. 为什么用栈？

函数调用/返回通常符合 LIFO。

## 3. 栈帧里有什么？

参数、局部变量、返回地址、临时值、保存寄存器、outgoing arguments、static link 等。

## 4. FP 为什么存在？

给当前帧提供稳定参考点，便于通过固定偏移访问变量。

## 5. caller-save 和 callee-save 区别？

前者由调用者保存寄存器，后者由被调用者保存。

## 6. 什么时候变量必须进 frame？

逃逸、被取地址、被嵌套函数访问、太大、数组、spill 等情况。

## 7. 什么是 escape？

变量生命周期或可访问性超出当前寄存器局部封闭范围，需要稳定地址。

## 8. static link 是什么？

每次调用传给内层函数的隐藏指针，指向其直接外层函数的最近活动记录。

## 9. static link 和 dynamic call chain 一样吗？

不一样。它反映静态嵌套关系，不是动态调用关系。

## 10. display 是什么？

按静态深度保存当前活动记录指针的全局数组。

## 11. lambda lifting 是什么？

把非局部变量改写成额外形式参数。

## 12. 为什么高阶函数会挑战栈模型？

因为外层函数返回后，其局部变量仍可能被返回出去的内部函数使用。

## 13. `F_access` 表示什么？

变量/参数的底层访问方式：在 frame 中还是在寄存器中。

## 14. `Tr_access` 为什么还要带 `level`？

因为要处理嵌套作用域和 static link 跳转层数。

## 15. temp 和 label 是干什么的？

分别是抽象寄存器名和抽象地址名，延迟到后续阶段再落实到物理位置。

------

如果你愿意，我下一条可以继续把这份笔记整理成 **更适合背诵的“精简版提纲”**，或者整理成 **“每页一句话 + 每页重点 + 高频考点”版**。