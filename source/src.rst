src
###############################################################################

ソースコードの構成

vm/compiler ::

  Compiler.cpp           JITコンパイラの主制御 別threadがTaskをQueue管理
  Compiler.h             VMへ公開するシンボルを宣言
  CompilerIR.h           中間表現の定義(CompileUnit, MIR, LIR)
  CompilerInternals.h    includeの制御
  CompilerUtility.h      DumpやGlowable系の定義
  Dataflow.cpp           DisAsm, SSA, Liveness, IndVar
  Dataflow.h             Dataflow解析結果を格納するIRの定義
  Frontend.cpp           コンパイル処理の主制御(TracingJIT)
                         dvmCompileTrace/dvmCompileLoop/dvmCompileMethod
  InlineTransformation.cpp inline展開の制御 PredicatedInlineによるPolyも対象
  IntermediateRep.cpp    IR用のUtility
  Loop.cpp               Loop用の解析処理と最適化
  Loop.h                 Loop用の構造体定義
  Ralloc.cpp             SSAベースの単純な型推論とLocalRegAlloc 1pass
  SSATransformation.cpp  SSAのUtility Dom DomFの解析 SSAへの変換 Loop検知
  Utility.cpp            compilerのメモリ管理とDebug用処理
  template               各種アセンブラのテンプレート
  codegen                archごとのLIRおよびコード生成
      CodegenFactory.cpp Load/StoreのFactoryの実装
      CompilerCodegen.h  Codegenの各種メソッドの宣言のみ
      Optimizer.h        LIRのOptimizerの宣言のみ
      Ralloc.h           RegAllocのメソッドの宣言のみ
      RallocUtil.cpp     RegAllocのUtility RegisterInfoとRegLocationの操作
      arm
      mips
      x86

vm/compiler/codegen/arm ::

  ArchFactory.cpp        TargetLIRのサポート
  ArchUtility.cpp        dumpやdecode用処理
  ArmLIR.h               ARM用のLIRの定義
  ArmRallocUtil.cpp      RallocUtilへのラッパー
  Assemble.cpp           簡易Assemble, LIR2Asm, TC2IC, 難しいです。。
  CalloutHelper.h        DalvikへのCallout定義
  Codegen.h              Codegenの関数宣言
  CodegenCommon.cpp      共通処理定義。newLIRs with 0-4
  CodegenDriver.cpp      Codegenの主制御。4kえ。 dvmCompilerDoWork()
                         genXXとhandleFmtXXが多数定義
  FP
  GlobalOptimizations.cpp applyRedundantBranchElimination()
  LocalOptimizations.cpp applyLoadStoreElimination() applyLoadHosting()
  README.txt
  Thumb
  Thumb2
  armv5te
  armv5te-vfp
  armv7-a
  armv7-a-neon
    MethodCodegenDriver.cpp methodBlockCodeGenを定義


