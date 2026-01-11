# Model Recommendations for Bot Roles

## Understanding Each Role

### **Default** (Forecasting Model)
- **Purpose**: Generates the main reasoning and probability forecasts
- **Usage**: Called `predictions_per_research_report` times per research report (currently: 2 times)
- **Requirements**: 
  - Strong reasoning capabilities
  - Good at probabilistic thinking
  - Handles complex prompts well
- **Current**: `openrouter/google/gemini-3-pro-preview` ✅ (Good choice - this is your most important model)

### **Researcher**
- **Purpose**: Generates research summaries about questions
- **Usage**: Called once per question (or `research_reports_per_question` times)
- **Requirements**:
  - Good at synthesizing information
  - Can provide concise but detailed summaries
  - Doesn't need to be as powerful as the default model
- **Current**: `openrouter/perplexity/sonar` ✅ (Good choice - specialized for research)

### **Parser**
- **Purpose**: Parses free-form LLM reasoning into structured formats (probabilities, percentiles, option lists)
- **Usage**: Called multiple times per question (once per forecast attempt, with validation)
- **Requirements**:
  - Excellent at following structured output instructions
  - Reliable extraction of data from text
  - Good at format compliance
  - Speed matters (called frequently)
- **Current**: `openrouter/google/gemini-3-flash-preview` ✅ (Good choice - fast, cheap, reliable)

### **Summarizer**
- **Purpose**: Summarizes research reports (if `use_research_summary_to_forecast=True`)
- **Usage**: Only used if research summarization is enabled (currently disabled: `use_research_summary_to_forecast=False`)
- **Current**: `openrouter/google/gemini-3-flash-preview` (Not currently used)

## Recommendations

### Option 1: Cost-Optimized (Recommended for Most Users)

```python
llms={
    "default": GeneralLlm(
        model="openrouter/google/gemini-3-pro-preview",  # Keep powerful model for forecasting
        temperature=0.3,
        timeout=40,
        allowed_tries=2,
    ),
    "researcher": "asknews/news-summaries",  # Use specialized research tool (free/cheap)
    "parser": "openrouter/google/gemini-2.0-flash-exp",  # Fast, cheap, good at structured output
    "summarizer": "openrouter/google/gemini-2.0-flash-exp",  # If you enable summarization
},
```

**Why:**
- **Researcher**: AskNews is specialized for research and often free/cheap. If you don't have AskNews, use a cheaper model.
- **Parser**: Gemini Flash is fast, cheap, and excellent at structured output tasks. Much cheaper than Gemini 3 Pro.

### Option 2: All Gemini (Balanced)

```python
llms={
    "default": GeneralLlm(
        model="openrouter/google/gemini-3-pro-preview",
        temperature=0.3,
        timeout=40,
        allowed_tries=2,
    ),
    "researcher": "openrouter/google/gemini-2.0-flash-exp",  # Cheaper for research
    "parser": "openrouter/google/gemini-2.0-flash-exp",  # Fast and reliable for parsing
    "summarizer": "openrouter/google/gemini-2.0-flash-exp",
},
```

**Why:**
- **Researcher**: Flash is sufficient for research synthesis, much cheaper
- **Parser**: Flash is excellent at structured output and faster

### Option 3: Specialized Tools (Best Performance)

```python
llms={
    "default": GeneralLlm(
        model="openrouter/google/gemini-3-pro-preview",
        temperature=0.3,
        timeout=40,
        allowed_tries=2,
    ),
    "researcher": "asknews/deep-research/medium-depth",  # Best research quality
    "parser": "openrouter/openai/gpt-4o-mini",  # OpenAI models are excellent at structured output
    "summarizer": "openrouter/google/gemini-2.0-flash-exp",
},
```

**Why:**
- **Researcher**: AskNews deep research provides highest quality research
- **Parser**: GPT-4o-mini is very reliable for structured output parsing

### Option 4: Maximum Cost Savings

```python
llms={
    "default": GeneralLlm(
        model="openrouter/google/gemini-3-pro-preview",
        temperature=0.3,
        timeout=40,
        allowed_tries=2,
    ),
    "researcher": "openrouter/google/gemini-2.0-flash-exp",
    "parser": "openrouter/google/gemini-2.0-flash-exp",
    "summarizer": "openrouter/google/gemini-2.0-flash-exp",
},
```

