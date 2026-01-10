# Repository Summary: Metaculus Bot Template

## Overview

This repository provides a template for creating automated forecasting bots to participate in the Metaculus AI Forecasting Tournament. The template includes two implementations: a framework-based approach using the `forecasting-tools` package (recommended) and a minimal-dependency version for custom implementations.

## Purpose

The repository serves as a starting point for developers to build AI-powered forecasting bots that can:
- Automatically discover and forecast on Metaculus questions
- Conduct research using various search providers
- Generate probabilistic forecasts using LLMs
- Submit predictions to the Metaculus platform
- Run automatically via GitHub Actions or locally

## Repository Structure

### Core Files

- **`main.py`**: Recommended template using the `forecasting-tools` framework
  - Object-oriented design with `SpringTemplateBot2026` class
  - Handles API interactions, research, and forecasting logic
  - Supports multiple question types with specialized handlers
  - Uses LLM-based output parsing via `structure_output`

- **`main_with_no_framework.py`**: Minimal-dependency implementation
  - Procedural approach without framework dependencies
  - Direct API calls to Metaculus
  - Manual parsing and CDF generation
  - Useful for understanding the underlying mechanics

- **`pyproject.toml`**: Poetry-based dependency management
  - Python 3.12+ requirement
  - Key dependencies: `forecasting-tools`, `openai`, `asknews`, `numpy`, `requests`
  - Development dependencies: `ipykernel`

- **`README.md`**: Documentation
  - Setup instructions
  - API key configuration
  - GitHub Actions automation guide
  - Example code for AskNews integration
  - Ideas for bot improvements

### Documentation

- **`docs/`**: Custom improvements and analysis

## Key Features

### Question Type Support

1. **Binary Questions**: Yes/No probability forecasts
2. **Numeric Questions**: Continuous distributions with percentile-based CDFs
3. **Multiple Choice Questions**: Probability distributions over discrete options
4. **Date Questions**: Temporal forecasts with date-based distributions
5. **Conditional Questions**: Complex questions with parent/child relationships

### Research Capabilities

The bot can conduct research using multiple providers:
- **AskNews**: News search and deep research endpoints
- **Perplexity**: Online LLM with search capabilities
- **Exa**: Smart search with highlights
- **SmartSearcher**: Framework-provided intelligent search
- **No Research**: Option to skip research entirely

### Forecasting Workflow

1. **Question Discovery**: Fetches open questions from tournaments
2. **Research Phase**: Conducts research on each question (configurable)
3. **Forecast Generation**: 
   - Runs multiple forecast attempts per research report
   - Uses LLM prompts tailored to question type
   - Aggregates predictions (median for numeric, average for multiple choice)
4. **Submission**: Posts forecasts and reasoning to Metaculus

### LLM Integration

- Supports multiple LLM providers via LiteLLM/OpenRouter
- Configurable models for different tasks (default, researcher, parser, summarizer)
- Temperature and timeout controls
- Rate limiting support

## Architecture

### Framework-Based (`main.py`)

- **Class Structure**: `SpringTemplateBot2026` extends `ForecastBot`
- **Key Methods**:
  - `run_research()`: Conducts research on questions
  - `_run_forecast_on_binary()`: Handles binary questions
  - `_run_forecast_on_numeric()`: Handles numeric questions
  - `_run_forecast_on_multiple_choice()`: Handles multiple choice questions
  - `_run_forecast_on_date()`: Handles date questions
  - `_run_forecast_on_conditional()`: Handles conditional questions

- **Concurrency**: Uses `asyncio.Semaphore` for rate limiting
- **Output Parsing**: Uses `structure_output` with LLM-based validation

### Minimal Framework (`main_with_no_framework.py`)

- **Procedural Design**: Functions for each question type
- **Manual Parsing**: Regex-based extraction of probabilities and percentiles
- **CDF Generation**: Custom `NumericDistribution` class with validation
- **Direct API Calls**: Uses `requests` library directly

## Configuration

### Environment Variables

Required:
- `METACULUS_TOKEN`: Authentication token for Metaculus API

Optional (for research):
- `OPENROUTER_API_KEY` or `OPENAI_API_KEY`: LLM provider keys
- `ASKNEWS_CLIENT_ID` / `ASKNEWS_SECRET`: AskNews credentials
- `PERPLEXITY_API_KEY`: Perplexity API key
- `EXA_API_KEY`: Exa search API key

### Bot Parameters

- `research_reports_per_question`: Number of research reports to generate
- `predictions_per_research_report`: Forecasts per research report
- `publish_reports_to_metaculus`: Whether to submit predictions
- `skip_previously_forecasted_questions`: Skip questions already forecasted
- `use_research_summary_to_forecast`: Whether to summarize research before forecasting

## Usage Modes

1. **Tournament Mode**: Forecasts on current AI Benchmark tournament and MiniBench
2. **Metaculus Cup Mode**: Forecasts on regularly open questions
3. **Test Questions Mode**: Tests on example questions

## Deployment

### GitHub Actions

- Automated runs to forecast on tournaments

### Local Execution

```bash
poetry install
poetry run python main.py --mode test_questions
```

## Technical Highlights

### CDF Generation (`main_with_no_framework.py`)

- Converts percentile distributions to 201-point CDFs
- Handles open/closed bounds
- Supports log-scaled questions
- Validates distribution properties (spacing, concentration, bounds)

### Prompt Engineering

- Structured prompts for each question type
- Emphasizes status quo bias
- Includes resolution criteria and fine print
- Encourages wide confidence intervals for numeric questions

### Error Handling

- Exception handling in async operations
- Validation of forecast outputs
- Graceful degradation when research fails

## Dependencies

### Core
- `forecasting-tools` (^0.2.80): Framework for Metaculus bot development
- `openai` (^2.0.0): OpenAI API client
- `numpy` (^2.3.0): Numerical operations
- `requests` (^2.32.3): HTTP requests

### Research Providers
- `asknews` (^0.13.0): AskNews SDK

### Utilities
- `python-decouple` (^3.8): Configuration management
- `python-dotenv` (^1.0.1): Environment variable loading

## Best Practices Implemented

1. **Multiple Forecasts**: Runs multiple predictions and aggregates (median/average)
2. **Research Integration**: Incorporates external research into forecasts
3. **Status Quo Bias**: Prompts emphasize considering current state
4. **Wide Intervals**: Encourages conservative confidence intervals
5. **Validation**: Validates outputs before submission
6. **Rate Limiting**: Prevents API abuse

## Extension Ideas (from README)

- Fine-tuned LLMs on Metaculus data
- Dataset exploration and analysis tools
- Question decomposition
- Meta-forecast aggregation
- Base rate research
- Monte Carlo simulations
- Personality/LLM diversity
- Worldbuilding scenarios
- Consistency forecasting
- Calibration adjustments
- Evidence point systems
- Search provider benchmarking

## Version Information

- **Template Version**: Spring 2026
- **Python Requirement**: ^3.12 (3.14 fails, but 3.12 works)
- **Last Major Updates**:
  - Additional prompting for numeric questions (percentile ordering)
  - Support for conditional and date questions
  - LLM-based output parsing
  - Nominal bounds support

## Notes

- The framework-based version (`main.py`) is recommended for most users
- The minimal version (`main_with_no_framework.py`) is useful for learning and customization
- Both versions have been tested but may have occasional bugs (track record: ~1 bug per season affecting 1-2% of questions)
- The bot is designed for the AI Forecasting Tournament but can also forecast on main Metaculus questions
