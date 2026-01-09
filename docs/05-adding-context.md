# Adding Context to Your Forecasting Bot

## Overview

You can add custom context to your bot in two ways:
1. **Research Context**: Added to every research prompt
2. **Forecast Context**: Added to every forecast prompt (binary, numeric, multiple choice, date, conditional)

Context is stored in simple text files in the `context/` directory.

## How It Works

### File Structure

```
metac-bot-template/
├── context/
│   ├── research_context.txt    # Context for research prompts
│   └── forecast_context.txt    # Context for forecast prompts
└── main.py
```

### Implementation Details

1. **Context Loading**: The bot automatically loads context files when initialized
2. **Automatic Integration**: Context is automatically added to all relevant prompts
3. **Graceful Handling**: If context files don't exist, the bot works normally (empty context)

## Using Context Files

### Research Context (`context/research_context.txt`)

This context is added to **every research prompt**. Use it for:
- Research guidelines and priorities
- Sources to prioritize
- Information to look for
- Research methodology

**Example:**
```
Focus on recent news and developments from the past 3 months.
Prioritize information from:
- Official government sources
- Peer-reviewed research
- Established news outlets

Look for:
- Expert opinions and consensus
- Market expectations
- Historical precedents
- Quantitative data when available
```

### Forecast Context (`context/forecast_context.txt`)

This context is added to **every forecast prompt** (all question types). Use it for:
- Forecasting principles and guidelines
- Cognitive biases to avoid
- Calibration techniques
- Domain-specific knowledge

**Example:**
```
Forecasting Principles:
- Consider base rates and reference classes
- Account for regression to the mean
- Avoid overconfidence - use wide confidence intervals
- Think about alternative scenarios
- Consider the track record of similar forecasts

Common Biases to Avoid:
- Anchoring on initial estimates
- Confirmation bias
- Availability heuristic
- Overweighting recent events
```

## File Format

- **Comments**: Lines starting with `#` are ignored
- **Empty Lines**: Empty lines are ignored
- **Plain Text**: Everything else is included as-is

**Example:**
```
# This is a comment and will be ignored

This text will be included in the prompt.

# Another comment
More text here.
```

## Customization Examples

### Example 1: Domain-Specific Research

If you're focusing on AI/technology questions:

```
# Research Context for AI/Technology Questions

Priority sources:
- arXiv preprints for technical developments
- Major tech company announcements
- Regulatory filings (SEC, etc.)
- Expert interviews and podcasts

Key information to find:
- Technical feasibility assessments
- Market adoption rates
- Regulatory timelines
- Competitive landscape
```

### Example 2: Calibration Guidelines

For better calibrated forecasts:

```
# Forecast Calibration Guidelines

Before making a forecast:
1. Find 3-5 similar past events and their outcomes
2. Consider the base rate for this type of question
3. Think about what would need to happen for the outcome
4. Consider both optimistic and pessimistic scenarios

When assigning probabilities:
- 50% means "as likely as not" - use when truly uncertain
- 90% means "very likely" - use only when very confident
- 10% means "very unlikely" - use only when very confident it won't happen
- Avoid extreme probabilities (0%, 100%) unless absolutely certain
```

### Example 3: Question-Type Specific Context

You can add different context for different question types by checking the question type in your custom prompts, or by using conditional logic in the context files (though the current implementation adds the same context to all types).

## Testing Your Context

1. **Edit the context files** with your custom instructions
2. **Run a test forecast**:
   ```bash
   poetry run python main.py --mode test_questions
   ```
3. **Check the logs** to see if context is being included
4. **Review the reasoning** in the forecast outputs to see if the context is being used

## Best Practices

1. **Keep it concise**: Long context can dilute the main prompt
2. **Be specific**: Vague instructions are less effective
3. **Update regularly**: Refine your context based on what works
4. **Test changes**: Always test after modifying context
5. **Version control**: Commit context files to track changes

## Troubleshooting

### Context Not Appearing

- Check that files are in `context/` directory (relative to `main.py`)
- Verify file names are exactly: `research_context.txt` and `forecast_context.txt`
- Check file encoding (should be UTF-8)
- Look for errors in logs about loading context files

### Context Too Long

- If prompts are getting too long, shorten your context
- Consider splitting into multiple focused sections
- Remove redundant instructions

### Context Not Being Used

- Check that context is actually relevant to the task
- Make sure instructions are clear and actionable
- Review the actual prompts in logs to see how context is integrated

## Advanced Usage

### Conditional Context

You can modify the code to load different context based on conditions:

```python
def _load_context_file(self, file_path: str, question_type: str = None) -> str:
    # Load base context
    base_context = self._load_context_file(file_path)
    
    # Load type-specific context if exists
    if question_type:
        type_context = self._load_context_file(f"context/{question_type}_context.txt")
        return f"{base_context}\n\n{type_context}" if type_context else base_context
    
    return base_context
```

### Environment-Based Context

You could also load different context files based on environment variables:

```python
context_file = os.getenv("RESEARCH_CONTEXT_FILE", "context/research_context.txt")
```

## Files Created

- `context/research_context.txt` - Template for research context
- `context/forecast_context.txt` - Template for forecast context

Edit these files to customize your bot's behavior!