**Why:**
- Use Flash for everything except the main forecasting
- Significant cost savings while maintaining good quality

## Current Configuration

```python
llms={
    "default": GeneralLlm(
        model="openrouter/google/gemini-3-pro-preview",
        temperature=0.3,
        timeout=160,
        allowed_tries=2,
    ),
    "summarizer": "openrouter/google/gemini-3-flash-preview",  # Not used (use_research_summary_to_forecast=False)
    "researcher": "openrouter/perplexity/sonar",
    "parser": "openrouter/google/gemini-3-flash-preview",
}
```

**Settings:**
- `predictions_per_research_report=2` (2 forecasts per research report)
- `research_reports_per_question=1` (1 research report per question)
- `use_research_summary_to_forecast=False` (summarizer not used)

## Model Comparison

| Model | Best For | Speed | Cost | Structured Output |
|-------|----------|-------|------|-------------------|
| **Gemini 3 Pro** | Complex reasoning, forecasting | Medium | High | Good |
| **Gemini 3 Flash** | Fast tasks, parsing, research | Very Fast | Low | Excellent |
| **Perplexity Sonar** | Research synthesis, web search | Medium | Medium | N/A |
| **Gemini 2.0 Flash** | Fast tasks, parsing, research | Very Fast | Low | Excellent |
| **GPT-4o-mini** | Structured output, parsing | Fast | Low-Medium | Excellent |
| **AskNews** | Research synthesis | Medium | Free/Low | N/A |

## Cost Analysis

Assuming you forecast on 100 questions with 2 predictions each (current setting):

**Current Setup:**
- Default: 200 calls × Gemini 3 Pro = High cost
- Researcher: 100 calls × Perplexity Sonar = Medium cost  
- Parser: 200 calls × Gemini 3 Flash = Low cost
- **Total**: Medium-High (optimized for cost)

**Previous Setup (All Gemini 3 Pro):**
- Default: 200 calls × Gemini 3 Pro = High cost
- Researcher: 100 calls × Gemini 3 Pro = High cost  
- Parser: 200 calls × Gemini 3 Pro = High cost
- **Total**: Very High

**Savings**: Current setup saves ~60-70% on researcher and parser costs compared to using Gemini 3 Pro for everything.

## Specific Recommendations

### For Researcher:
1. **Current**: `"openrouter/perplexity/sonar"` ✅ (Specialized for research with web search capabilities)
2. **Alternative**: `"asknews/news-summaries"` or `"asknews/deep-research/medium-depth"` (if you have AskNews)
3. **Budget option**: `"openrouter/google/gemini-3-flash-preview"` (fast, cheap, good quality)
4. **Other**: `"openrouter/openai/gpt-4o-mini"` (reliable, slightly more expensive)

### For Parser:
1. **Current**: `"openrouter/google/gemini-3-flash-preview"` ✅ (fast, cheap, excellent at structured output)
2. **Alternative**: `"openrouter/openai/gpt-4o-mini"` (very reliable, good at following instructions)
3. **Avoid**: Large expensive models (overkill for parsing)

## Testing Your Setup

After changing models, test with:
```bash
poetry run python main.py --mode test_questions
```

Check:
1. **Parser accuracy**: Are forecasts being parsed correctly?
2. **Research quality**: Is research useful for forecasting?
3. **Cost**: Monitor your API usage

## Current Setup Analysis

**Your current configuration is well-optimized:**
- ✅ **Default**: Gemini 3 Pro - Best choice for forecasting quality
- ✅ **Researcher**: Perplexity Sonar - Specialized for research with web search
- ✅ **Parser**: Gemini 3 Flash - Fast, cheap, reliable for structured output
- ✅ **Predictions per report**: 2 (good balance between quality and cost)

**This setup provides:**
- High-quality forecasts (Gemini 3 Pro)
- Good research quality (Perplexity Sonar with web search)
- Cost-effective parsing (Gemini 3 Flash)
- Reasonable total cost (2 predictions per report instead of 5)

## Final Recommendation

**Your current setup is excellent!** ✅

If you want to experiment:
- **For better research**: Consider `asknews/deep-research/medium-depth` if you have AskNews access
- **For cost savings**: You could try `gemini-3-flash-preview` for researcher (but Perplexity Sonar is better for research)
- **For more forecasts**: Increase `predictions_per_research_report` to 3-5 if you want more robust aggregations

The current configuration strikes a good balance between quality and cost.
