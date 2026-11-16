
{
  "indicator": {
    "name": "WeightedPricePDF",
    "description": "volume-weighted price distribution",
    "purpose": "Estimate the weighted median, upper quantile, lower quantile, and mode of the aggregated price distribution constructed from recent M1 candles.",
    "window": "chart_window"
  },

  "formula": {
    "definition": "Construct a weighted price histogram from the latest inp_period closed M1 candles. Each candle contributes a uniform price distribution over its Low-High range, weighted by its tick_volume. Distribute each candle's weight among histogram bins proportionally to geometric overlap. Calculate Q50, QUP, QDN, and Mode from the resulting aggregated histogram.",

    "explanation": "The indicator estimates the distribution of traded price levels over a rolling window. Each source candle is modeled as a uniform distribution between its Low and High prices. Its tick_volume represents the total weight of that distribution. The candle's weight is distributed across equal-width histogram bins in proportion to the length of overlap between the candle's price range and each bin. The aggregated histogram is used to extract three weighted quantiles and the most populated price bin.",

    "variables": {
      "inp_period": "Number of source M1 candles included in the calculation.",
      "histogram_bins": "Number of equal-width price intervals used to construct the histogram.",
      "upper_quantile": "Upper percentile threshold used to calculate QUP.",
      "lower_quantile": "Lower percentile threshold used to calculate QDN.",
      "H_i": "High price of source candle i.",
      "L_i": "Low price of source candle i.",
      "V_i": "Tick volume of source candle i.",
      "P_min": "Minimum Low price among all selected source candles.",
      "P_max": "Maximum High price among all selected source candles.",
      "W_j": "Accumulated weight of histogram bin j.",
      "B_j": "Price interval represented by histogram bin j.",
      "Delta_P": "Equal width of each histogram bin.",
      "Total_Weight": "Sum of all accumulated histogram bin weights.",
      "Q50": "Weighted median of the aggregated histogram.",
      "QUP": "Weighted upper quantile of the aggregated histogram.",
      "QDN": "Weighted lower quantile of the aggregated histogram.",
      "Mode": "Center price of the histogram bin with the highest accumulated weight."
    },

    "calculation_steps": [
      "Select the latest inp_period closed M1 candles.",
      "Find P_min as the minimum Low and P_max as the maximum High across the selected candles.",
      "Divide the interval [P_min, P_max] into histogram_bins equal-width bins.",
      "Calculate Delta_P = (P_max - P_min) / histogram_bins.",
      "Define each bin as a half-open interval [B_j_low, B_j_high), except the final bin, which includes P_max.",
      "For each source candle i, model its price distribution as uniform over [L_i, H_i] with total weight V_i.",
      "If H_i > L_i, calculate the geometric overlap length between the candle range and each histogram bin.",
      "For each overlapping bin, calculate the candle's contribution as V_i multiplied by the overlap length divided by (H_i - L_i).",
      "If H_i equals L_i, assign the entire V_i weight to the histogram bin containing that price. Do not discard or divide the weight.",
      "Aggregate all candle contributions to obtain W_j for every histogram bin.",
      "Ensure that the sum of contributions from each candle equals its original tick_volume, within floating-point tolerance.",
      "Calculate Total_Weight as the sum of all histogram bin weights.",
      "Calculate each quantile using the cumulative histogram weights and linear interpolation within the bin containing the target percentile.",
      "Calculate Q50 using percentile 50.",
      "Calculate QUP using percentile upper_quantile.",
      "Calculate QDN using percentile lower_quantile.",
      "Calculate Mode as the center price of the bin with the highest accumulated weight.",
      "If multiple bins have equal maximum accumulated weight, select the bin with the smallest index."
    ],

    "quantile_method": {
      "type": "linear_interpolation_within_bin",
      "target_weight": "Total_Weight * percentile / 100",
      "method": "Find the first bin j for which cumulative weight through bin j is greater than or equal to target_weight. Let cumulative_before be the accumulated weight of all bins preceding j. Assuming uniform weight distribution within the bin, calculate the quantile price as B_j_low + ((target_weight - cumulative_before) / W_j) * Delta_P.",
      "boundary_behavior": [
        "A target percentile of 0 corresponds to P_min.",
        "A target percentile of 100 corresponds to P_max.",
        "The interpolation must not divide by zero.",
        "The quantile must remain within the selected bin boundaries."
      ]
    },

    "histogram": {
      "bin_width": "(P_max - P_min) / histogram_bins",
      "weight_distribution": "For a non-zero-range candle, distribute tick_volume proportionally to geometric overlap length.",
      "zero_range_behavior": "Assign the entire tick_volume to the bin containing the candle price.",
      "boundary_policy": "Bins are half-open intervals [low, high), except the final bin, which includes its upper boundary.",
      "weight_conservation": "Each candle's total distributed weight must equal its original tick_volume within floating-point tolerance."
    }
  },

  "data": {
    "source": "M1 OHLC and tick_volume data.",
    "timeframe": "M1",
    "applied_price": "High and Low",
    "lookback_period": "inp_period closed M1 candles",
    "bar_policy": "Use only closed M1 source candles. Recalculate only when a new candle opens on the current chart timeframe, meaning the previous chart candle has closed.",
    "warmup_behavior": "Do not calculate or publish valid output values until inp_period valid closed M1 candles are available.",
    "history_dependency": "Rolling window. Each calculation uses the latest inp_period closed M1 candles. The result is independent of previous indicator calculation results."
  },

  "inputs": [
    {
      "name": "inp_period",
      "type": "int",
      "default": 60,
      "description": "Number of source M1 candles used to construct the price distribution.",
      "constraints": "Minimum: 30. No specified upper limit."
    },
    {
      "name": "upper_quantile",
      "type": "int",
      "default": 95,
      "description": "Upper quantile percentile used to calculate QUP.",
      "constraints": "Minimum: 65. Maximum: 95."
    },
    {
      "name": "lower_quantile",
      "type": "int",
      "default": 5,
      "description": "Lower quantile percentile used to calculate QDN.",
      "constraints": "Minimum: 5. Maximum: 35."
    },
    {
      "name": "histogram_bins",
      "type": "int",
      "default": 100,
      "description": "Number of equal-width price intervals used to construct the weighted histogram.",
      "constraints": "Minimum: 5. Maximum: 100."
    }
  ],

  "outputs": [
    {
      "id": "Q50",
      "name": "Q50",
      "description": "Weighted median price. The price at which 50% of the total histogram weight lies at or below this level.",
      "type": "line",
      "empty_value": "EMPTY_VALUE when calculation is unavailable."
    },
    {
      "id": "QUP",
      "name": "QUP",
      "description": "Upper weighted quantile. Its percentile is defined by upper_quantile. For example, upper_quantile = 95 means that 95% of the total histogram weight lies at or below this price.",
      "type": "line",
      "empty_value": "EMPTY_VALUE when calculation is unavailable."
    },
    {
      "id": "QDN",
      "name": "QDN",
      "description": "Lower weighted quantile. Its percentile is defined by lower_quantile. For example, lower_quantile = 5 means that 5% of the total histogram weight lies at or below this price.",
      "type": "line",
      "empty_value": "EMPTY_VALUE when calculation is unavailable."
    },
    {
      "id": "Mode",
      "name": "Mode",
      "description": "Center price of the histogram bin containing the highest accumulated weight. Represents the most populated price interval in the weighted price distribution.",
      "type": "line",
      "empty_value": "EMPTY_VALUE when calculation is unavailable."
    }
  ],

  "visualization": {
    "window": "chart_window",
    "plots": [
      {
        "output_id": "Q50",
        "style": "DRAW_LINE",
        "color": "Implementation default",
        "width": 1,
        "label": "Q50"
      },
      {
        "output_id": "QUP",
        "style": "DRAW_LINE",
        "color": "Implementation default",
        "width": 1,
        "label": "QUP"
      },
      {
        "output_id": "QDN",
        "style": "DRAW_LINE",
        "color": "Implementation default",
        "width": 1,
        "label": "QDN"
      },
      {
        "output_id": "Mode",
        "style": "DRAW_LINE",
        "color": "Implementation default",
        "width": 1,
        "label": "Mode"
      }
    ],
    "levels": [],
    "scaling": "Use actual calculated price values. Do not apply normalization or artificial scaling."
  },

  "special_cases": [
    "Each candle's total tick_volume weight must be preserved after distribution across histogram bins.",
    "A candle with High equal to Low must contribute its entire tick_volume to the bin containing that price.",
    "If P_max equals P_min, all source candles have zero price range. Assign all their tick_volume to the single effective price level and set Q50, QUP, QDN, and Mode to P_min.",
    "If Total_Weight is zero, all outputs must be EMPTY_VALUE.",
    "A candle with zero tick_volume contributes zero weight.",
    "Histogram bins must cover the entire interval from P_min to P_max without gaps or overlapping interiors.",
    "The final histogram bin includes P_max.",
    "Quantiles must use linear interpolation within the selected bin, assuming uniform weight distribution within that bin.",
    "The Mode must be selected from accumulated bin weights, not from candle frequencies or individual candle volumes.",
    "When multiple bins have equal maximum accumulated weight, select the bin with the smallest index.",
    "The indicator must use M1 source data regardless of the current chart timeframe.",
    "The indicator must update only when a new candle opens on the current chart timeframe.",
    "The source data must contain only closed M1 candles."
  ],

  "verification": [
    "Verify that the sum of all bin contributions from each non-zero-range candle equals its original tick_volume within floating-point tolerance.",
    "Verify that a zero-range candle contributes its entire tick_volume to exactly one containing bin.",
    "Verify that the sum of all histogram bin weights equals the sum of the selected candles' tick_volume values.",
    "Verify that Q50 corresponds to percentile 50 using linear interpolation within the selected bin.",
    "Verify that QUP corresponds to upper_quantile using linear interpolation within the selected bin.",
    "Verify that QDN corresponds to lower_quantile using linear interpolation within the selected bin.",
    "Verify that Mode is the center of the bin with maximum accumulated weight.",
    "Verify deterministic Mode selection when multiple bins have equal maximum weight.",
    "Verify that all quantiles remain within the histogram range.",
    "Verify that when P_max equals P_min, all outputs equal P_min if Total_Weight is positive.",
    "Verify that when Total_Weight is zero, all outputs are EMPTY_VALUE.",
    "Verify that changing the chart timeframe changes the calculation update schedule but does not change the M1 source data definition.",
    "Verify that repeated ticks within the same chart candle do not trigger a new calculation.",
    "Verify that a new chart candle triggers a calculation using the latest inp_period closed M1 candles."
  ]
}
