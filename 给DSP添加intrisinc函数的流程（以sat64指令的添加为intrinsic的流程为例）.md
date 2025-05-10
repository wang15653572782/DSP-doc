**给DSP添加intrisinc函数的流程（以sat64指令的添加为intrinsic的流程为例）**

1.在clang前端定义intrinsic函数（ clang/include/clang/Basic/BuiltinsDSP.def）：

![image-20250509114738840](C:\Users\WCF\AppData\Roaming\Typora\typora-user-images\image-20250509114738840.png)

2.添加语法树到IR的转换（clang/lib/CodeGen/CGBuiltin.cpp）：



![image-20250509114929538](C:\Users\WCF\AppData\Roaming\Typora\typora-user-images\image-20250509114929538.png)

3.编写新增加的intrinsic的语义描述（ llvm/include/llvm/IR/IntrinsicsDSP.td）：定义intrinsic函数的属性和数据类型。

![image-20250509115057304](C:\Users\WCF\AppData\Roaming\Typora\typora-user-images\image-20250509115057304.png)

4.在后端td文件里面增加后端指令描述和pattern（llvm/lib/Target/DSP/DSPInstrInfo.td）：

![image-20250509115517424](C:\Users\WCF\AppData\Roaming\Typora\typora-user-images\image-20250509115517424.png)







## [添加LLVM Intrinsic](https://blog.csdn.net/tristan_tian/article/details/84309989)



在LLVM中添加Intrinsic函数可以为特定的ISA增加新的指令。以下是一个简单的示例，展示如何在LLVM中添加一个Intrinsic函数。

示例代码

> \#include <stdlib.h>
>
> \#include <stdio.h>
>
> int main() {
>
> int a = 2;
>
> int b = 6;
>
> int c = __builtin_x86_max_qb(a, b);
>
> printf("%d\n", c);
>
> return 0;
>
> }

步骤

1. **在Clang中增加Intrinsic函数**： 在*clang/include/clang/Basic/BuiltinsX86.def*中增加新的Intrinsic函数[1](https://blog.csdn.net/tristan_tian/article/details/84309989)。 *BUILTIN(__builtin_x86_max_qb, "iii", "")*
2. **在LLVM IR中定义Intrinsic**： 在*llvm/include/llvm/IR/IntrinsicsX86.td*中定义新的Intrinsic[2](https://blog.csdn.net/tristan_tian/article/details/82221590)。 *let TargetPrefix = "x86" in { def int_x86_max_qb: GCCBuiltin<"__builtin_x86_max_qb">, Intrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty], [Commutative]>; }*
3. **在X86后端增加指令**： 在*llvm/lib/Target/X86/X86ISelLowering.h*中增加新的ISD节点[3](https://blog.csdn.net/tristan_tian/article/details/84309989)。 *def X86max_qb : SDNode<"X86ISD::max_qb", SDTBinaryArithWithFlags, [SDNPCommutative]>;*
4. **在X86InstrInfo.td中定义指令格式**： 在*llvm/lib/Target/X86/X86InstrInfo.td*中定义指令的输出格式[3](https://blog.csdn.net/tristan_tian/article/details/84309989)。 *def max_qb : I<0xF0, MRMSrcReg, (outs GR32:$dst), (ins GR32:$src1, GR32:$src2), "max_qb\t{$dst, $src1, $src2|$dst, $src2, $src1}", [(set GR32:$dst, EFLAGS, (X86max_qb GR32:$src1, GR32:$src2))]>, Sched<[WriteIMul32Reg]>, OpSize32;*
5. **编译和测试**： 使用以下命令编译并生成LLVM IR文件[1](https://blog.csdn.net/tristan_tian/article/details/84309989)。 *clang -emit-llvm -O0 test.c -c -o test.bc llc test.bc*