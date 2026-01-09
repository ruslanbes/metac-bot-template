# Using Perplexity for Research

## Overview

Perplexity AI is a powerful research tool that combines LLM capabilities with real-time web search. It's excellent for forecasting research because it:
- Provides up-to-date information from the web
- Cites sources for verification
- Synthesizes information from multiple sources
- Handles complex research queries well

## Analysis: Can Perplexity Be Used?

**Yes!** Perplexity can be used for research in the bot. The codebase shows:
1. ✅ Perplexity is mentioned in README as a supported research provider
2. ✅ `main_with_no_framework.py` has a working `call_perplexity()` implementation
3. ✅ The framework-based `main.py` can use Perplexity via OpenRouter or by adding direct API support

## Two Ways to Use Perplexity

### Option 1: Via OpenRouter (Easiest - Recommended)

Perplexity models are available through OpenRouter, which means you can use them without additional API keys if you already have `OPENROUTER_API_KEY`.

**Available Perplexity Models on OpenRouter:**
- `openrouter/perplexity/sonar` - General purpose research model
- `openrouter/perplexity/sonar-pro` - More powerful version
- `openrouter/perplexity/sonar-reasoning` - Enhanced reasoning capabilities

**Configuration:**

In `main.py`, modify the bot initialization to use Perplexity via OpenRouter:

```python
template_bot = SpringTemplateBot2026(
    # ... other parameters ...
    llms={
        "default": GeneralLlm(
            model="openrouter/google/gemini-3-pro-preview",
            temperature=0.3,
            timeout=40,
            allowed_tries=2,
        ),
        "researcher": "openrouter/perplexity/sonar",  # Use Perplexity for research
        "parser": "openrouter/google/gemini-3-flash-preview",
        "summarizer": "openrouter/google/gemini-3-flash-preview",
    },
)
```

**Advantages:**
- ✅ No additional API key needed (uses OpenRouter)
- ✅ Simple configuration (just change the model string)
- ✅ Works with existing code structure
- ✅ Can easily switch between models

**Disadvantages:**
- ⚠️ May have different rate limits than direct API
- ⚠️ Slightly higher latency (routed through OpenRouter)

### Option 2: Direct Perplexity API (More Control)

Use Perplexity's direct API for more control and potentially better rate limits.

**Step 1: Get Perplexity API Key**

