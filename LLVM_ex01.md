1. clang -S test.c
   -S 代表 Only run preprocess and compilation steps（只跑預處理和編譯階段，且只編成組語）（不會最佳化？)

   test.c 如下

   ```c
   int foo()
   {
   	int a, b, c;
   	a = 10;
   	b = 20;
   	c = a + b;
   	return c;
   }
   ```

   會生出test.s如下

   ```assembly
   	.text
   	.file	"test.c"
   	.globl	foo                     # -- Begin function foo
   	.p2align	1
   	.type	foo,@function
   foo:                                    # @foo
   .Lfoo$local:
   # %bb.0:
   	addi	sp, sp, -32
   	sd	ra, 24(sp)
   	sd	s0, 16(sp)
   	addi	s0, sp, 32
   	addi	a0, zero, 10
   	sw	a0, -20(s0)
   	addi	a0, zero, 20
   	sw	a0, -24(s0)
   	lw	a0, -20(s0)
   	lw	a1, -24(s0)
   	add	a0, a0, a1
   	sw	a0, -28(s0)
   	lw	a0, -28(s0)
   	ld	s0, 16(sp)
   	ld	ra, 24(sp)
   	addi	sp, sp, 32
   	ret
   .Lfunc_end0:
   	.size	foo, .Lfunc_end0-foo
                                           # -- End function
   	.ident	"clang version 11.0.0 (https://github.com/llvm/llvm-project.git 05c0d3491822b3a74f49be2fe8c8273e436ab7ec)"
   	.section	".note.GNU-stack","",@progbits
   	.addrsig
   ```

2. clang -Xclang -disable-O0-optnone -c -emit-llvm test.c

   | 參數               | 功能                                               |
   | ------------------ | -------------------------------------------------- |
   | -Xclang <arg>      | Pass <arg> to the linker                           |
   | disable-O0-optnone | 關閉最佳化                                         |
   | -c                 | 後面接一些編譯相關的參數像-emit-llvm再接檔名test.c |
   | -emit-llvm         | compiling to bitcode                               |

   會產生test.bc(bitcode)

3. llvm-dis test.bc

   llvm-dis（全名LLVM diassembler）把bitcode file轉成人類看得懂的LLVM assembly language

   下面是test.ll檔

   ```assembly
   ; ModuleID = 'test.bc'
   source_filename = "test.c"
   target datalayout = "e-m:e-p:64:64-i64:64-i128:128-n64-S128"
   target triple = "riscv64-unknown-unknown-elf"
   
   ; Function Attrs: noinline nounwind
   define dso_local signext i32 @foo() #0 {
     %1 = alloca i32, align 4
     %2 = alloca i32, align 4
     %3 = alloca i32, align 4
     store i32 10, i32* %1, align 4
     store i32 20, i32* %2, align 4
     %4 = load i32, i32* %1, align 4
     %5 = load i32, i32* %2, align 4
     %6 = add nsw i32 %4, %5
     store i32 %6, i32* %3, align 4
     %7 = load i32, i32* %3, align 4
     ret i32 %7
   }
   
   attributes #0 = { noinline nounwind "correctly-rounded-divide-sqrt-fp-math"="false" "disable-tail-calls"="false" "frame-pointer"="all" "less-precise-fpmad"="false" "min-legal-vector-width"="0" "no-infs-fp-math"="false" "no-jump-tables"="false" "no-nans-fp-math"="false" "no-signed-zeros-fp-math"="false" "no-trapping-math"="false" "stack-protector-buffer-size"="8" "target-features"="+a,+c,+m,+relax,-save-restore" "unsafe-fp-math"="false" "use-soft-float"="false" }
   
   !llvm.module.flags = !{!0, !1}
   !llvm.ident = !{!2}
   
   !0 = !{i32 1, !"wchar_size", i32 4}
   !1 = !{i32 1, !"target-abi", !"lp64"}
   !2 = !{!"clang version 11.0.0 (https://github.com/llvm/llvm-project.git 05c0d3491822b3a74f49be2fe8c8273e436ab7ec)"}
   ```

