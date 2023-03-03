# RISC-V-Trace-Control-Interface-problem

* p6`The Trace Encoder transmits trace messages to a Trace Sink.`

  sink可能指的是输出packet的目的地p7

* p6`In multi-core systems, each core has its own Trace Encoder, and typically all will connect to a Trace Funnel that aggregates trace data from multiple sources and sends the data to a single destination`

  之前不是说每个hart都要一个encoder(还是说是interface√)吗？

  另外Funnel指的是将多个encoder来源的数据会聚成一个流输出p7

* p6`Implementation choices include whether to support branch trace, data trace, instrumentation trace, timestamps, external triggers, various trace sink types, and various optimization tradeoffs between gate count, features, and bandwidth requirements.`

  目前仅仅先实现一个分支trace

* p7`WARL - Write any, read legal. If a non-legal value is written, the writen value must be ignored and register will keep previous, legal value`

  如果写入的值不合法，其会保持之前的合法值

* p8`The Trace Encoder control interface consists of a set of 32-bit registers occupying up to a 4K-byte space`

  **trace encoder control-interface**大小4KB 每个寄存器32位

* p8`The Trace Encoder control registers would typically be accessed by a debugger through the debug module. The Trace Encoder may or may not also be accessible through loads and stores performed by one or more harts in the system. Typically, the Trace Encoder connects to the system bus as a peripheral device, but it may use a dedicated bus connection from the Debug Module, or could attach to the DMI bus defined in the RISC-V Debug Specification.`

  `Additional control path(s) may also be implemented, such as a dedicated debug bus or messagepassing network`

  **trace encoder control registers** 通常 被调试器的调试模块访问

  **trace encoder** 不一定（可能有）通过一个或多个 harts 的load/store访问

  **trace encoder**视为外设，可以通过多种bus来连接到系统

  **control path** 也有 多种实现方法

  **note：这里说的是不是都是一个东西，甚至包括控制通路也是一种访问方法？**

* p8`Mapping the control interface into physical memory accessible from a hart allows that hart to manage a trace session independently from an external debugger. A hart may act as an internal debugger or may act in cooperation with an external debugger. Two possible use models are collecting crash information in the field and modifying trace collection parameters during execution. If a system has physical memory protection (PMP), a range can be configured to restrict access to the trace system from hart(s).`

  将**control interface**映射到物理内存，来允许hart独立于外部调试器管理一个trace会话。

  如果有**PMP**，为什么要配置范围来限制hart访问trace

* p8`There is typically one Trace Encoder per core. A core with multiple harts (i.e., multi-threaded) will generate messages with a field indicating which hart is responsible for that message. Cores capable of retiring more than one instruction per cycle are typically accommodated with a single Trace Encoder, though this is not required.`

  每个core都有一个encoder，hart通过某个字段表明这是自己生成的消息。**每个周期能退休多条指令的core通知使用单个encoder？**

* p8`The Trace Funnel is a variant of the Trace Encoder and shares many of the same control registers. Each Trace Encoder and the Trace Funnel has its own set of control registers in its own register block`

  Funnel是encoder的一个变体，和encoder有许多相同的控制寄存器，但是也都有自己的控制寄存器集。

* p8`The 4K block occupied by a Trace Encoder or Trace Funnel is divided into eight sections of 256 bytes. Section 0 is required and is used for local control registers. Other sections are used for control registers of trace components that are conceptually separate, even if they are physically part of the Trace Encoder/Funnel. Examples of possible subcomponents are:`

  **encoder/funnel占据的4K的空间被分为8个section，每个256B**

  第0section用于控制本地寄存器。

  其他用于控制trace的组件

* p8`Registers in the 4K range that are not implemented read as 0 and ignore writes`

  未实现的4K范围内的寄存器读取值为0，并忽略写入操作。

* p9

  | Address Offset | Trace Encoder | Trace Funnel | Compliance | 描述                             |
  | -------------- | ------------- | ------------ | ---------- | -------------------------------- |
  | 0x000          | teControl     | tfControl    | 必需的     | trace encoder/funnel的控制寄存器 |
  | 0x004          | teImpl        | tfImpl       | 必需的     | trace encoder/funnel的实现信息   |

