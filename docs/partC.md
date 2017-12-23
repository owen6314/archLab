# archlab Part C文档
软件52班 张迁瑞 2015013226
## rmxchg
交换寄存器与内存的值。

用法：rmxchg rA, D(rB)

### 修改过程
- 修改 sim/misc/yas-grammar.lex, 在Instr中加入rmxchg
- 修改 sim/misc/isa.h,  在itype\_t中加入I\_RMXCHG
- 修改 sim/misc/isa.c
	- 在 instruction_set[]集合中加入{"rmxchg", HPACK(I\_RMXCHG, F\_NONE), 6,  R\_ARG, 1, 1, M\_ARG, 1, 0}
	- 在 step_state函数下的need\_regids和need\_imm下加入hi0 == I\_RMXCHG,表明这个指令需要用到寄存器与立即数
	- 在 step_state函数下的switch语句中加入case I\_RMXCHG的描述

### seq rmxchg描述

rmxchg rA, D(rB)

fetch:

    icode:ifun <- M1[PC]
    rA:rB <- M1[PC + 1]
    valC <- M4[PC + 2] 
    valP <- PC + 6

decode:   

    valA <- R[rA] 
    valB <- R[rB]

 execute:

	valE <- valB + valC
 
 memory:

    valM <- M4[valE]
    M4[valE] <- valA

 write back:

    R[rA] <- valM

 PC update:

    PC <- valP

### pipeline
由于指令涉及内存的读取，可能会产生Load/Use Data Hazards,所以这条指令的数据冒险问题不能仅用转发来解决，需要加载互锁。这一点和mrmovl基本相同，仿照其进行了添加。

在测试用例中，使用rmxchg后马上使用更换后的值，证明了判断的正确性，如下图。