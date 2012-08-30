jitcompiler
###############################################################################

JITコンパイルの処理順
===============================================================================

Workerの制御::

  dvmCompilerWorkEnqueue()

  dvmLockMutex(&gDvmJit.compilerLock)
  CompilerWorkOrder() * newOrder <-- workDequeue()で取得
  dvmUnlockMutex(&gDvmJit.compilerLock)

Compilerの制御 ::

  compilerThreadStart()

  work = workDequeue()
  dvmCompilerDoWork(&work) <--  機種依存の中に定義あり
    dvmCompileTrace()
  dvmLockMutex()
    dvmJitSetCodeAddr(work ...)
  dvmUnlockMutex()

  dvmCompileTrace
    こちらがメインのJIT

  dvmCompileMethod
    MethodCompileが許可されている場合に限り、
    invokeVirtualのinline展開の際に限り、こちらが呼ばれる。

Frontend::dvmCompileTrace
===============================================================================

処理の流れ::

  dvmCompilerAnalyzeMethodBody()
  for () {
    dvmCompilerNewBB()
  }
  for (bb:bblist) {
    findBlockBoundary()
    if (isLoop) compileLoop()
  }
  dvmInitializeSSAConversion()
  dvmCompilerNonLoopAnalysis()
  dvmCompilerMIR2LIR()
  dvmCompilerAssembleLIR()
  dvmJitInstallClassObjectPointers()

Frontend::compileLoop
===============================================================================

Frontend::compileTrace
===============================================================================

Frontend::compileMethod
===============================================================================

codegen
===============================================================================

LocalOptimizer ::

  LoadStoreElimination
  LoadHoisting

GlobalOptimizer ::

  arm
    RedundantBranchElimination

  mips
    RedundantBranchElimination
    CopyPropagation
    MergeMovs
    introduceBranchDelaySlot


LoadStoreElimination ::

  arm/mips

LoadHoisting ::

  arm/mips

TrackLiveTemps ::

  arm/arm neon/mips

SuppressLoads ::

  arm/mips

MethodInlining ::

  all

MethodJit ::

  arm/mips

  ../InlineTransformation


dvmCompilerInlineMIR
===============================================================================

OP_INVOKE_SUPER

  DIRECT

  STATIC

  SUPER_QUICK

    RANGE

isRange = true

OP_INVOKE_VIRTUAL

  QUICK

OP_INVOKE_INTERFACE

  RANGE




tryInlineSingletonCallsite
===============================================================================

tryInlineVirtualCallsite
===============================================================================

isNativeMethod

inlineEmptyVirtualCallee

inlineGetter

inlineSetter


METHOD_IS_LEAF ???

inlineGetter inlineSetterする際に、

Virtualな場合、

isPredicatedフラグがtrue

invokePolyGetterInlining

sigletonは、false


inlinePredicateっていうopcode

TAG MIR_INLINED_PRED

armとmipsにターゲットがある

CompilerCodegen
===============================================================================

dvmCompilerMIR2LIR()
  for (bb:bblist) {


MIRのデザイン
===============================================================================

CompilationUnit ::

  non


JitEntry ::

  non


BasicBlock ::

  id
  Method*
  BBType
  MIR*first
  MIR*last

  BasicBlockDataFlow
  BitVector pred, dom, idom domfr

  FallThroght


MIR ::

  compilerIR.hに定義あり
  struct MIR*prev, next
  struct SSARepresentation * ssarep
  ??? DecordedInstrunction dalvikInsn

  ssarepのみの片方向参照か

SSARepresentation ::

  non

LIR ::

  LIR* prev, next, target
  int offset


CallSiteinfo ::

  codegenの下、機種依存ごとに定義



INVOKE_INTERFACE_RANGE

dvmCompilerMIR2LIR
===============================================================================

処理概要::

  for (bb:bblist) {
    for (mir:mirlist) {
    }
  }
  dvmCompilerApplyLocalOptimizations()
    applyLoadStoreElimination()
    applyLoadHosting()
  dvmCompilerApplyGlobalOptimizations()
    applyRedundantBranchElimination()


Debug方法
==============================================================================

cUnit.printMe

dvmCompilerCodegenDump

// Debugging only

dvmDumpCFG(&cUnit, "/sdcard/cfg/");

修正箇所 ::

  } else if (strncmp(argv[i], "-Xjitverbose", 12) == 0) {
  gDvmJit.printMe = true;
  } else if (strncmp(argv[i], "-Xjitprofile", 12) == 0) {
  gDvmJit.profileMode = kTraceProfilingContinuous;
  } else if (strncmp(argv[i], "-Xjitdisableopt", 15) == 0) {



Debug/Dump系のメソッド一覧
===============================================================================
/// Frontend

/// dot graphを生成する。

dvmDumpCFG()

dvmCompilerMIR2LIR

cUnit.printMe をフラグ立てる

dumpRegPool

dvmCompilerCodegenDump

dumpDependentInsnPair

上記を有効にしてデバッグも出力すれば最適化の適応具合も見れそう

