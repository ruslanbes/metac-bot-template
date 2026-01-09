# Troubleshooting: Running the Bot

## Common Issues When Running `poetry run python main.py --mode test_questions`

### ⚠️ CRITICAL: Python 3.14 Compatibility Issue

**If you see the error: `RuntimeError: Timeout context manager should be used inside a task`**

This is a **Python 3.14 compatibility issue** with aiohttp/litellm. See [python-3.14-issue.md](./python-3.14-issue.md) for detailed solutions.

**Quick fix**: Use Python 3.11, 3.12, or 3.13 instead:
```bash
poetry env use python3.12
poetry install
poetry run python main.py --mode test_questions
```

### Issue 1: Missing Environment Variables

**Problem**: The bot requires environment variables to be set, but no `.env` file exists in the repository.

**Symptoms**:
- `MetaculusClient` fails to authenticate
- LLM calls fail with authentication errors
- Import errors or missing configuration

**Solution**:

1. Create a `.env` file in the root directory with the following variables:

```bash
# Required: Metaculus API token
METACULUS_TOKEN=your_metaculus_token_here

# Required: At least one LLM provider API key
OPENAI_API_KEY=your_openai_key_here
# OR
OPENROUTER_API_KEY=your_openrouter_key_here
# OR
ANTHROPIC_API_KEY=your_anthropic_key_here

# Optional: Research provider keys (at least one recommended)
ASKNEWS_CLIENT_ID=your_asknews_client_id
ASKNEWS_SECRET=your_asknews_secret
# OR
PERPLEXITY_API_KEY=your_perplexity_key
# OR
EXA_API_KEY=your_exa_key
```

2. Get your API keys:
   - **METACULUS_TOKEN**: Create at https://metaculus.com/aib
   - **OPENROUTER_API_KEY**: Get free credits at https://forms.gle/aQdYMq9Pisrf1v7d8 or create at https://openrouter.ai/
   - **OPENAI_API_KEY**: Create at https://platform.openai.com/api-keys
   - **ASKNEWS**: See README for instructions

### Issue 2: Missing METACULUS_TOKEN

**Problem**: The `MetaculusClient()` initialization on line 689 requires a valid token.

**Error Pattern**:
```
AuthenticationError: Invalid token
# OR
ValueError: METACULUS_TOKEN not found
# OR
requests.exceptions.HTTPError: 401 Unauthorized
```

**Solution**: 
- Ensure `METACULUS_TOKEN` is set in your `.env` file
- Verify the token is valid at https://metaculus.com/aib
- Make sure the `.env` file is in the root directory (same level as `main.py`)

### Issue 3: Missing LLM API Keys

**Problem**: The bot needs at least one LLM provider to generate forecasts. Without it, the `get_llm()` calls will fail.

**Error Pattern**:
```
litellm.exceptions.AuthenticationError: No API key found
# OR
openai.error.AuthenticationError: Invalid API key
# OR
AttributeError: 'NoneType' object has no attribute 'invoke'
```

**Solution**:
- Set at least one of: `OPENAI_API_KEY`, `OPENROUTER_API_KEY`, or `ANTHROPIC_API_KEY`
- If using OpenRouter, you can access multiple models with a single key
- The bot will use default models if no custom `llms` parameter is provided

### Issue 4: Missing Research Provider Keys

**Problem**: While not strictly required, the bot expects at least one research provider for the `run_research()` method.

**Error Pattern**:
- Research may return empty string or "No research done"
- Bot will still work but forecasts may be less accurate

**Solution**:
- Set at least one research provider key (AskNews, Perplexity, or Exa)
- Or modify the code to skip research if no provider is available

### Issue 5: Import Errors

**Problem**: Missing dependencies or incorrect Python version.

**Error Pattern**:
```
ModuleNotFoundError: No module named 'forecasting_tools'
# OR
ImportError: cannot import name 'ForecastBot' from 'forecasting_tools'
```

**Solution**:
1. Ensure dependencies are installed:
   ```bash
   poetry install
   ```

2. Verify Python version (requires 3.11+):
   ```bash
   poetry run python --version
   ```

3. If using a virtual environment, activate it:
   ```bash
   poetry shell
   ```

### Issue 6: Network/API Errors

**Problem**: API rate limits, network issues, or invalid endpoints.

**Error Pattern**:
```
requests.exceptions.ConnectionError
# OR
RateLimitError: Rate limit exceeded
# OR
HTTPError: 429 Too Many Requests
```

**Solution**:
- Check your internet connection
- Verify API keys have sufficient credits/quota
- Wait and retry if rate limited
- Consider adjusting `_max_concurrent_questions` in the bot class

### Issue 7: Question Fetching Errors

**Problem**: The test questions URLs might be invalid or the questions might be closed.

**Error Pattern**:
```
HTTPError: 404 Not Found
# OR
ValueError: Question not found
```

**Solution**:
- Verify the example question URLs in the code are still valid
- Check that questions are still open on Metaculus
- Update the `EXAMPLE_QUESTIONS` list with current question URLs

## Step-by-Step Setup Checklist

1. ✅ Install Poetry (if not already installed)
   ```bash
   pipx install poetry
   ```

2. ✅ Install dependencies
   ```bash
   poetry install
   ```

3. ✅ Create `.env` file in root directory
   ```bash
   touch .env
   ```

4. ✅ Add required environment variables to `.env`
   - METACULUS_TOKEN (required)
   - At least one LLM API key (required)
   - At least one research provider key (recommended)

5. ✅ Test the setup
   ```bash
   poetry run python main.py --mode test_questions
   ```

## Quick Test Without Submitting

To test the bot without submitting predictions, modify line 672 in `main.py`:

```python
publish_reports_to_metaculus=False,  # Change from True to False
```

This will run the forecasting logic but won't submit to Metaculus.

## Getting Help

If you continue to experience issues:

1. Check the full error traceback - it will indicate the specific problem
2. Verify all environment variables are set correctly
3. Test with a single question URL first
4. Check the Metaculus Discord: https://discord.com/invite/NJgCC2nDfh
5. Contact: ben [at] metaculus [.com]

## Example .env File Template

```bash
# Metaculus Authentication (REQUIRED)
METACULUS_TOKEN=your_token_here

# LLM Provider (REQUIRED - at least one)
OPENROUTER_API_KEY=your_openrouter_key_here
# OR
OPENAI_API_KEY=your_openai_key_here
# OR  
ANTHROPIC_API_KEY=your_anthropic_key_here

# Research Providers (OPTIONAL - at least one recommended)
ASKNEWS_CLIENT_ID=your_client_id
ASKNEWS_SECRET=your_secret

# Alternative research providers
PERPLEXITY_API_KEY=your_perplexity_key
EXA_API_KEY=your_exa_key
```

**Note**: The `.env` file should be added to `.gitignore` to keep your API keys secure. Never commit API keys to version control.
