# lab1实验总结
## 功能总结
实现了sys_task_info系统调用,可以获取当前进程的 运行状态/运行时长/每种系统调用的使用次数.  
- 运行状态: 由于只能查询当前任务, 所以任务状态只能为 Running
- 运行时长: 在TaskControlBlock中记录开始运行的时间戳, 查询时使用 (当前时间 - 开始时间) 获取运行时长
- 系统调用的使用次数: 在中断转发时,转发前一刻将系统调用使用次数+1
## 问答题
1. 正确进入 U 态后，程序的特征还应有：使用 S 态特权指令，访问 S 态寄存器后会报错。 请同学们可以自行测试这些内容 (运行 Rust 三个 bad 测例 (ch2b_bad_*.rs) ， 注意在编译时至少需要指定 LOG=ERROR 才能观察到内核的报错信息) ， 描述程序出错行为，同时注意注明你使用的 sbi 及其版本。
  
**在低特权级使用更高特权级的指令 / 访问非法地址等都会触发异常,出core**
[ERROR] [kernel] PageFault in application, bad addr = 0x0, bad instruction = 0x80400408, core dumped.
[ERROR] [kernel] IllegalInstruction in application, core dumped.
[ERROR] [kernel] IllegalInstruction in application, core dumped.  
2. 深入理解 trap.S 中两个函数 __alltraps 和 __restore 的作用，并回答如下问题:

- L40：刚进入 __restore 时，a0 代表了什么值。请指出 __restore 的两种使用情景。  
a0 寄存器放置函数返回值, a0中的地址是 trap_handler 的返回值, 指向任务内核栈上的trap_context的地址
两种使用场景: 完成中断处理后返回用户态继续执行 / 首次被调度为Running时进入用户态

- L46-L51：这几行汇编代码特殊处理了哪些寄存器？这些寄存器的的值对于进入用户态有何意义？请分别解释。
```
ld t0, 32*8(sp)
ld t1, 33*8(sp)
ld t2, 2*8(sp)
csrw sstatus, t0 // 记录中断发生前的许多信息如特权级等
csrw sepc, t1 // 系统调用返回用户态时从何处开始执行 or 异常代码发生的位置
csrw sscratch, t2 // 保存内核栈位置
```

- L53-L59：为何跳过了 x2 和 x4？
x2: x2即为sp寄存器, 其实已经保存到sscratch了
x4: 使用不到,不用保存
```
ld x1, 1*8(sp)
ld x3, 3*8(sp)
.set n, 5
.rept 27
   LOAD_GP %n
   .set n, n+1
.endr
```
- L63：该指令( csrrw sp, sscratch, sp )之后，sp 和 sscratch 中的值分别有什么意义？
sp 和 sscratch 分别指向 内核栈/用户栈 其中的一个和另一个, 执行之后他们的指向发生交换, 此处sp指向用户栈, sscratch指向进程内核栈

- __restore：中发生状态切换在哪一条指令？为何该指令执行之后会进入用户态？
sret指令;指令执行后:CPU 会将当前的特权级按照 sstatus 的 SPP 字段设置为 U 或者 S ；CPU 会跳转到 sepc 寄存器指向的那条指令，然后继续执行。

- L13：该指令( csrrw sp, sscratch, sp )之后，sp 和 sscratch 中的值分别有什么意义？
sp 和 sscratch 分别指向 内核栈/用户栈 其中的一个和另一个, 执行之后他们的指向发生交换, 此处sscratch指向用户栈, sp指向进程内核栈

- 从 U 态进入 S 态是哪一条指令发生的？
ecall
## 对本次实验设计及难度/工作量的看法，以及有哪些需要改进的地方
难度适中,对于起步正好合适