* p10`Each program counter discontinuity results in one trace message, either a Direct or Indirect Branch Message. Linear instructions (or sequences of linear instrucions) do not result in any trace messages/packets.`

  每个PC不连续都会产生一条message

  顺序指令不产生任何message

* p10`Indirect Branch Messages normally contain a compressed address to reduce bandwidth. The TE emits a Branch With Sync Message containing the complete instruction address under certain conditions. This message type is a variant of the Direct or Indirect Branch Message and includes a full address and a field indicating the reason for the Sync.`

  间接分支message 中会包含一个压缩地址（增量）来减少带宽

  trace encoder 也会发出同步分支message 包含完整的指令地址，其是直接/间接分支message的变体，包含完整地址，还有同步的原因

* p10`Both the E-Trace Processor Trace Specification and the Nexus standard define systems of messages intended to improve compression by reporting only whether conditional branches are taken by encoding each branch outcome is encoded in single bit. The destinations of non-inferable jumps and calls are reported as compressed addresses. Much better compression can be achieved, but an Encoder implementation will typically require more hardware.`

  报告分支时，仅需要1bit来报告是否发生了跳转

  而不可推断的跳转/调用 则报告压缩的地址

* p10`Several other optimizations are possible to improve trace compression. These are optional for any Trace Encoder and there should be a way to disable optimizations in case the trace system is used with code that does not follow recommended API rules. Examples of optimizations are a Return-address stack, Branch repetition, Statically-inferable jump, and Branch prediction.`

  有些优化，是可选的。但是应该设置一个禁用的方法，万一某代码不遵循推荐的API

  包括：返回地址栈、分支重复、静态可推断的跳转、分支预测。

* p10`The Trace Encoder transmits completed messages to a Trace Sink. This specification defines a number of different sink types, all optional, and allows an implementation to define other sink types. A Trace Encoder must have at least one sink attached to it.`

  所有的sink都是可选的，但是至少包含一种。

