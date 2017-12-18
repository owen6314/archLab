# archlab Part B文档
软件52班 张迁瑞 2015013226
## Seq程序描述
### iaddl
对iaddl指令的描述如下：

iaddl **V** **rb**

fetch:

    icode:ifun <- M1[PC]
    rA:rB <- M1[PC+ 1]
    valC <- M4[PC + 2]
    valP <- PC + 6  

 decode:

    valB <- R[rb]

 execute:

    valE <- valB + valC
    set CC

 memory:
 write back:

    R[rb] <- valE

 PC update:

    PC <- valP

### leave
对leave的描述如下:

leave

 fetch:

    icode:ifun <- M1[PC] 
    valP <- PC + 1

 decode:   

	# 其实这里的valA最后并没有用上，指示仿照popl的写法加入了valA的设置
    valA <- R[%esp]
    valB <- R[%ebp]    

 execute:

    valE <- valB + 4

 memory:

    valM <- M4[valB]

 write back:

    R[%esp] <- valE
    R[%ebp] <- valM

 PC update:

    PC <- valP

## Pipeline

## 难点

## 收获与感受
没有别的感想，做完这个实验，只感觉对CSAPP的作者五体投地。

如果其他的课程也有这样既有乐趣又能学到很多知识的大作业就好了。

最主要的收获，是通过这个实验加深了对于Y86（包括顺序和pipeline模式）的印象。本来以为自己对于硬件描述语言不熟悉，做起来可能会很困难，但是在程序中的注释的帮助下，发现如果理解了问题所在，这个程序的完成只需要填几个空而已。


