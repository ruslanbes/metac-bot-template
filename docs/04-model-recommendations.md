# Model Recommendations for Bot Roles

## Understanding Each Role

### **Default** (Forecasting Model)
- **Purpose**: Generates the main reasoning and probability forecasts
- **Usage**: Called `predictions_per_research_report` times per research report (default: 5 times)
- **Requirements**: 
  - Strong reasoning capabilities
  - Good at probabilistic thinking
  - Handles complex prompts well
- **Current**: Gemini 3 Pro ✅ (Good choice - this is your most important model)

### **Researcher**
- **Purpose**: Generates research summaries about questions
- **Usage**: Called once per question (or `research_reports_per_question` times)
- **Requirements**:
  - Good at synthesizing information
  - Can provide concise but detailed summaries
  - Doesn't need to be as powerful as the default model
- **Current**: Gemini 3 Pro ⚠️ (Likely overkill - consider cheaper alternatives)

### **Parser**
- **Purpose**: Parses free-form LLM reasoning into structured formats (probabilities, percentiles, option lists)
- **Usage**: Called multiple times per question (once per forecast attempt, with validation)
- **Requirements**:
  - Excellent at following structured output instructions
  - Reliable extraction of data from text
  - Good at format compliance
  - Speed matters (called frequently)
- **Current**: Gemini 3 Pro ⚠️ (Likely overkill - smaller models often work better)

### **Summarizer**
- **Purpose**: Summarizes research reports (if `use_research_summary_to_forecast=True`)
- **Usage**: Only used if research summarization is enabled (currently disabled)
- **Current**: Gemini 3 Pro (Not currently used)

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

## Model Comparison

| Model | Best For | Speed | Cost | Structured Output |
|-------|----------|-------|------|-------------------|
| **Gemini 3 Pro** | Complex reasoning, forecasting | Medium | High | Good |
| **Gemini 2.0 Flash** | Fast tasks, parsing, research | Very Fast | Low | Excellent |
| **GPT-4o-mini** | Structured output, parsing | Fast | Low-Medium | Excellent |
| **AskNews** | Research synthesis | Medium | Free/Low | N/A |

## Cost Analysis

Assuming you forecast on 100 questions with 5 predictions each:

**Current Setup (All Gemini 3 Pro):**
- Default: 500 calls × $X = High cost
- Researcher: 100 calls × $X = Medium cost  
- Parser: 500 calls × $X = High cost
- **Total**: Very High

**Recommended Setup (Option 1):**
- Default: 500 calls × $X = High cost (necessary)
- Researcher: 100 calls × AskNews (free) or Flash ($Y << $X) = Low cost
- Parser: 500 calls × Flash ($Y << $X) = Low cost
- **Total**: Much Lower (60-80% savings on parser/researcher)

## Specific Recommendations

### For Researcher:
1. **Best**: `"asknews/news-summaries"` or `"asknews/deep-research/medium-depth"` (if you have AskNews)
2. **Good**: `"openrouter/google/gemini-2.0-flash-exp"` (fast, cheap, good quality)
3. **Alternative**: `"openrouter/openai/gpt-4o-mini"` (reliable, slightly more expensive)

### For Parser:
1. **Best**: `"openrouter/google/gemini-2.0-flash-exp"` (fast, cheap, excellent at structured output)
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

## Final Recommendation

**Start with Option 1** (Cost-Optimized):
- Keep Gemini 3 Pro for forecasting (most important)
- Use AskNews for research (if available) or Gemini Flash
- Use Gemini Flash for parsing (fast, cheap, reliable)

This gives you the best balance of quality and cost. You can always upgrade the parser if you notice parsing errors.
