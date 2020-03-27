# 操作步驟

1. build the executable program (image) of a sample program

   使用的是如果是risc-v(處理器)的llvm，如下網址

   https://github.com/sifive/riscv-llvm?fbclid=IwAR3svsOfOmX8noSfA0p3TVc3Kt4zKeU_PHFFSKuv84ZiiWa_Ubla5R5XhUw

   就看自己的cpu是x32還是x64選擇指令的options

   32 bit就下

   ```makefile
   clang -O -c test.c --target=riscv32
   riscv64-unknown-elf-gcc test.o -o test -march=rv32imac -mabi=ilp32
   qemu-riscv32 test
   ```

   64 bit就下

   ```makefile
   clang -O -c test.c
   riscv64-unknown-elf-gcc test.o -o test -march=rv64imac -mabi=lp64
   qemu-riscv64 test
   ```

2. disassemble the whole executable program back to assembly (using objdump)

   使用的是如果是risc-v(處理器)的llvm，可能不能用系統預載的objdump，因為處理器x86，objdump只能反組譯x86的executable program。因此就用當初直接載好的risc-v版的llvm的反組譯程式，也就是riscv64-unknown-elf-objdump

   假如要反組譯test程式，就執行下面的指令

   ```makefile
   riscv64-unknown-elf-objdump --disassemble test
   ```
   
   大概會列印出下面的東西
   
   ```assembly
   test      檔案格式 elf64-littleriscv
   
   
   .text 區段的反組譯：
   
   00000000000100e8 <register_fini>:
      100e8:	00000793          	li	a5,0
      100ec:	c789                	beqz	a5,100f6 <register_fini+0xe>
      100ee:	6541                	lui	a0,0x10
      100f0:	38050513          	addi	a0,a0,896 # 10380 <__libc_fini_array>
      100f4:	a4c1                	j	103b4 <atexit>
      100f6:	8082                	ret
   (後略...)
   ```



