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



### Bonus part

1. 基本配置 只要有基本的 x86 gcc compiler 就可以用了

2. 操作步驟很簡單
   依照順序執行下面的指令 [假定你的程式碼名稱為test.c]

   ```makefile
   gcc test.c -o test
   ```

   指定可執行檔的名稱為test 然後就會看到test檔

   執行看看

   ``` makefile
   ./test
   ```

   就如一般的可執行檔執行

3. disassemble the executable file
   接著要把可執行檔反組譯成 assembly code
   操作下面指令

   ``` makefile
   objdump test --disassemble > test.txt
   ```

   objdump test --disassemble這行本身是在將 可執行檔test反組譯回 assembly code

   如果直接執行

   ``` makefile
   objdump test --disassemble
   ```

   反組譯後的 assembly code就會跑到視窗上

   \> test.txt 則是 匯出到test.txt檔

   打開test.txt 就會看到以下的code

   ``` assembly
   
   test:     file format elf64-x86-64
   
   
   Disassembly of section .init:
   
   0000000000001000 <_init>:
       1000:	48 83 ec 08          	sub    $0x8,%rsp
       1004:	48 8b 05 dd 2f 00 00 	mov    0x2fdd(%rip),%rax        # 3fe8 <__gmon_start__>
       
   (....以下省略)
   ```

4. 順便附上老師給的x86指令集
   https://www.intel.com/content/dam/www/public/us/en/documents/manuals/64-ia-32-architectures-software-developer-instruction-set-reference-manual-325383.pdf 

   

5. **Reference** [網站都給你了 自己點進去看看]
   https://c9x.me/x86/html/file_module_x86_id_269.html
   https://c9x.me/x86/html/file_module_x86_id_176.html
   https://c9x.me/x86/html/file_module_x86_id_308.html
   https://hackmd.io/@dange/rk9xmgHKX?type=view

6. Cheat sheet for x86-64 [不確定規格是不是對的]
   https://cs.brown.edu/courses/cs033/docs/guides/x64_cheatsheet.pdf