4. opt -mem2reg test.bc -o test01.bc

   opt（LLVM optimizer)

   ```
   opt [options] [filename]
   ```
   | 參數          | 功能                                                         |
   | ------------- | ------------------------------------------------------------ |
   | -mem2reg      | This file promotes memory references to be register references. It promotes alloca instructions which only have loads and stores as uses. An `alloca` is transformed by using dominator frontiers to place phi nodes, then traversing the function in depth-first order to rewrite loads and stores as appropriate. This is just the standard SSA construction algorithm to construct “pruned” SSA form. |
   | -o <filename> | 關閉最佳化                                                   |
   
   結果產生test01.ll
   
   ```assembly
   ; ModuleID = 'test01.bc'
   source_filename = "test.c"
   target datalayout = "e-m:e-p:64:64-i64:64-i128:128-n64-S128"
   target triple = "riscv64-unknown-unknown-elf"
   
   ; Function Attrs: noinline nounwind
   define dso_local signext i32 @foo() #0 {
     %1 = add nsw i32 10, 20
     ret i32 %1
   }
   
   attributes #0 = { noinline nounwind "correctly-rounded-divide-sqrt-fp-math"="false" "disable-tail-calls"="false" "frame-pointer"="all" "less-precise-fpmad"="false" "min-legal-vector-width"="0" "no-infs-fp-math"="false" "no-jump-tables"="false" "no-nans-fp-math"="false" "no-signed-zeros-fp-math"="false" "no-trapping-math"="false" "stack-protector-buffer-size"="8" "target-features"="+a,+c,+m,+relax,-save-restore" "unsafe-fp-math"="false" "use-soft-float"="false" }
   
   !llvm.module.flags = !{!0, !1}
   !llvm.ident = !{!2}
   
   !0 = !{i32 1, !"wchar_size", i32 4}
   !1 = !{i32 1, !"target-abi", !"lp64"}
   !2 = !{!"clang version 11.0.0 (https://github.com/llvm/llvm-project.git 05c0d3491822b3a74f49be2fe8c8273e436ab7ec)"}
   ```
   
   