1. Go to [Perplexity API Portal](https://www.perplexity.ai/settings/api)
2. Sign up or log in
3. Generate an API key
4. Add to your `.env` file:
   ```bash
   PERPLEXITY_API_KEY=your_perplexity_api_key_here
   ```

**Step 2: Modify `main.py` to Support Perplexity**

You need to add Perplexity support to the `run_research()` method. Add this code in the `run_research()` method, around line 182 (after the AskNews check, before the SmartSearcher check):

```python
async def run_research(self, question: MetaculusQuestion) -> str:
    async with self._concurrency_limiter:
        research = ""
        researcher = self.get_llm("researcher")

        # ... existing prompt code ...

        if isinstance(researcher, GeneralLlm):
            research = await researcher.invoke(prompt)
        elif (
            researcher == "asknews/news-summaries"
            or researcher == "asknews/deep-research/low-depth"
            or researcher == "asknews/deep-research/medium-depth"
            or researcher == "asknews/deep-research/high-depth"
        ):
            research = await AskNewsSearcher().call_preconfigured_version(
                researcher, prompt
            )
        elif researcher == "perplexity" or researcher == "perplexity/sonar":
            # Add Perplexity support here
            research = await self._call_perplexity(prompt)
        elif researcher.startswith("smart-searcher"):
            # ... existing SmartSearcher code ...
        # ... rest of the code ...
```

**Step 3: Add Perplexity Helper Method**

Add this method to the `SpringTemplateBot2026` class (around line 210, after the research methods):

```python
async def _call_perplexity(self, prompt: str) -> str:
    """
    Call Perplexity API directly for research.
    Requires PERPLEXITY_API_KEY environment variable.
    """
    import os
    import requests
    
    api_key = os.getenv("PERPLEXITY_API_KEY")
    if not api_key:
        raise ValueError("PERPLEXITY_API_KEY not found in environment variables")
    
    url = "https://api.perplexity.ai/chat/completions"
    headers = {
        "accept": "application/json",
        "authorization": f"Bearer {api_key}",
        "content-type": "application/json",
    }
    
    payload = {
        "model": "llama-3.1-sonar-huge-128k-online",  # Or "sonar-pro" for better quality
        "messages": [
            {
                "role": "system",
                "content": "You are an assistant to a superforecaster. Generate concise but detailed research summaries with citations."
            },
            {
                "role": "user",
                "content": prompt,
            },
        ],
    }
    
    response = requests.post(url=url, json=payload, headers=headers)
    if not response.ok:
        raise Exception(f"Perplexity API error: {response.text}")
    
    content = response.json()["choices"][0]["message"]["content"]
    return content
```

**Step 4: Configure Bot to Use Perplexity**

In the bot initialization (around line 750):

```python
template_bot = SpringTemplateBot2026(
    # ... other parameters ...
    llms={
        "default": GeneralLlm(
            model="openrouter/google/gemini-3-pro-preview",
            temperature=0.3,
            timeout=40,
            allowed_tries=2,
        ),
        "researcher": "perplexity",  # Use direct Perplexity API
        "parser": "openrouter/google/gemini-3-flash-preview",
        "summarizer": "openrouter/google/gemini-3-flash-preview",
    },
)
```

**Advantages:**
- ✅ Direct API access (potentially better rate limits)
- ✅ More control over model selection
- ✅ Can use latest Perplexity models

**Disadvantages:**
- ⚠️ Requires code modification
- ⚠️ Need separate API key
- ⚠️ More maintenance if Perplexity API changes

## Recommended Approach

**For most users: Use Option 1 (OpenRouter)**

It's simpler, requires no code changes, and works immediately. Just change the researcher model string.

**For advanced users: Use Option 2 (Direct API)**

If you need:
- Better rate limits
- Access to specific Perplexity models not on OpenRouter
- More control over the API calls

## Perplexity Models Comparison

| Model | Best For | Speed | Cost | Quality |
|-------|----------|-------|------|---------|
| **sonar** | General research | Fast | Low | Good |
| **sonar-pro** | Complex research | Medium | Medium | Excellent |
| **sonar-reasoning** | Deep analysis | Slower | Higher | Excellent |

## Configuration Examples

### Example 1: Basic Perplexity Research (OpenRouter)

```python
llms={
    "researcher": "openrouter/perplexity/sonar",
}
```

### Example 2: High-Quality Perplexity Research (OpenRouter)

```python
llms={
    "researcher": "openrouter/perplexity/sonar-pro",
}
```

### Example 3: Direct API with Custom Model

```python
# In _call_perplexity method, change model to:
"model": "sonar-pro",  # or "sonar-reasoning"
```

## Testing Perplexity Research

1. **Set up your API key** (if using direct API):
   ```bash
   echo "PERPLEXITY_API_KEY=your_key" >> .env
   ```

2. **Configure the bot** to use Perplexity (see examples above)

3. **Run a test**:
   ```bash
   poetry run python main.py --mode test_questions
   ```

4. **Check the logs** to see Perplexity research output:
   - Look for "Found Research for URL..." messages
   - Research should include citations and up-to-date information

## Tips for Best Results

1. **Use appropriate models**: 
   - `sonar` for quick research
   - `sonar-pro` for complex questions
   - `sonar-reasoning` for deep analysis

2. **Combine with context files**: 
   - Use `context/research_context.txt` to guide Perplexity's research focus
   - Add domain-specific instructions

3. **Monitor costs**: 
   - Perplexity via OpenRouter may have different pricing
   - Direct API has its own pricing structure
   - Check both for cost optimization

4. **Rate limits**: 
   - Perplexity has rate limits (check their docs)
   - Adjust `_max_concurrent_questions` if needed
   - Consider adding retry logic for rate limit errors

## Troubleshooting

### "PERPLEXITY_API_KEY not found"
- Make sure you've added the key to your `.env` file
- Restart your terminal/IDE after adding to `.env`
- Check the key name is exactly `PERPLEXITY_API_KEY`

### "Model not found" (OpenRouter)
- Check that the model string is correct
- Verify OpenRouter supports the model: https://openrouter.ai/models
- Try `openrouter/perplexity/sonar` first (most common)

### Rate Limit Errors
- Reduce `_max_concurrent_questions` in the bot class
- Add delays between requests
- Check your Perplexity API plan limits

### Research Quality Issues
- Try a more powerful model (`sonar-pro` instead of `sonar`)
- Improve your research context in `context/research_context.txt`
- Adjust the research prompt in `run_research()` method

## Comparison with Other Research Providers

| Provider | Strengths | Best For |
|----------|-----------|----------|
| **Perplexity** | Real-time web search, citations | Current events, breaking news |
| **AskNews** | News-focused, deep research | News-heavy questions |
| **SmartSearcher** | Customizable, multi-source | General research with control |
| **LLM-only** | Fast, no API needed | Simple questions, no web needed |

## References

- [Perplexity API Documentation](https://docs.perplexity.ai/)
- [OpenRouter Perplexity Models](https://openrouter.ai/models?q=perplexity)
- [Perplexity API Portal](https://www.perplexity.ai/settings/api)
- Example implementation in `main_with_no_framework.py` (lines 299-329)

## Quick Start (OpenRouter Method)

**Easiest way to get started:**

1. Make sure you have `OPENROUTER_API_KEY` in your `.env`
2. In `main.py`, change line ~750:
   ```python
   "researcher": "openrouter/perplexity/sonar",
   ```
3. Run the bot - Perplexity research will be used automatically!

No code changes needed, just update the model string. ✅
