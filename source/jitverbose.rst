jitverbose
###############################################################################

dalvik vmの起動時に、-Xjitverboseオプションを付加すると、

いろいろとログが参照出来ます。

エミュレータを起動したのち、adb logcatで参照可能

オプションの指定の仕方がわからなかったので、

まずはオプションを強制trueにしてrebuild

dexに関しては下記が詳しい

http://source.android.com/tech/dalvik/dalvik-bytecode.html

サンプル出力
*******************************************************************************

JBで最初にJITコンパイルされるログの解説 ::

  D/dalvikvm(  149): JIT started for system_server

  /// dvmCompileTrace()
  D/dalvikvm(  149): --------
  D/dalvikvm(  149): Compiler: Building trace for fixSlashes, offset 0xb
                                                  ^ method->name
  
  /// parseInsn() dump dex
  D/dalvikvm(  149): 0x45c968ae: 0x0049 aget-char v0, v6, v1
  D/dalvikvm(  149): 0x45c968b2: 0x0013 const/16 v7, (#47), (#0)
  D/dalvikvm(  149): 0x45c968b6: 0x0033 if-ne v0, v7, (+f)
                     ^ codePtr   ^opcode ^decodedString
  
  /// dvmCompileTrace()
  //  traceSize of dexCode->insnsSize, numBlocks
  D/dalvikvm(  149): TRACEINFO (1): 0x45c96898 Ljava/io/File;fixSlashes.(Ljava/lang/String;)Ljava/lang/String; 0xb 6 of 56, 6 blocks
  
  /// dvmCompilerDumpCompilationUnit()
  D/dalvikvm(  149): Compiling Ljava/io/File; fixSlashes
  D/dalvikvm(  149): 56 insns
  D/dalvikvm(  149): 6 blocks in total
  D/dalvikvm(  149): Block 0 (Entry Block) (insn 000b - 000b empty)
  D/dalvikvm(  149):   Fallthrough : block 1 (000b)
  D/dalvikvm(  149): Block 1 (Code Block) (insn 000b - 000f)
  D/dalvikvm(  149):   Taken branch: block 3 (001e)
  D/dalvikvm(  149):   Fallthrough : block 2 (0011)
  D/dalvikvm(  149): Block 2 (Normal Chaining Cell) (insn 0011 - 0011 empty)
  D/dalvikvm(  149): Block 3 (Normal Chaining Cell) (insn 001e - 001e empty)
  D/dalvikvm(  149): Block 4 (PC Reconstruction) (insn 0000 - 0000 empty)
  D/dalvikvm(  149): Block 5 (Exception Handling) (insn 0000 - 0000 empty)
  
  /// dvmCompilerCodegenDump()
  D/dalvikvm(  149): Trace Dalvik PC: 0x45c968ae
  D/dalvikvm(  149): Dumping LIR insns
  D/dalvikvm(  149): installed code is at 0x4d487000
  D/dalvikvm(  149): total size is 116 bytes
  
  
  /// dvmDumpLIRInsn()
  D/dalvikvm(  149): 0x4d487000 (0000): data    0xc028(49192)
  D/dalvikvm(  149): 0x4d487002 (0002): data    0x4ac2(19138)
  D/dalvikvm(  149): 0x4d487004 (0004): data    0x0054(84)
  D/dalvikvm(  149): 0x4d487006 (0006): ldr     r0, [r15pc, -#8]
  D/dalvikvm(  149): 0x4d48700a (000a): ldr     r1, [r0, #0]
  D/dalvikvm(  149): 0x4d48700c (000c): adds    r1, r1, #1
  D/dalvikvm(  149): 0x4d48700e (000e): str     r1, [r0, #0]
  D/dalvikvm(  149): -------- entry offset: 0x000b
  D/dalvikvm(  149): L0x4ac2a110:
  D/dalvikvm(  149): -------- dalvik offset: 0x000b @ aget-char v0, v6, v1
  D/dalvikvm(  149): 0x4d487010 (0010): ldr     r0, [r5, #24]
  D/dalvikvm(  149): 0x4d487012 (0012): ldr     r1, [r5, #4]
  D/dalvikvm(  149): 0x4d487014 (0014): cbz     r0,0x4d487036 (L0x4ac2a5b0)
  D/dalvikvm(  149): 0x4d487016 (0016): ldr     r3, [r0, #8]
  D/dalvikvm(  149): 0x4d487018 (0018): add     r2,r0,#16
  D/dalvikvm(  149): 0x4d48701c (001c): cmp     r1, r3
  D/dalvikvm(  149): 0x4d48701e (001e): bcs     0x4d487036 (L0x4ac2a5b0)
  D/dalvikvm(  149): 0x4d487022 (0022): ldrh    r4, [r2, r1, LSL #1]
  D/dalvikvm(  149): -------- dalvik offset: 0x000d @ const/16 v7, (#47), (#0)
  D/dalvikvm(  149): 0x4d487026 (0026): movs    r7, #47
  D/dalvikvm(  149): -------- dalvik offset: 0x000f @ if-ne v0, v7, (+f)
  D/dalvikvm(  149): 0x4d487028 (0028): cmp     r4, r7
  D/dalvikvm(  149): 0x4d48702a (002a): str     r7, [r5, #28]
  D/dalvikvm(  149): 0x4d48702c (002c): str     r4, [r5, #0]
  D/dalvikvm(  149): 0x4d48702e (002e): bne     0x4d48704c (L0x4ac2a190)
  D/dalvikvm(  149): 0x4d487032 (0032): b       0x4d487040 (L0x4ac2a150)
  D/dalvikvm(  149): 0x4d487034 (0034): undefined
  D/dalvikvm(  149): L0x4ac2a5b0:
  D/dalvikvm(  149): -------- reconstruct dalvik PC : 0x45c968ae @ +0x000b
  D/dalvikvm(  149): 0x4d487036 (0036): ldr     r0, [r15pc, #56]
  D/dalvikvm(  149): Exception_Handling:
  D/dalvikvm(  149): 0x4d48703a (003a): ldr     r1, [r6, #108]
  D/dalvikvm(  149): 0x4d48703c (003c): blx     r1
  D/dalvikvm(  149): 0x4d48703e (003e): .align4
  D/dalvikvm(  149): L0x4ac2a150:
  D/dalvikvm(  149): -------- chaining cell (normal): 0x0011
  D/dalvikvm(  149): 0x4d487040 (0040): b       0x4d487044 (L0x4ac2ad44)
  D/dalvikvm(  149): 0x4d487042 (0042): orrs    r0, r0
  D/dalvikvm(  149): L0x4ac2ad44:
  D/dalvikvm(  149): 0x4d487044 (0044): ldr     r0, [r6, #100]
  D/dalvikvm(  149): 0x4d487046 (0046): blx     r0
  D/dalvikvm(  149): 0x4d487048 (0048): data    0x68ba(26810)
  D/dalvikvm(  149): 0x4d48704a (004a): data    0x45c9(17865)
  D/dalvikvm(  149): 0x4d48704c (004c): .align4
  D/dalvikvm(  149): L0x4ac2a190:
  D/dalvikvm(  149): -------- chaining cell (normal): 0x001e
  D/dalvikvm(  149): 0x4d48704c (004c): b       0x4d487050 (L0x4ac2af44)
  D/dalvikvm(  149): 0x4d48704e (004e): orrs    r0, r0
  D/dalvikvm(  149): L0x4ac2af44:
  D/dalvikvm(  149): 0x4d487050 (0050): ldr     r0, [r6, #100]
  D/dalvikvm(  149): 0x4d487052 (0052): blx     r0
  D/dalvikvm(  149): 0x4d487054 (0054): data    0x68d4(26836)
  D/dalvikvm(  149): 0x4d487056 (0056): data    0x45c9(17865)
  D/dalvikvm(  149): -------- end of chaining cells (0x0058)
  D/dalvikvm(  149): 0x4d487070 (0070): .word (0x45c968ae)
  D/dalvikvm(  149): End Ljava/io/File;fixSlashes, 3 Dalvik instructions