5. 組語解讀

   這組語不知道為什麼翻的有點智障，可能是沒有最佳化的關係？

   1. 出現的數字跟記憶體有關的話單位都是byte

   2. sp和s0跟stack息息相關，呼叫函數sp會往下挖，而s0會在sp原本的位置，在存變數的過程會大概像這樣

      存sw a0, -20(s0) ~ sw a0, -24(s0) ~ sw a0, -28(s0)，就存的地方越來越下面

   3. double~8bytes ; integer~4bytes

   下面是test.c的講解

   ```assembly
   	.text
   	.file	"test.c"
   	.globl	foo                     # -- Begin function foo
   	.p2align	1
   	.type	foo,@function
   foo:                                    # @foo
   .Lfoo$local:
   # %bb.0:
   	addi	sp, sp, -32 # sp(stack pointer）
   	sd	ra, 24(sp) # ra(return address)，將ra存到『sp指向的記憶體位置+24byte～sp指向的記憶體位置+32byte』
   	sd	s0, 16(sp) # s0( Saved register or Frame pointer)，將ra存到『sp指向的記憶體位置+16byte～sp指向的記憶體位置+24byte』
   	addi	s0, sp, 32 # sp+32存到s0。變回原本呼叫函數前sp的位址 -> 開始往下存變數到stack～sw xxx, 更大的負數(s0)
   	addi	a0, zero, 10 # a0當作a變數，初始化為10
   	sw	a0, -20(s0) # 將a0存到『s0 - 20 bit 位址』
   	addi	a0, zero, 20 # 又將a0當作b變數，初始化為20
   	sw	a0, -24(s0) # 將a0存到『s0 - 24 byte』
   	lw	a0, -20(s0) # a0存10
   	lw	a1, -24(s0) # a1存20
   	add	a0, a0, a1 # a0 = 10 + 20
   	sw	a0, -28(s0) # 存a0(值30)到『s0-28』
   	lw	a0, -28(s0) # 再load 30回去(智障?)a0
   	ld	s0, 16(sp) # 回覆原本的值s0的值
   	ld	ra, 24(sp) # 回覆原本的ra(return address)
   	addi	sp, sp, 32 # 把指向堆疊尾巴的sp加回去原本的位址
   	ret # 回傳
   .Lfunc_end0:
   	.size	foo, .Lfunc_end0-foo
                                           # -- End function
   	.ident	"clang version 11.0.0 (https://github.com/llvm/llvm-project.git 05c0d3491822b3a74f49be2fe8c8273e436ab7ec)"
   	.section	".note.GNU-stack","",@progbits
   	.addrsig
   ```

   LLVM IR的一些基本指令和介紹

   參考網址：https://releases.llvm.org/2.6/docs/LangRef.html#i_alloca

   %number -> 可能是值會記憶體位址（有人說是指標，有人說是local variable）

   ```assembly
    %1 = alloca i32, align 4 
   ```

   %1 -> 指標，指向配置的記憶體

   alloca -> 分配stack記憶體的指令，當函數回傳會自動釋放記憶體

   i32->整數型態？~32bit

   align 4 -> 對齊4bytes（記憶體必定佔用4bytes，這樣存取會比較整齊，記憶體對齊）

   ```assembly
     store i32 10, i32* %1, align 4 
   ```

   | store | i32                            | 10       | i32*%1,                     | align 4    |
   | ----- | ------------------------------ | -------- | --------------------------- | ---------- |
   |       | 要存的值的型態(integer-32bit?) | 要存的值 | 指標指向的記憶體(大小32bit) | 對齊4bytes |

   就是將10存到%1指向的記憶體位址

   ```assembly
   %4 = load i32, i32* %1, align 4
   ```

   將%1指向的記憶體中的值取出來給%4（%4代表的是值?，%number指的可能是記憶體位址或值?)

   ```assembly
   %6 = add nsw i32 %4, %5 
   ```

    <result> = add nsw <ty> <op1>, <op2> 

   就是做加法 ~ %6 = %4 + %5（簡單看就是這樣)

   nsw代表No Signed Wrap，就是當無號加法overflow時<result>的值為未定義，另外還有nuw代表No Unsigned Wrap，就是當有號加法overflow時<result>的值為未定義

   下面是沒有最佳化的LLVM IR(bitcode反組譯得來的）的簡單註解

   ```assembly
   ; ModuleID = 'test.bc'
   source_filename = "test.c"
   target datalayout = "e-m:e-p:64:64-i64:64-i128:128-n64-S128"
   target triple = "riscv64-unknown-unknown-elf"
   
   ; Function Attrs: noinline nounwind
   define dso_local signext i32 @foo() #0 {
     %1 = alloca i32, align 4  # 宣告4bytes記憶體，讓%1指標指向它
     %2 = alloca i32, align 4  # 宣告4bytes記憶體，讓%2指標指向它
     %3 = alloca i32, align 4  # 宣告4bytes記憶體，讓%3指標指向它
     store i32 10, i32* %1, align 4  # %1指向的記憶體位址存10
     store i32 20, i32* %2, align 4 # %2指向的記憶體位址存20
     %4 = load i32, i32* %1, align 4 # %4=10
     %5 = load i32, i32* %2, align 4 # %5 =20
     %6 = add nsw i32 %4, %5 # %6 = %4 + %5 -> %6 = 10 + 20 = 30
     store i32 %6, i32* %3, align 4 # %6是值存到%3記憶體位址
     %7 = load i32, i32* %3, align 4 # %7是%3記憶體位址的值（智障?)
     ret i32 %7 # 回傳%7（值～30)
   }
   
   attributes #0 = { noinline nounwind "correctly-rounded-divide-sqrt-fp-math"="false" "disable-tail-calls"="false" "frame-pointer"="all" "less-precise-fpmad"="false" "min-legal-vector-width"="0" "no-infs-fp-math"="false" "no-jump-tables"="false" "no-nans-fp-math"="false" "no-signed-zeros-fp-math"="false" "no-trapping-math"="false" "stack-protector-buffer-size"="8" "target-features"="+a,+c,+m,+relax,-save-restore" "unsafe-fp-math"="false" "use-soft-float"="false" }
   
   !llvm.module.flags = !{!0, !1}
   !llvm.ident = !{!2}
   
   !0 = !{i32 1, !"wchar_size", i32 4}
   !1 = !{i32 1, !"target-abi", !"lp64"}
   !2 = !{!"clang version 11.0.0 (https://github.com/llvm/llvm-project.git 05c0d3491822b3a74f49be2fe8c8273e436ab7ec)"}
   ```

   下面是最佳化的LLVM IR的簡單註解，就直接做加法，不像個智障load, store反覆

   ```assembly
   ; ModuleID = 'test01.bc'
   source_filename = "test.c"
   target datalayout = "e-m:e-p:64:64-i64:64-i128:128-n64-S128"
   target triple = "riscv64-unknown-unknown-elf"
   
   ; Function Attrs: noinline nounwind
   define dso_local signext i32 @foo() #0 {
     %1 = add nsw i32 10, 20
     ret i32 %1
   }
   
   attributes #0 = { noinline nounwind "correctly-rounded-divide-sqrt-fp-math"="false" "disable-tail-calls"="false" "frame-pointer"="all" "less-precise-fpmad"="false" "min-legal-vector-width"="0" "no-infs-fp-math"="false" "no-jump-tables"="false" "no-nans-fp-math"="false" "no-signed-zeros-fp-math"="false" "no-trapping-math"="false" "stack-protector-buffer-size"="8" "target-features"="+a,+c,+m,+relax,-save-restore" "unsafe-fp-math"="false" "use-soft-float"="false" }
   
   !llvm.module.flags = !{!0, !1}
   !llvm.ident = !{!2}
   
   !0 = !{i32 1, !"wchar_size", i32 4}
   !1 = !{i32 1, !"target-abi", !"lp64"}
   !2 = !{!"clang version 11.0.0 (https://github.com/llvm/llvm-project.git 05c0d3491822b3a74f49be2fe8c8273e436ab7ec)"}
   ```

   