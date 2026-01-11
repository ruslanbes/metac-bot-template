# Python 3.14 Compatibility Issue

## Problem

When running the bot with Python 3.14, you encounter the following error:

```
2026-01-09 12:56:56,218 - forecasting_tools.forecast_bots.forecast_bot - ERROR - Exception occurred during forecasting:
  + Exception Group Traceback (most recent call last):
  |   File ".../metac-bot-template/.venv/lib/python3.14/site-packages/forecasting_tools/forecast_bots/forecast_bot.py", line 341, in _run_individual_question_with_error_propagation
  |     return await self._run_individual_question(question)
  
...

RuntimeError: 4 errors occurred while forecasting: [ExceptionGroup("1 sub-exceptions -> Error while processing question url: 'https://www.metaculus.com/questions/578': 1 sub-exceptions -> All 1 research reports/predictions failed: Errors: ['APIError: litellm.APIError: APIError: OpenrouterException - Timeout context manager should be used inside a task']"
```

This error occurs in the `aiohttp` library used by `litellm` when making API calls to OpenRouter (or other LLM providers).

**Root Cause**: Python 3.14 is very new (released in October 2024), and there's a compatibility issue between `aiohttp` and Python 3.14's asyncio implementation. The timeout context manager in aiohttp requires being used within a proper asyncio task context, but Python 3.14's stricter asyncio enforcement is causing this to fail.

## Solutions

### Solution 1: Use Python 3.12 (Recommended)

The project specifies `python = "^3.12"` in `pyproject.toml`, which means Python 3.12 or 3.13 should work. Python 3.14 is too new and has compatibility issues.

**Steps to fix:**

1. Install Python 3.12:
   ```bash
   # On macOS with Homebrew
   brew install python@3.12
   ```

2. Tell Poetry to use Python 3.12:
   ```bash
   poetry env use python3.12
   # Or
   poetry env use /opt/homebrew/opt/python@3.12/bin/python3.12
   ```

3. Reinstall dependencies:
   ```bash
   poetry install
   ```

4. Run the bot again:
   ```bash
   poetry run python main.py --mode test_questions
   ```

### Solution 2: Update Dependencies (May Not Work Yet)

Try updating `litellm` and `aiohttp` to their latest versions, which might have Python 3.14 support:

```bash
poetry update litellm aiohttp
```


### Solution 3: Disable Research (Workaround)

If you need to test immediately and can't change Python versions, you can temporarily disable research by modifying `main.py`:

In the `run_research` method (around line 124), you can return an empty string:

```python
async def run_research(self, question: MetaculusQuestion) -> str:
    # Temporarily disable research to avoid Python 3.14 compatibility issue
    return ""
    
    # Original code commented out:
    # async with self._concurrency_limiter:
    #     research = ""
    #     researcher = self.get_llm("researcher")
    #     ...
```

**Note**: This will make forecasts less accurate since they won't have research context, but it will allow you to test the forecasting logic.

### Solution 4: Use a Different Research Provider

If you have AskNews credentials, you can configure the bot to use AskNews instead of an LLM for research, which might avoid the aiohttp issue:

```python
template_bot = SpringTemplateBot2026(
    # ... other parameters ...
    llms={
        "researcher": "asknews/news-summaries",  # Use AskNews instead of LLM
        # ... other llm configs ...
    },
)
```

## Verification

After applying Solution 1 (using Python 3.12), verify it works:

```bash
poetry run python --version
# Should show: Python 3.12.x

poetry run python main.py --mode test_questions
# Should run without the timeout error
```

## Why This Happens

The error trace shows:
```
File ".../aiohttp/helpers.py", line 678, in __enter__
    raise RuntimeError("Timeout context manager should be used inside a task")
```

This is aiohttp's internal check that ensures timeouts are only used within proper asyncio tasks. Python 3.14's stricter asyncio implementation is detecting that the context isn't properly set up, even though it should be.

## Related Issues

- This is a known issue with aiohttp and very new Python versions
- The `forecasting-tools` library uses `litellm`, which uses `aiohttp` for async HTTP requests
- The issue specifically affects OpenRouter API calls (and potentially other providers using aiohttp)

## Long-term Fix

The proper fix will come from:
1. `aiohttp` updating to fully support Python 3.14
2. `litellm` updating to use aiohttp in a way compatible with Python 3.14
3. `forecasting-tools` updating to use compatible versions

Until then, using Python 3.12 or 3.13 is the recommended solution.
