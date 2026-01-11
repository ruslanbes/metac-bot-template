# Forecasting on Specific Tournaments

## Overview

Yes, you can run the bot on any tournament if you know the tournament ID. The `forecast_on_tournament()` method accepts tournament IDs (both integers and string slugs).

## Tournament ID Formats

Tournament IDs can be:
- **Integer**: `32564`, `32921`, etc.
- **String slug**: `"spring-aib-2026"`, `"fall-aib-2025"`, `"ai-2027"`, `"minibench"`

## Method: Modify the Code Directly (Simplest and Only Method)

The bot uses the **`tournament`** mode to forecast on tournaments. To use a specific tournament ID, simply modify the code.

### For Tournament Mode

Edit `main.py` around line 754-755. Replace `client.CURRENT_AI_COMPETITION_ID` with your tournament ID:

**Current code (line 754-755):**
```python
seasonal_tournament_reports = asyncio.run(
    template_bot.forecast_on_tournament(
        client.CURRENT_AI_COMPETITION_ID, return_exceptions=True
    )
)
```

**Change to:**
```python
seasonal_tournament_reports = asyncio.run(
    template_bot.forecast_on_tournament(
        32564, return_exceptions=True  # Replace 32564 with your tournament ID
    )
)
```

**Or if you want to forecast on ONLY your custom tournament (skip MiniBench):**
```python
if run_mode == "tournament":
    # Forecast on a specific tournament only
    CUSTOM_TOURNAMENT_ID = 32564  # Replace with your tournament ID
    forecast_reports = asyncio.run(
        template_bot.forecast_on_tournament(
            CUSTOM_TOURNAMENT_ID, return_exceptions=True
        )
    )
    # Comment out or remove the minibench_reports section
```

**Or if you want to forecast on your custom tournament AND MiniBench:**
```python
if run_mode == "tournament":
    # Forecast on your custom tournament
    CUSTOM_TOURNAMENT_ID = 32564  # Replace with your tournament ID
    seasonal_tournament_reports = asyncio.run(
        template_bot.forecast_on_tournament(
            CUSTOM_TOURNAMENT_ID, return_exceptions=True
        )
    )
    # Also forecast on MiniBench
    minibench_reports = asyncio.run(
        template_bot.forecast_on_tournament(
            client.CURRENT_MINIBENCH_ID, return_exceptions=True
        )
    )
    forecast_reports = seasonal_tournament_reports + minibench_reports
```

### For Metaculus Cup Mode

Already supports custom IDs (see line 768). Just change the ID:

```python
elif run_mode == "metaculus_cup":
    client.CURRENT_METACULUS_CUP_ID = 32921  # Change to your tournament ID
    # ... rest of code
```

## How `forecast_on_tournament()` Works

The `forecast_on_tournament(tournament_id)` method:
- Accepts tournament ID as parameter (integer or string)
- Fetches all open questions from that tournament
- Forecasts on each question
- Returns a list of forecast reports

**Parameter**: `tournament_id` can be:
- Integer: `32564`
- String slug: `"spring-aib-2026"`

**Example usage in code:**
```python
# Forecast on tournament with ID 32564
forecast_reports = asyncio.run(
    template_bot.forecast_on_tournament(
        32564, return_exceptions=True
    )
)

# Forecast on tournament with slug "spring-aib-2026"
forecast_reports = asyncio.run(
    template_bot.forecast_on_tournament(
        "spring-aib-2026", return_exceptions=True
    )
)
```

## Finding Tournament IDs

### Method 1: From URL
If you have a tournament URL like:
- `https://www.metaculus.com/tournaments/32564/`
- The ID is `32564`

### Method 2: From Tournament Slug
If you have a tournament URL like:
- `https://www.metaculus.com/tournaments/spring-aib-2026/`
- The slug is `"spring-aib-2026"`

### Method 3: From Question URL
If you have a question in a tournament:
- `https://www.metaculus.com/questions/12345/...`
- Check the question details via API or inspect the page to find the tournament ID

