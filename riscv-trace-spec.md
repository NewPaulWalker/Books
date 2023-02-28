# riscv-trace-spec

### problem 1

p15

`This works by tracking execution from a known start address and sending messages about the address deltas taken by the program`

从已知的地址追踪执行是指程序第一条指令的地址？还是任何一个给定的能够跑到的地址？

什么叫做程序获取的地址增量消息？谁发送？

### problem 2

p15

`in most implementations, it will supply a large amount of data (instruction address, instruction type, context information, ...) for each core execution clock cycle`

for each core execution clock cycle是什么意思？

### problem 3

p16

`Because of this, there is no need to report sequential instructions in the trace, only whether the branches were taken or not and the address of taken indirect branches or jumps. If the program counter is changed by an amount that cannot be determined from the execution binary, the trace decoder needs to be given the destination address (i.e. the address of the next valid instruction). Examples of this are indirect branches or jumps, where the next instruction address is determined by the contents of a register rather than a constant embedded in the program binary.`

* 顺序指令不用trace

* 分支指令是否跳转

  ?

  bne

  beq

  bge

  bgeu

  blt

  bltuf

  jal

* 间接分支/跳转的地址，以及一切不能从二进制文件得到（可能从寄存器得到）的PC增量的下一条有效地址

  ?

  jalr

  ecall

  ebreak

  sret

  mret

  异常

  中断

### problem 4

p16

`The decoder generally does not know where an interrupt occurs in the instruction sequence, so the trace encoder must report the address where normal program flow ceased, as well as give an indication of the asynchronous destination which may be as simple as reporting the exception type. When an interrupt or exception occurs, or the processor is halted, the final instruction retired beforehand must be included in the trace.`

需要trace三个东西：异常发生前最后一条指令、异常类型、异常后第一条指令

### problem 5

p19

`This chapter describes the fields required to control the Trace Encoder. How fields are organized and accessed (e.g packet based or memory mapped) is outside the scope of this document.`

目前仅仅谈论trace encoder的控制需求，尚不谈论这些域的组织方式和访问方式

之后可以参考https://github.com/riscv-non-isa/tg-nexus-trace/blob/master/docs/RISC-V-Trace-Control-Interface.adoc#register-map实现组织和访问方式

为尽快熟悉流程，先仅考虑M（Mandatory），指令trace的必做部分

### problem 6

p19

`Other abbreviations used in the tables are:`
`W column heading indicates field width (in bits)`

`G column heading indicates field group`

`RW column heading indicates whether bit is read-only (R), or read-write (RW). For the latter, it is allowed but not required that the bit be writable`
`Rst column heading indicates field reset value. SD in this column indicates a system dependent reset value`

* W:域的宽度
* G:所属的组，目前仅考虑M
* RW:读/读写
* Rst:复位的值，SD表示系统依赖（system dependent）复位值

### problem 7

p21

| field      | W    | G    | RW   | Rst  |
| ---------- | ---- | ---- | ---- | ---- | 
| Active     | 1    | M    | RW   | 0    | 
| teEnable   | 1    | M    | RW   | 0    | 
| iTracing   | 1    | M    | RW   | 0    | 
| ResyncMode | 2    | M    | RW   | SD   |

* Active:

  `Primary enable for trace system. When 0, the trace system may have clocks gated off or be powered down, and other register locations may be inaccessible. Hardware may take arbitrarily long to process powerup or power-down and will indicate completion when the read value of this bit matches the value written.`

  trace系统的主使能。

  0:什么叫做clocks gated off?其他寄存器位置不可访问？

  什么叫做读值和写值匹配时？WARL, WLRL?

* teEnable:

  `Primary trace enable. Trace can be enabled via iTracing or dTracing when 1. Setting to 0 flushes any queued trace data to the designated sink.`

  主trace使能。

  1:可以启用指令trace或者数据trace

  0:是指输出成packet？还是丢弃？

* iTracing:

  `Instruction trace enable. When 1, trace will be generated, subject to any optional filtering. May be written by software, or via triggers if iTrigEnable is 1.`

  指令trace使能。

  1:启动指令trace

  什么是受任何可选筛选影响？

  软件可写？或者通过trigger如果iTriEnable为1

* ResyncMode:

  选择重同步机制，至少需要实现一种非零的

  0:关闭

  1:计数packet

  2:计数时钟周期

  3:计数instruction (16-bit) half-words，指令的半字数，C扩展计数1，其他指令计数2。





