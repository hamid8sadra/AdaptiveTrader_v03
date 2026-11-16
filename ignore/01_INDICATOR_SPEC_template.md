You are a senior MQL5 indicator developer. Use the specification below as the
sole mathematical source of truth. Do not invent, approximate, or simplify any
formula. Emit exactly one compile-ready MQL5 indicator following the frozen
rulebook (Prompt 2).

Platform: MT5 only. Language: MQL5.

[INDICATOR SPECIFICATION — JSON]
{
  "indicator": {
    "name": "<NAME>",
    "description": "<SHORT_DESCRIPTION>",
    "purpose": "<WHAT_THIS_INDICATOR_MEASURES>",
    "output_filename": "Ind_<NAME>.mq5"
  },

  "formula": {
    "spec": "<MATHEMATICAL_FORMULA + PRECISE_EXPLANATION, single authoritative field>",
    "variables": {
      "<VARIABLE>": "<DEFINITION>"
    },
    "calculation_steps": ["<STEP_1>", "<STEP_2>"]
  },

  "data": {
    "source": "<PRICE | VOLUME | TIME | OTHER>",
    "timeframe": "<CHART_TIMEFRAME | SPECIFIC_TIMEFRAME>",
    "applied_price": "<OPEN | HIGH | LOW | CLOSE | TYPICAL | OTHER>",
    "lookback_period": "<BARS_REQUIRED_BEFORE_FIRST_VALID_VALUE>",
    "bar_policy": "<CURRENT_BAR | CLOSED_BAR>",
    "history_dependency": "<INDEPENDENT | RECURSIVE | OTHER>"
  },

  "inputs": [
    {
      "name": "<INPUT_NAME>",
      "type": "<int | double | bool | enum>",
      "default": "<VALUE>",
      "description": "<TEXT>",
      "constraints": "<VALID_RANGE_OR_CONDITIONS>"
    }
  ],

  "outputs": [
    {
      "id": "<OUTPUT_ID>",
      "name": "<OUTPUT_NAME>",
      "description": "<MATHEMATICAL_MEANING>",
      "type": "<line | histogram | color_line | calculation_only>",
      "empty_value": "<WHAT_TO_WRITE_WHEN_UNCOMPUTABLE — use EMPTY_VALUE unless stated>"
    }
  ],

  "visualization": {
    "window": "<chart_window | separate_window>",
    "plots": [
      {
        "output_id": "<OUTPUT_ID>",
        "style": "<DRAW_STYLE>",
        "color": "<COLOR>",
        "width": "<WIDTH>",
        "label": "<DISPLAY_LABEL>"
      }
    ],
    "levels": [
      { "value": "<LEVEL>", "color": "<COLOR>", "style": "<STYLE>" }
    ],
    "scaling": "<SCALING_REQUIREMENTS>"
  },

  "special_cases": [
    "<EDGE_CASE_REQUIRING_A_GUARD — non-empty-value cases only>"
  ],

  "verification": [
    {
      "description": "<TEST_DESCRIPTION>",
      "input_data": "<INPUT_VALUES>",
      "expected_output": "<EXPECTED_RESULT>"
    }
  ]
}
[END SPECIFICATION]

Note: verification[] entries are documentation-only; do not attempt to execute
them. Each special_cases[] entry requires a corresponding guard in the code.