### Method 4: List Tournaments via API

You can list tournaments using the Metaculus API:

```python
from forecasting_tools import MetaculusClient

client = MetaculusClient()
# The client has constants like:
# - client.CURRENT_AI_COMPETITION_ID
# - client.CURRENT_MINIBENCH_ID
# - client.CURRENT_METACULUS_CUP_ID
```

## Examples

### Example 1: Forecast on AXC 2025 Tournament

```python
# In main.py, modify tournament mode:
if run_mode == "tournament":
    AXC_2025_TOURNAMENT_ID = 32564
    forecast_reports = asyncio.run(
        template_bot.forecast_on_tournament(
            AXC_2025_TOURNAMENT_ID, return_exceptions=True
        )
    )
```

### Example 2: Forecast on AI 2027 Tournament

```python
# Using string slug:
if run_mode == "tournament":
    AI_2027_TOURNAMENT_ID = "ai-2027"
    forecast_reports = asyncio.run(
        template_bot.forecast_on_tournament(
            AI_2027_TOURNAMENT_ID, return_exceptions=True
        )
    )
```

### Example 3: Forecast on Multiple Tournaments

```python
if run_mode == "tournament":
    tournament_ids = [32564, "spring-aib-2026", 32921]
    all_reports = []
    
    for tournament_id in tournament_ids:
        reports = asyncio.run(
            template_bot.forecast_on_tournament(
                tournament_id, return_exceptions=True
            )
        )
        all_reports.extend(reports)
    
    forecast_reports = all_reports
```

## Common Tournament IDs

From `main_with_no_framework.py`, here are some known tournament IDs:

| Tournament | ID | Type |
|------------|----|----|
| Q4 2024 AI Benchmarking | `32506` | Integer |
| Q1 2025 AI Benchmarking | `32627` | Integer |
| Fall 2025 AI Benchmarking | `"fall-aib-2025"` | String |
| Spring 2026 AI Benchmarking | `"spring-aib-2026"` | String |
| AXC 2025 | `32564` | Integer |
| AI 2027 | `"ai-2027"` | String |
| MiniBench | `"minibench"` | String |
| Metaculus Cup | `32921` | Integer (current) |

## Important Notes

1. **Tournament IDs can change**: Some tournaments use slugs that might change, while integer IDs are more stable
2. **Access permissions**: Make sure your `METACULUS_TOKEN` has access to the tournament
3. **Question types**: Different tournaments may have different question types enabled
4. **Skip previously forecasted**: Set `skip_previously_forecasted_questions=True` to avoid re-forecasting

## Quick Reference

**To forecast on a specific tournament:**

1. **Open `main.py`** and find line 754-755
2. **Replace** `client.CURRENT_AI_COMPETITION_ID` with your tournament ID:

```python
# Line 754-755: Change from:
seasonal_tournament_reports = asyncio.run(
    template_bot.forecast_on_tournament(
        client.CURRENT_AI_COMPETITION_ID, return_exceptions=True
    )
)

# To:
seasonal_tournament_reports = asyncio.run(
    template_bot.forecast_on_tournament(
        32564, return_exceptions=True  # Your tournament ID here
    )
)
```

3. **Run the bot:**
```bash
poetry run python main.py --mode tournament
```

**Note**: The `tournament` mode will also forecast on MiniBench by default (lines 758-761). If you only want your custom tournament, comment out or remove the minibench section.

## Troubleshooting

### "Tournament not found" Error
- Verify the tournament ID is correct
- Check if the tournament is still active
- Ensure your API token has access

### "No questions found"
- The tournament might not have any open questions
- Check the tournament status on Metaculus website
- Try a different tournament ID

### Type Errors
- Tournament IDs can be integers (`32564`) or strings (`"spring-aib-2026"`)
- Make sure you're using the correct type
- String slugs should be in quotes if hardcoding
