# archlab Part A文档
软件52班 张迁瑞 2015013226
## sum.ys
第一个程序，非递归地计算链表的和。

这是我和Y86的第一次接触，我在这个程序上花费了很长时间。但是它也让我对Y86有了总体上的认识。

这个程序的完成主要得益于之前对于X86的学习，Y86与X86的编程思想基本相同，只是在某些具体语句上会有差别，按照书上的提示，可以先用编译器编译成X86的代码，之后再手动改成符合Y86的形式。不过，为了防止考试的时候出现这样的题目自己不会做，我还是选择手动将C代码“翻译”成了Y86代码。稍微有点麻烦，不过也很有收获。

```
	 		.pos 0
init:		irmovl 	Stack, %esp
			irmovl 	Stack, %ebp
			call   	Main
			halt				  	

			.align 	4			  	# test data
ele1:		.long 	0x00a
			.long 	ele2
ele2:		.long 	0x0b0
			.long 	ele3
ele3:		.long 	0xc00
			.long 	0

Main:		pushl 	%ebp
			rrmovl 	%esp, %ebp
			irmovl	ele1, %edx		# edx=ele1
			pushl	%edx			# set edx as parameter of sum_list
			call	sum_list
			rrmovl	%ebp, %esp
			popl	%ebp
			ret
	
sum_list:   pushl	%ebp
			rrmovl	%esp, %ebp
			xorl	%eax, %eax		# init result value  对应C代码 int val=0
			mrmovl	8(%ebp), %edx	# init edx as the start of list
			andl	%edx, %edx		# test if array length is 0
			je	End					# if array length is 0, stop calculating

Loop:		mrmovl	(%edx), %esi	# esi = current value  对应C代码中循环
			addl	%esi, %eax		# add current value to total value 对应C代码val+=ls->val
			irmovl	$4, %edi		# edi=4
			addl	%edi, %edx		# set edx as next
			mrmovl  (%edx), %esi	# get next value to esi	对应C代码 ls = ls -> next
			#andl	%esi, %esi		
			rrmovl  %esi, %edx
			andl	%edx, %edx		# test if next is 0		对应C代码循环中的判断条件部分while(ls)
			je	End
			jmp	Loop
		
End:		rrmovl	%ebp, %esp
			popl	%ebp
			ret


# the stack starts here and grows to lower address
			.pos 0x100
Stack: 
```

## rsum.ys
递归地计算列表的和。在sum.ys的基础上，这个函数的完成很容易。将函数的循环部分改为递归即可。

唯一值得强调的是，使用递归的时候要注意寄存器状态的保存，要选择合适的时机将它们压入栈中，否则它们的值就会在递归中被破坏。
```
	 		.pos 0
init:		irmovl 	Stack, %esp
			irmovl 	Stack, %ebp
			call   	Main
			halt				  	

			.align 	4			  	# test data
ele1:		.long 	0x00a
			.long 	ele2
ele2:		.long 	0x0b0
			.long 	ele3
ele3:		.long 	0xc00
			.long 	0

Main:		pushl 	%ebp
			rrmovl 	%esp, %ebp
			xorl	%eax, %eax
			irmovl	ele1, %edx		# edx=ele1
			pushl	%edx			# set edx as parameter of sum_list
			call	rsum_list
			popl	%edx
			rrmovl	%ebp, %esp
			popl	%ebp
			ret

rsum_list:  pushl   %ebp
            rrmovl  %esp, %ebp
            mrmovl  8(%ebp), %edx   # init edx as the start of list
            andl    %edx, %edx      # test if array length is 0
            je      End             # if array length is 0, stop calculating 对应C代码 if(!ls) return 0;

recursive:
            mrmovl  (%edx), %esi	# esi = current value 对应C代码 int val = ls->val
            pushl	%esi			# save esi
            irmovl  $4, %edi		# edi=4
            addl    %edi, %edx		# set edx as next
            mrmovl  (%edx), %esi	# get next value to esi
            rrmovl  %esi, %edx		
            pushl   %edx			
            call    rsum_list		# recursively call rsum_list  对应C代码 int rest = rsum_list(ls->next)
            popl	%edx
            popl	%esi			# recover esi
			addl    %esi, %eax		# add current value to total value	对应C代码 return val + rest

End:        rrmovl  %ebp, %esp
            popl    %ebp
            ret


# the stack starts here and grows to lower address
                .pos 0x100
Stack:

```
## copy.ys
进行给定长度区块的复制，并计算checksum。

有了前两个程序的积累，这个程序的完成也很轻松，本质上就是一个循环，和第一个程序很类似。

```
            .pos 0
init:		irmovl  Stack, %esp
			irmovl  Stack, %ebp
			call    Main
			halt
	
            .align 4 		# src and dest
src:
			.long 0x00a
			.long 0x0b0 
			.long 0xc00	
dest:
			.long 0x111
			.long 0x222 
			.long 0x333

Main:		pushl	%ebp
			rrmovl	%esp, %ebp
			irmovl  src, %edx		# push parameters into stack
			pushl	%edx
			irmovl	dest, %edx
			pushl	%edx
			irmovl  $3, %edx		# push length of area
			pushl	%edx
			call	copy_block
			rrmovl	%ebp, %esp
			popl	%ebp
			ret

copy_block:	pushl	%ebp
			rrmovl	%esp, %ebp
			mrmovl	8(%ebp), %ecx	# ecx=length
			xorl	%eax, %eax		# init result, eax=result 对应C代码 int result=0
			mrmovl	12(%ebp), %edx  # edx=dest
			mrmovl  16(%ebp), %ebx  # ebx=src
			andl	%ecx, %ecx		# if length=0, return
			je	End

Loop:		mrmovl  (%ebx), %esi	# 对应C代码 int val = src
			rmmovl	%esi, (%edx)	# copy src to dest 对应C代码	*dest=val
			xorl	%esi, %eax		# result^=val 对应C代码 result^=val
			irmovl	$4, %edi		# edi = 4
			addl	%edi, %ebx		# src++	 对应C代码src++
			addl	%edi, %edx		# dest++ 对应C代码dest++
			irmovl  $1, %esi		# esi=1
			subl	%esi, %ecx		# len--  对应C代码len--
			andl	%ecx, %ecx		# check if len is zero 对应C代码(len>0)
			jne	Loop						
		
End:		rrmovl  %ebp, %esp
			popl	%ebp
			ret

	    	.pos 0x100
Stack:         

```