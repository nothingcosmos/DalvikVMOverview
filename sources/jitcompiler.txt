jitcompiler
###############################################################################

DalvikVMは、最初dexをインタプリタ実行するのですが、その後JITコンパイルします。

その際、dexはmethod単位ではなく、よく実行されるdexをひとまとまりにして、Tracing JITします。

DalvikVMは、JITCompilerを別スレッドで起動しっぱなしにしておき、

JITCompilerに、よく実行されるdex群を入力として、JITコンパイルを依頼します。

依頼する際のインターフェースは、JITCompilerのもつqueueに突っ込んでいくだけ。

JITコンパイルの処理順
===============================================================================

JITCompilerは、queueに入っているworkを、１つずつコンパイルしていきます。

queueから取り出し->コンパイル->install->取り出し->コンパイル->install ...

Workerの制御::

  dvmCompilerWorkEnqueue()

  dvmLockMutex(&gDvmJit.compilerLock)
  CompilerWorkOrder() * newOrder <-- workDequeue()で取得
  dvmUnlockMutex(&gDvmJit.compilerLock)

Compilerの制御 ::

  compilerThreadStart()

  work = workDequeue()
  dvmCompilerDoWork(&work) <-- 機種依存の中に定義あり
    dvmCompileTrace()      <-- インタプリタ実行と並行してコンパイル
  dvmLockMutex()           <-- installのときだけ、Mutex
    dvmJitSetCodeAddr(work ...)
  dvmUnlockMutex()


メインのJITCompile処理 dvmCompileTrace

実装としてdvmCompileMethodも存在するが、初期状態では使用されない。

MethodCompileが許可されている場合に限り、

invokeVirtualのpredicated inlining、こちらが呼ばれる。

Frontend::dvmCompileTrace
===============================================================================

特徴として、MIRレベルでは解析および変換フラグのみ付加していく。

MIRの動的な変換は行わず、LIRへの変換、およびLIR上でフラグを参照して最適化するはず

ログを参照すると、dexは3～10くらいの単位でコンパイルが依頼されてくるため、

あまり最適化の余地はない。

自前で作成したループや関数呼び出しを食べされれば、詳細が見えてくるかもしれない。。

処理の流れ::

  dvmCompilerAnalyzeMethodBody()
  for () {
    dvmCompilerNewBB()        <-- dexをたどってBasicBlockを生成して歩く
  }
  for (bb:bblist) {
    findBlockBoundary()
    if (isLoop) compileLoop() <-- ループの検知
  }
  dvmInitializeSSAConversion()   <-- SSA形式のMIRを生成する
  dvmCompilerNonLoopAnalysis()
  dvmCompilerMIR2LIR()           <-- 機種依存のLIRへ変換 および最適化
  dvmCompilerAssembleLIR()       <-- アセンブル
  dvmJitInstallClassObjectPointers()

以降は手抜きです。。
===============================================================================

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

