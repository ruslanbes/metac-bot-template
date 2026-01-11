# Confidence Interval Instruction

## What Was Removed

**Instruction**: "Set wide 95/5 confidence intervals to account for unknown unknowns."

**Location**: 
- Numeric questions: `_run_forecast_on_numeric()` (previously line 409)
- Date questions: `_run_forecast_on_date()` (previously line 512)

## What It Does

This instruction tells the LLM to:
1. **Be humble/uncertain** - Reminds the model to account for uncertainty
2. **Set wide distributions** - Encourages larger gaps between percentiles
3. **Account for unknown unknowns** - Reminds to consider unexpected scenarios

**95/5 confidence interval** means:
- Range from 5th to 95th percentile
- 90% confidence interval (95 - 5 = 90)
- "I'm 90% confident the answer falls between these values"

## Why It Was Removed

- Testing whether the LLM naturally produces appropriate uncertainty without explicit instruction
- Other instructions (thinking about unexpected scenarios) may be sufficient
- Simplifying the prompt

## What Happens Without It

**The LLM will still:**
- ✅ Provide all requested percentiles (5, 10, 20, 30, 40, 50, 60, 70, 80, 90, 95)
- ✅ Follow other instructions (think about unexpected scenarios, etc.)
- ✅ Produce valid distributions

**But it might:**
- ⚠️ Be more overconfident (narrower distributions)
- ⚠️ Set percentiles closer together (smaller gaps)
- ⚠️ Underestimate uncertainty
- ⚠️ Produce distributions that are too tight

## Other Instructions That Help

The prompt still includes:
- "(e) A brief description of an unexpected scenario that results in a low outcome"
- "(f) A brief description of an unexpected scenario that results in a high outcome"

These encourage thinking about extremes, but may not be sufficient to ensure wide distributions.

## When to Bring It Back

Consider restoring this instruction if you observe:
1. **Overconfident forecasts** - Distributions are consistently too narrow
2. **Poor calibration** - Forecasts are too certain (actual outcomes fall outside predicted ranges too often)
3. **Tight percentiles** - The gaps between percentiles (especially 5-10 and 90-95) are too small
4. **Missing tail events** - The model doesn't account for extreme scenarios well enough

## Alternative Phrasings

If you want to bring it back but with different wording:

**Option 1 (Original)**:
```
Set wide 95/5 confidence intervals to account for unknown unknowns.
```

**Option 2 (More explicit)**:
```
Remember to set wide confidence intervals (from 5th to 95th percentile) to account for uncertainty and unknown unknowns.
```

**Option 3 (More detailed)**:
```
Good forecasters are humble and account for uncertainty. Set wide confidence intervals (5th to 95th percentile) to ensure you're not overconfident.
```

**Option 4 (Softer)**:
```
Consider setting wide confidence intervals to account for uncertainty and unexpected scenarios.
```

## Testing Recommendation

After removing the instruction:
1. Run forecasts on several numeric/date questions
2. Check if distributions are appropriately wide
3. Verify that percentiles 5 and 95 are meaningfully different from 10 and 90
4. Monitor calibration over time

If distributions become too narrow or forecasts become overconfident, restore the instruction.

## Current Status

**Status**: ✅ Reintroduced (as of 2026-01-09)

**Location**: 
- Numeric questions: `_run_forecast_on_numeric()` (line ~485)
- Date questions: `_run_forecast_on_date()` (line ~588)

**Instruction**: "Set wide 95/5 confidence intervals to account for unknown unknowns."
