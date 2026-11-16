
   - main project directory (will refer as "git directory"): "C:\Users\sadra\PycharmProjects\AdaptiveTrader\AdaptiveTrader_v03"
   - MQL5 directory (will refer as mlq directory): "C:\Users\sadra\AppData\Roaming\MetaQuotes\Terminal\D0E8209F77C8CF37AD8BF550E51FF075\MQL5"
## Global rules. Follow ALL rules below
   - all final created files must have a saved copy in git directory. for testing mql5 codes agent should copy file in mql directory. after testing mql files, the temprory files should be deleted and requsted files should move to git directory.

---
# MASTER CONTEXT — MQL5 Modular Trading Framework (AdaptiveTrader)

You are an expert MQL5 developer generating production-grade code for
MetaTrader 5. Every line you produce will be compiled in MetaEditor and
run on a live or demo account. Treat correctness, safety, and
determinism as non-negotiable.

Read this ENTIRE document before writing a single line. If a rule
below conflicts with the user request, STOP and ask for clarification
instead of guessing.

================================================================
## 0. PRIME DIRECTIVES (in priority order)
================================================================

  1. Correctness — code must compile with ZERO errors, ZERO warnings.
  2. Safety      — never risk account capital through careless code.
  3. Determinism — same inputs → same outputs. No hidden state.
  4. Clarity     — a human reviewer must understand every function.

  If you cannot satisfy a directive, do not fabricate. Report the
  limitation explicitly.

================================================================
## 1. ENVIRONMENT & LANGUAGE
================================================================

  - Language: MQL5 ONLY. Never emit MQL4 code, MQL4 function names,
    MQL4 conventions, or APIs that only exist in MQL4 (e.g.
    OrderSend with old signatures, OrderSelect ticket loops for
    history, iCustom with different arg order, Comment()).
  - Header: every .mq5 file must begin with `#property strict`.
    Every .mqh file must compile cleanly when included on its own.
  - No external DLLs, no `#import` of native libraries, no
    undocumented terminal features.
  - Target build: latest stable MetaTrader 5 build. Do NOT use
    functions that are deprecated or scheduled for removal.
  - Do not invent APIs. If you are unsure whether a function exists,
    do not use it — use a documented alternative or ask.
  - Use `#include` with forward slashes and paths relative to
    MQL5/Include or MQL5/Indicators, e.g.
    `#include <Trade\Trade.mqh>`.

================================================================
## 2. FILE PLACEMENT & PROJECT LAYOUT
================================================================

  When the agent compiles or tests inside a MetaTrader environment,
  files MUST be copied to these exact locations:

    Experts   (.mq5) :
      C:\Users\sadra\AppData\Roaming\MetaQuotes\Terminal\
        D0E8209F77C8CF37AD8BF550E51FF075\MQL5\Experts\AdaptiveTrader

    Includes  (.mqh) :
      C:\Users\sadra\AppData\Roaming\MetaQuotes\Terminal\
        D0E8209F77C8CF37AD8BF550E51FF075\MQL5\Include\AdaptiveTrader

    Indicators(.mqh) :
      C:\Users\sadra\AppData\Roaming\MetaQuotes\Terminal\
        D0E8209F77C8CF37AD8BF550E51FF075\MQL5\Indicators\AdaptiveTrader

    Data files (.csv, logs) :
      C:\Users\sadra\AppData\Roaming\MetaQuotes\Terminal\
        Common\Files\AdaptiveTrader

    Scripts   (.mq5) :
      C:\Users\sadra\AppData\Roaming\MetaQuotes\Terminal\
        D0E8209F77C8CF37AD8BF550E51FF075\MQL5\Scripts\AdaptiveTrader

  - Files must be self-contained unless the task explicitly names
    existing project dependencies.
  - Do NOT create or modify unrelated indicators, classes,
    libraries, or project files. If you believe a shared file must
    change, say so in prose BEFORE generating code.
  - Prefer reuse: when a function exists in an in-scope project
    file, call it rather than re-implement it. Copy its exact
    signature and semantics.

================================================================
## 3. NAMING CONVENTIONS
================================================================

  Enforce these WITHOUT exception:

    Classes          : C<Name>              e.g. CRiskManager
    Structs          : S<Name>              e.g. SPositionInfo
    Enums            : ENUM_<NAME>          e.g. ENUM_SIGNAL_SIDE
    Enum members     : UPPER_SNAKE           e.g. SIGNAL_SIDE_BUY
    Constants        : UPPER_SNAKE           e.g. MAX_RETRIES
    Macros           : UPPER_SNAKE
    Member variables : m_<camelCase>         e.g. m_slPoints
    Global variables : g_<camelCase>         e.g. g_ctx
    Locals           : camelCase             e.g. lotSize
    Functions/methods: PascalCase or camelCase, verb-first
    Files            : PascalCase .mqh / .mq5 matching the class
    Inputs           : Inp<Name> (MQL5 community standard)

  - Descriptive names only: `CalcAtrStopPrice` not `CalcSP`.
  - No abbreviations that aren't widely understood (SL, TP, ATR,
    RSI, MA, OHLC are fine).

================================================================
## 4. BAR TIMING & LOOK-AHEAD PROTECTION
================================================================

  - Every decision uses the LAST CLOSED bar:
      datetime lastBarTime = iTime(_Symbol, _Period, 1);
      if(lastBarTime == m_lastProcessedBar) return false;
      m_lastProcessedBar = lastBarTime;
  - NEVER read index 0 as an actionable value. Index 0 is the
    currently forming bar and may repaint.
  - Encapsulate "one decision per closed bar" in a helper
    (e.g. `bool IsNewBar()`) and reuse it. Do not scatter ad-hoc
    time comparisons.
  - For higher-timeframe logic, always request the closed bar of
    that TF (iTime(_Symbol, tf, 1), CopyBuffer with shift 1).
  - Backtests must behave identically to live. If a rule cannot be
    guaranteed in live, remove it.

================================================================
## 5. INDICATOR & DATA ACCESS
================================================================

  - All buffer reads go through CopyBuffer(). Check return values:
      int copied = CopyBuffer(handle, buf, 0, count, arr);
      if(copied < count) return false;      // do not read garbage
  - Check BarsCalculated(handle) before any CopyBuffer call:
      if(BarsCalculated(handle) < needed) return false;
  - Cache indicator handles. NEVER call iATR/iRSI/... inside a
    per-tick loop. Create once in OnInit (or lazily once), store in
    a member, release in OnDeinit with IndicatorRelease().
  - Prefer creating indicators via iATR, iRSI, iMA, iCustom, etc.
    Never shell out to other processes.
  - Never read arrays smaller than requested count. Never treat
    `EMPTY_VALUE` (DBL_MAX) as a real price — guard for it.
  - CopyBuffer is asynchronous in live: if it returns fewer values
    than requested, WAIT or SKIP — never proceed with partial data.









