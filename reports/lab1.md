### 实现

主要就是在TCB结构体里加入三个成员变量，分别用来管理Task是否已经运行，Task开始时间，Task系统调用的次数。然后实现私有方法并封装成公有函数供外部调用。

## 问答题

1. 正确进入 U 态后，程序的特征还应有：使用 S 态特权指令，访问 S 态寄存器后会报错。 请同学们可以自行测试这些内容 (运行 [Rust 三个 bad 测例 (ch2b_bad_*.rs)](https://github.com/LearningOS/rCore-Tutorial-Test-2023A/tree/master/src/bin) ， 注意在编译时至少需要指定 `LOG=ERROR` 才能观察到内核的报错信息) ， 描述程序出错行为，同时注意注明你使用的 sbi 及其版本。
答：出错行为
```bash
[ERROR] [kernel] .bss [0x80263000, 0x8028c000)
[kernel] PageFault in application, bad addr = 0x0, bad instruction = 0x804003c4, kernel killed it.
[kernel] IllegalInstruction in application, kernel killed it.
[kernel] IllegalInstruction in application, kernel killed it.
[kernel] Panicked at src/trap/mod.rs:72 Unsupported trap Exception(LoadFault), stval = 0x18!
make[1]: Leaving directory '/home/wlb/codes/rust/2023a-rcore-aLingYun/os'
```
使用的`RustSBI version: 0.3.0-alpha.2`

2. 深入理解 [trap.S](https://github.com/LearningOS/rCore-Tutorial-Code-2023A/blob/ch3/os/src/trap/trap.S) 中两个函数 `__alltraps` 和 `__restore` 的作用，并回答如下问题:
    
    1. L40：刚进入 `__restore` 时，`a0` 代表了什么值。请指出 `__restore` 的两种使用情景。
    答：`a0`保存了调用`__restore`时的第一个参数。 在这里代表了`TrapContext`上下文的地址。用于从特权态中恢复。
        
    2. L43-L48：这几行汇编代码特殊处理了哪些寄存器？这些寄存器的的值对于进入用户态有何意义？请分别解释。
        
        ld t0, 32*8(sp)
        ld t1, 33*8(sp)
        ld t2, 2*8(sp)
        csrw sstatus, t0
        csrw sepc, t1
        csrw sscratch, t2
    答： 特殊处理了`sstatus, sepc, sscratch`三个`csr`寄存器， 必须正确处理这些状态寄存器才能完成特权级的切换与恢复。
        1. `sstatus`标志了`CPU`所处的特权级
        2. `sepc`标志了`Trap`结束后的下一条指令地址
        3. `sscratch`指向当前应用的内核栈栈顶
        
    3. L50-L56：为何跳过了 `x2` 和 `x4`？
        
        ld x1, 1*8(sp)
        ld x3, 3*8(sp)
        .set n, 5
        .rept 27
           LOAD_GP %n
           .set n, n+1
        .endr
    答： `x2`是栈指针， 此时它指向内核栈。 而用户栈指针已经被保存到`sscratch`。 `x4`一般不使用
        
    4. L60：该指令之后，`sp` 和 `sscratch` 中的值分别有什么意义？
        
        csrrw sp, sscratch, sp
    答：`sp`是用户栈指针， `sscratch`是内核栈指针
        
    5. `__restore`：中发生状态切换在哪一条指令？为何该指令执行之后会进入用户态？
    答：发生在`sret`指令， 返回到用户态的程序内容中继续执行
        
    6. L13：该指令之后，`sp` 和 `sscratch` 中的值分别有什么意义？
        
        csrrw sp, sscratch, sp
    答： 执行后`sp`是内核栈指针， 而`sscratch`是用户栈指针
        
    7. 从 U 态进入 S 态是哪一条指令发生的？
    答：`ecall`, 调用后进入`stvec`寄存器的`Trap`地址开始执行

## 荣誉准则

1. 在完成本次实验的过程（含此前学习的过程）中，我曾分别与 **以下各位** 就（与本次实验相关的）以下方面做过交流，还在代码中对应的位置以注释形式记录了具体的交流对象及内容：
    
    > _无
    
2. 此外，我也参考了 **以下资料** ，还在代码中对应的位置以注释形式记录了具体的参考来源及内容：
    
    > _《[rCore-Tutorial-Guide-2023A](https://learningos.github.io/rCore-Tutorial-Guide-2023A/)》_
    > _《[rCore-Tutorial-Book-v3](https://rcore-os.github.io/rCore-Tutorial-Book-v3/)》_

3. 我独立完成了本次实验除以上方面之外的所有工作，包括代码与文档。 我清楚地知道，从以上方面获得的信息在一定程度上降低了实验难度，可能会影响起评分。

4. 我从未使用过他人的代码，不管是原封不动地复制，还是经过了某些等价转换。 我未曾也不会向他人（含此后各届同学）复制或公开我的实验代码，我有义务妥善保管好它们。 我提交至本实验的评测系统的代码，均无意于破坏或妨碍任何计算机系统的正常运转。 我清楚地知道，以上情况均为本课程纪律所禁止，若违反，对应的实验成绩将按“-100”分计。