* p12`Many features of the Trace Encoder are optional. In most cases, optional features are enabled using a WARL (write any, read legal) register field. A debugger can determine if an optional feature is present by writing to the register field and reading back the result.`

  可选的特性通过一个WARL寄存器域来启用。

  调试器可以通过写入寄存器字段并读回结果判断是否存在可选的特性

  **teControl**0x000

  | bit   | field             | RW   | Reset |
  | ----- | ----------------- | ---- | ----- |
  | 0     | teActive          | RW   | 0     |
  | 1     | teEnable          | RW   | 0     |
  | 2     | teTracing         | RW   | 0     |
  | 3     | teEmpty           | R    | 1     |
  | 4-6   | teInstMode        | WARL | SD    |
  | 7-12  | ——                | WARL | SD    |
  | 11    | teInstTrigEnable  | WARL | 0     |
  | 12    | teInstStallDelta  | R    | 0     |
  | 13    | teInstStallEnable | WARL | SD    |
  | 14    | teStopOnWrap      | WARL | SD    |
  | 15    | teInhibitSrc      | WARL | SD    |
  | 16-17 | teSyncMode        | WARL | SD    |
  | 18-19 | Reserved          | ——   | 0     |
  | 20-23 | teSyncMax         | WARL | SD    |
  | 24-26 | teFormat          | WARL | SD    |
  | 28-31 | teSink            | WARL | SD    |
  
  SD:其复位时的值必须一样
  
  * **teActive**
  
    `Master enable for given TE. 0 resets the TE and it may be powered down or clocks may be gated off. Hardware may take an arbitrarily long time to process power-up and power-down and will indicate completion when the read value of this bit matches what was written. When teActive=0, all other TE registers may not be accessible.`
  
    主使能 对于给定的trace encoder	允许用户启用和禁用整个模块
  
    为0时，重置trace encoder，可能也会关闭电源，时钟截至？同时此时te寄存器不可访问
  
    为1时，trace encoder启用，寄存器可访问。
  
    当该比特的读值和写入值匹配时指示完成
  
  * **teEnable**
  
    `1=TE enabled. Allows teTracing to turn all tracing on and off. Setting teEnable to 0 flushes any queued trace data to the designated sink. This bit can be set to 1 only by direct write to it.`
  
    为1时，trace encoder使能，允许**teTracing**打开或关闭。
  
    为0时，将任何队列中的数据刷到指定的sink。
  
    只有直接写入时，该比特位才能设置为1。
  
  * **teTracing**
  
    `1=Trace is being generated. Written from tool or controlled by triggers. When teTracing=1, trace data may be subject to additional filtering in some implementations (additional teInstruction modes or data tracing).`
  
    为1时。指示正在生成trace。由工具写，或者是由trigger控制。
  
    在某些实现中，有可能会有额外的模式，如过滤，或者数据trace
  
  * **teEmpty**
  
    `Reads as 1 when all generated trace has been emitted.`
  
    当所有生成的trace都被发出去时，读值为1
  
  * **teInstMode**
  
    `Main instruction trace generation mode`
  
    主要的指令trace生成模式/主指令trace生成模式？
  
    0：禁用指令trace
  
    1-2：保留给分支trace的子集（如 定期PC采样）
  
    3：使用分支trace生成指令trace（每个跳转的分支？（还是指跳转？）都生成trace）
  
    4-5：保留给分支历史trace的子集
  
    6：生成非优化的指令分支历史trace（每个分支添加单个历史位）
  
    7：生成优化的指令trace（如果 **teInstFeatures**存在，则定义指令trace的特性和优化）
  
  * **——**
  
    `Vendor-specific controls`
  
    特定于供应商的控制
  
  * **teInstTrigEnable**
  
    `Global enable/disable for instruction trace triggers`
  
    指令trace trigger的全局 使能/禁用
  
  * **teInstStallDelta**
  
    `Read as 1 if stall happened. Clears to 0 on reading`
  
    stall（停顿？）发生，则读值为1
  
    读取后清空为0
  
  * **teInstStallEnable**
  
    `0 = If TE cannot send a message, an overflow is generated when trace is restarted.
    1 = If TE cannot send a message, the core is stalled until it can.`
  
    两个都是trace encoder无法发送消息。
  
    为0时，则重启trace时发生溢出。当TE无法发送跟踪消息时，缓存中的消息数量将增加，直到缓存达到最大容量时，将生成溢出消息。这些溢出消息将在重新启动跟踪时被发送。
  
    为1时，核心停止，直到trace可以发送消息。当TE无法发送跟踪消息时，核心将停止执行直到缓存有足够的空间可以发送消息。
  
  * **teStopOnWrap**
  
    `Disable trace (teEnable → 0) when circular buffer fills for the first time.`
  
    该选项表示当循环缓冲区首次填满时禁用跟踪，即当缓冲区中的跟踪消息数量达到缓冲区容量时，禁用跟踪。
  
    循环缓冲区是一种循环使用的数据结构，当数据写满缓冲区后，后续的写操作将覆盖最早写入的数据。这个选项的目的是在跟踪消息数量达到缓冲区容量时停止跟踪，以避免新消息覆盖最早的消息。此后，可以根据需要将缓冲区中的跟踪消息传输到外部处理器。
  
    这种行为可以确保最早的跟踪消息不会被覆盖，但也可能导致跟踪信息不完整，因为未能记录所有的跟踪消息。
  
  * **teInhibitSrc**
  
    `1=Disable source field in trace messages. Unless disabled, a trace source field (of teImpl.nSrcBits) is added to every trace message to indicate which TE generated each message. If teImpl.nSrcBit is 0, this bit is not active.`
  
    为1时，禁用trace消息中的源字段。
  
    除非禁用，否则跟踪源字段(teImpl.nSrcBits)会添加到每个跟踪消息中，以表明每个消息是由哪个TE生成的。
  
    如果teImpl.nSrcBit为0，表示该比特位不活跃
  
  * **teSyncMode**
  
    `Select periodic synchronization mechanism. At least one non-zero mechanism must be
    implemented.`
  
    选择定时同步机制，至少实现一个非零的
  
    0：关闭
  
    1：计数trace message/packet
  
    2：计数时钟周期
  
    3：计数指令的半字数
  
  * **Reserved**
  
  * **teSyncMax**
  
    `The maximum interval (in units determined by teSyncMode) between synchronization messages/packets. Generate synchronization when count reaches 2^(teSyncMax + 4). If synchronization packet is generated from another reason internal counter should be reset.`
  
    同步message/packet之间的最大间隔（单位由**teSyncMode**决定）
  
    当计数到达2<sup>(teSyncMax+4)</sup>时产生同时
  
    如果同步packet由其他原因产生，则内部的计数器需要重置
  
  * **teFormat**
  
    `Trace recording format`
  
    trace 记录格式
  
    0：E-trace spec定义的格式
  
    1：Nexus message(6 MDO + 2 MSEO)
  
    2-6：保留给未来的格式
  
    7：特定于供应商的格式
  
  * **teSink**
  
    `Which sink to send trace to.`
  
    将trace发送到哪一个sink
  
    0-3：保留
  
    4：SRAM sink
  
    5：ATB sink
  
    6：PIB sink
  
    7：System Memory sink
  
    8：Funnel sink
  
    9-11：保留给未来的sink类型
  
    12-15：保留给特定于供应商的sink类型
  
  **teImpl**0x004
  
  | bit   | field         | RW   | Reset |
  | ----- | ------------- | ---- | ----- |
  | 0-3   | teVersion     | R    | 1     |
  | 4     | hasSRAMSink   | R    | SD    |
  | 5     | hasATBSink    | R    | SD    |
  | 6     | hasPIBSink    | R    | SD    |
  | 7     | hasSBASink    | R    | SD    |
  | 8     | hasFunnelSink | R    | SD    |
  | 9-11  |               | R    | 0     |
  | 12-15 |               | R    | SD    |
  | 16-19 |               | —    | —     |
  | 20-23 | teSrcID       | WARL | SD    |
  | 24-26 | teSrcBits     | WARL | SD    |
  | 27    |               | —    | —     |
  | 28-31 |               | —    | —     |
  
  * teVersion
  
    `TE Version (value 0 means legacy version - see 'Legacy Interface Version' chapter at the end)`
  
    trace encoder版本
  
    为0时表示遗留版本
  
  * hasSRAMSink
  
    `1 if this TE has an on-chip SRAM sink. Size of SRAM may be determined by writing all 1s to teRamWP, then reading the value back.`
  
    为1时表示这个trace encoder有一个片上SRAM sink
  
    SRAM的大小可以通过向**reRamWP**写全1，然后读出来得到
  
  * hasATBSink
  
    `1 if this TE has an ATB sink.`
  
    为1时表示这个trace encoder由一个ATB sink
  
  * hasPIBSink
  
    `1 if this TE has an off-chip trace port via a Pin Interface Block (PIB)`
  
    为1时表示这个trace encoder有一个 芯片外的trace端口 通过一个引脚接口块 
  
  * hasSBASink
  
    `1 if this TE has an on-chip system memory bus master trace sink.`
  
    为1时表示这个trace encoder有一个片上系统内存总线 主trace sink
  
  * hasFunnelSink
  
    `1 if this TE feeds into a trace funnel device.`
  
    为1时表示这个trace encoder需要将trace发送到funnel设备进行处理和聚合
  
  * 9-11
  
    `Reserved for future sink types`
  
    保留给未来的sink类型使用
  
  * 12-15
  
    `Reserved for vendor-specific sink types`
  
    保留给特定于供应商的sink类型
  
  * 16-19
  
    `Reserved for vendor-specific features`
  
    保留给特定于供应商的功能
  
  * teSrcID
  
    `This TE’s source ID. If teSrcBits>0 and trace source is not disabled by teInhibitSrc, then messages will all include a trace source field of teSrcBits bits. Messages from this TE will use this value as trace source field. May be fixed or variable.`
  
    当前trace encoder的源id
  
    如果**teSrcBits>0** && trace source 没有被**teInhibitSrc**禁止
  
    那么message都会包含一个trace source域, **teSrcBits**位
  
    这个值可以是固定的，也可以是可变的
  
  * teSrcBits
  
    `The number of bits in the trace source field, unless disabled by teInhibitSrc. May be fixed or variable.`
  
    trace message中trace source域的位数。除非被**teInhibitSrc**禁止
  
    可以是固定的，或可变的
  
  * 27
  
    保留
  
  * 28-31
  
    保留给特定于供应商的功能
