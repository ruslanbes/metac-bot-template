# Comment Posting Control

## Current Behavior

**Current Setting**: `publish_reports_to_metaculus=True` (line 733)

This setting is **global** and applies to **all modes**:
- ✅ Tournament mode (AI Competition + MiniBench)
- ✅ Metaculus Cup mode
- ✅ Test questions mode

## How Comments Are Posted

The `publish_reports_to_metaculus` parameter controls whether:
1. **Forecasts are submitted** to Metaculus
2. **Comments/explanations are posted** with the forecasts

This is handled by the `ForecastBot` parent class in the `forecasting-tools` framework. When `publish_reports_to_metaculus=True`, the framework automatically posts both the forecast and the reasoning/explanation as a comment.

## Current Configuration Analysis

Looking at the code:

```python
template_bot = SpringTemplateBot2026(
    # ... other settings ...
    publish_reports_to_metaculus=True,  # Line 733 - GLOBAL setting
    # ...
)

# All modes use the same bot instance:
if run_mode == "tournament":
    # Uses publish_reports_to_metaculus=True
    template_bot.forecast_on_tournament(...)
elif run_mode == "metaculus_cup":
    # Uses publish_reports_to_metaculus=True
    template_bot.forecast_on_tournament(...)
elif run_mode == "test_questions":
    # Uses publish_reports_to_metaculus=True
    template_bot.forecast_questions(...)
```

**Current behavior**: Comments are posted in **all modes** because `publish_reports_to_metaculus=True` is set globally.

## Desired Behavior

You want:
- ✅ **Comments for AI Competition tournament only** (tournament mode with `CURRENT_AI_COMPETITION_ID`)
- ❌ **No comments for other modes** (metaculus_cup, test_questions, or custom tournaments)

## Solution: Conditional Bot Configuration

To achieve this, you need to create separate bot instances for different modes, or conditionally set `publish_reports_to_metaculus` based on the mode.

### Option 1: Create Bot Instances Per Mode (Recommended)

Modify the code to create bot instances with different settings:

```python
client = MetaculusClient()

# Create bot for AI Competition (with comments)
ai_competition_bot = SpringTemplateBot2026(
    research_reports_per_question=1,
    predictions_per_research_report=5,
    use_research_summary_to_forecast=False,
    publish_reports_to_metaculus=True,  # ✅ Comments enabled
    folder_to_save_reports_to=None,
    skip_previously_forecasted_questions=True,
    extra_metadata_in_explanation=True,
    llms={...},  # Your LLM config
)

# Create bot for other modes (no comments)
other_modes_bot = SpringTemplateBot2026(
    research_reports_per_question=1,
    predictions_per_research_report=5,
    use_research_summary_to_forecast=False,
    publish_reports_to_metaculus=False,  # ❌ Comments disabled
    folder_to_save_reports_to=None,
    skip_previously_forecasted_questions=True,
    extra_metadata_in_explanation=True,
    llms={...},  # Your LLM config
)

if run_mode == "tournament":
    # Use bot with comments for AI Competition
    seasonal_tournament_reports = asyncio.run(
        ai_competition_bot.forecast_on_tournament(
            client.CURRENT_AI_COMPETITION_ID, return_exceptions=True
        )
    )
    # Use bot without comments for MiniBench
    minibench_reports = asyncio.run(
        other_modes_bot.forecast_on_tournament(
            client.CURRENT_MINIBENCH_ID, return_exceptions=True
        )
    )
    forecast_reports = seasonal_tournament_reports + minibench_reports
elif run_mode == "metaculus_cup":
    # Use bot without comments
    client.CURRENT_METACULUS_CUP_ID = 32921
    other_modes_bot.skip_previously_forecasted_questions = False
    forecast_reports = asyncio.run(
        other_modes_bot.forecast_on_tournament(
            client.CURRENT_METACULUS_CUP_ID, return_exceptions=True
        )
    )
elif run_mode == "test_questions":
    # Use bot without comments
    other_modes_bot.skip_previously_forecasted_questions = False
    questions = [...]
    forecast_reports = asyncio.run(
        other_modes_bot.forecast_questions(questions, return_exceptions=True)
    )
```

