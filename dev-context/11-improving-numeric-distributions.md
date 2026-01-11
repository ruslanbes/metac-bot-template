# Improving Numeric Distribution Granularity

## Problem Analysis

### Current Implementation

**Percentiles Requested**: Only 6 anchor points (lines 413-418 in `main.py`):
- Percentile 10
- Percentile 20
- Percentile 40
- Percentile 60
- Percentile 80
- Percentile 90

**What Happens Next**:
1. The framework (`NumericDistribution.from_question()`) takes these 6 percentiles
2. Interpolates them into a full CDF with **201 points** (default `cdf_size=201`)
3. The interpolation happens between the anchor points

### Why Distributions Look Rough

1. **Large gaps between percentiles**:
   - Gap between 20→40 (20 percentile points)
   - Gap between 40→60 (20 percentile points)
   - The interpolation has to "guess" what happens in these large gaps

2. **Missing edge percentiles**:
   - No percentile 5 or 95 (extreme tails)
   - No percentile 1 or 99 (very extreme tails)
   - Edge behavior is poorly defined

3. **Sparse middle percentiles**:
   - Only 40, 60 in the middle (20 percentile gap)
   - Missing 25, 30, 35, 45, 50, 55, 65, 70, 75
   - The "bell curve" shape isn't well-anchored

4. **Interpolation limitations**:
   - The framework uses linear or monotonic interpolation
   - With few anchor points, it can't capture smooth curves
   - Especially problematic at edges where distributions should taper smoothly

## Solutions

### Option 1: Add More Percentiles to the Prompt (Recommended)

**Add more anchor points**, especially:
- **Edges**: 5, 10, 15, 20, 25 (for left tail)
- **Middle**: 30, 40, 50, 60, 70 (for center/bell shape)
- **Right tail**: 75, 80, 85, 90, 95 (for right tail)

**Modified prompt section** (lines 411-419):
```python
The last thing you write is your final answer as:
"
Percentile 5: XX (very low tail)
Percentile 10: XX (low tail)
Percentile 15: XX
Percentile 20: XX
Percentile 25: XX
Percentile 30: XX
Percentile 40: XX
Percentile 50: XX (median)
Percentile 60: XX
Percentile 70: XX
Percentile 75: XX
Percentile 80: XX
Percentile 85: XX
Percentile 90: XX (high tail)
Percentile 95: XX (very high tail)
"
```

**Benefits**:
- ✅ More anchor points = smoother interpolation
- ✅ Better edge behavior (5th and 95th percentiles)
- ✅ Better middle shape (25, 30, 50, 70, 75)
- ✅ Framework can create smoother curves

**Trade-offs**:
- ⚠️ Longer prompts (slightly more tokens)
- ⚠️ LLM needs to think about more points (but this is good for quality)

### Option 2: Add Edge-Specific Instructions

**Add instructions about smooth edge behavior** (around line 409):

```python
Set wide 90/10 confidence intervals to account for unknown unknowns.

Important: When setting percentiles, ensure smooth transitions especially at the edges:
- The difference between percentile 5 and 10 should be gradual (not a sudden jump)
- The difference between percentile 90 and 95 should be gradual (not a sudden jump)
- The distribution should taper smoothly at both ends, resembling a bell curve
- Avoid sharp corners or sudden changes in the distribution shape
```

### Option 3: Add More Middle Percentiles

**Focus on the middle** where the "bell" shape matters most:

```python
The last thing you write is your final answer as:
"
Percentile 10: XX (low tail)
Percentile 20: XX
Percentile 25: XX
Percentile 30: XX
Percentile 40: XX
Percentile 50: XX (median - most likely outcome)
Percentile 60: XX
Percentile 70: XX
Percentile 75: XX
Percentile 80: XX
Percentile 90: XX (high tail)
"
```

### Option 4: Combine All Improvements (Best)

**Full improved prompt**:

```python
Set wide 90/10 confidence intervals to account for unknown unknowns.

When creating your distribution, ensure it has a smooth, bell-curve-like shape:
- The distribution should taper gradually at both edges (percentiles 5-10 and 90-95)
- The middle percentiles (40-60) should represent the most likely outcomes
- Avoid sudden jumps or sharp corners between adjacent percentiles
- Think about how a smooth probability distribution would look

The last thing you write is your final answer as:
"
Percentile 5: XX (very low tail - smooth transition from here)
Percentile 10: XX (low tail)
Percentile 15: XX
Percentile 20: XX
Percentile 25: XX
Percentile 30: XX
Percentile 40: XX
Percentile 50: XX (median - most likely outcome)
Percentile 60: XX
Percentile 70: XX
Percentile 75: XX
Percentile 80: XX
Percentile 85: XX
Percentile 90: XX (high tail)
Percentile 95: XX (very high tail - smooth transition to here)
"
```

## Implementation Location

**File**: `main.py`
**Function**: `_run_forecast_on_numeric()` (starts at line 360)
**Lines to modify**: 411-419 (the percentile output format)

## Current vs. Improved

### Current (6 percentiles):
```
10 → [gap] → 20 → [gap] → 40 → [gap] → 60 → [gap] → 80 → [gap] → 90
```

### Improved (15 percentiles):
```
5 → 10 → 15 → 20 → 25 → 30 → 40 → 50 → 60 → 70 → 75 → 80 → 85 → 90 → 95
```

**Result**: Much smoother interpolation, especially at edges and in the middle.

## Framework Behavior

The `forecasting-tools` framework:
- Takes your declared percentiles
- Interpolates between them to create 201 CDF points
- Uses monotonic interpolation (values must increase)
- Applies standardization/smoothing if `standardize_cdf=True`

**More anchor points = better interpolation quality**

## Testing

After making changes:
1. Run forecasts on a few numeric questions
2. Check the distribution graphs on Metaculus
3. Verify:
   - Smooth edges (no sharp corners at 10th/90th percentiles)
   - Bell-curve-like shape in the middle
   - Gradual tapering at both ends
   - No sudden jumps between percentiles

## Recommendation

**Use Option 4** (combine all improvements):
- Add 15 percentiles (5, 10, 15, 20, 25, 30, 40, 50, 60, 70, 75, 80, 85, 90, 95)
- Add instructions about smooth transitions
- Emphasize bell-curve shape
- Focus on edge behavior

This will give you the smoothest, most professional-looking distributions.
