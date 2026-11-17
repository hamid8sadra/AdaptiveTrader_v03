   - main project directory (will refer as "git directory"): "C:\Users\sadra\PycharmProjects\AdaptiveTrader\AdaptiveTrader_v03"
   - MQL5 directory (will refer as mlq directory): "C:\Users\sadra\AppData\Roaming\MetaQuotes\Terminal\D0E8209F77C8CF37AD8BF550E51FF075\MQL5"
   - all final created files must have a saved copy in git directory. for testing mql5 codes agent should copy file in mql directory. after testing mql files, the temprory files should be deleted and requsted files should move to git directory.

================================================================
## FILE PLACEMENT & PROJECT LAYOUT
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