### Option 2: Conditional Setting (Simpler but less flexible)

Create bot once, then modify the setting before each mode:

```python
template_bot = SpringTemplateBot2026(
    # ... settings ...
    publish_reports_to_metaculus=False,  # Default: no comments
    # ...
)

client = MetaculusClient()
if run_mode == "tournament":
    # Enable comments for AI Competition
    template_bot.publish_reports_to_metaculus = True
    seasonal_tournament_reports = asyncio.run(
        template_bot.forecast_on_tournament(
            client.CURRENT_AI_COMPETITION_ID, return_exceptions=True
        )
    )
    # Disable comments for MiniBench
    template_bot.publish_reports_to_metaculus = False
    minibench_reports = asyncio.run(
        template_bot.forecast_on_tournament(
            client.CURRENT_MINIBENCH_ID, return_exceptions=True
        )
    )
    forecast_reports = seasonal_tournament_reports + minibench_reports
elif run_mode == "metaculus_cup":
    # Comments disabled (default)
    template_bot.publish_reports_to_metaculus = False
    # ... rest of code
elif run_mode == "test_questions":
    # Comments disabled (default)
    template_bot.publish_reports_to_metaculus = False
    # ... rest of code
```

**Note**: This approach assumes `publish_reports_to_metaculus` can be modified after bot creation. You may need to check if the framework allows this.

### Option 3: Tournament-Specific Check (Most Flexible)

Check the tournament ID and conditionally enable comments:

```python
template_bot = SpringTemplateBot2026(
    # ... settings ...
    publish_reports_to_metaculus=False,  # Default: no comments
    # ...
)

def should_post_comments(tournament_id) -> bool:
    """Return True only for AI Competition tournament"""
    client = MetaculusClient()
    return tournament_id == client.CURRENT_AI_COMPETITION_ID

client = MetaculusClient()
if run_mode == "tournament":
    # AI Competition - enable comments
    template_bot.publish_reports_to_metaculus = should_post_comments(
        client.CURRENT_AI_COMPETITION_ID
    )
    seasonal_tournament_reports = asyncio.run(
        template_bot.forecast_on_tournament(
            client.CURRENT_AI_COMPETITION_ID, return_exceptions=True
        )
    )
    # MiniBench - disable comments
    template_bot.publish_reports_to_metaculus = should_post_comments(
        client.CURRENT_MINIBENCH_ID
    )
    minibench_reports = asyncio.run(
        template_bot.forecast_on_tournament(
            client.CURRENT_MINIBENCH_ID, return_exceptions=True
        )
    )
    forecast_reports = seasonal_tournament_reports + minibench_reports
# ... other modes
```

## Verification

To verify the current behavior:

1. **Check if comments are posted in tournament mode:**
   - Run: `poetry run python main.py --mode tournament`
   - Check if comments appear on AI Competition questions
   - Check if comments appear on MiniBench questions

2. **Check if comments are posted in other modes:**
   - Run: `poetry run python main.py --mode metaculus_cup`
   - Check if comments appear on Metaculus Cup questions
   - Run: `poetry run python main.py --mode test_questions`
   - Check if comments appear on test questions

## Recommendation

**Use Option 1** (separate bot instances) for clarity and reliability:
- Clear separation of concerns
- No risk of state leakage between modes
- Easy to understand and maintain
- Works regardless of framework implementation details

## Important Notes

1. **`publish_reports_to_metaculus` controls both forecasts AND comments**: You cannot post forecasts without comments, or comments without forecasts, using this parameter.

2. **Framework behavior**: The `forecasting-tools` framework handles comment posting internally. The exact mechanism depends on the framework implementation.

3. **Testing**: After making changes, test each mode to verify comments are posted/not posted as expected.
