# Category-Specific Context Implementation

## Overview

Based on the `api_json` structure, questions have a `projects.category` array with category objects containing:
- `id`: Category ID
- `name`: Category name (e.g., "Politics", "Geopolitics")
- `slug`: Category slug (e.g., "politics", "geopolitics")
- `description`: Category description
- `type`: Always "category"

We can use these category slugs to load category-specific context files.

## Option 1: File-Based with Naming Conventions (Recommended)

### Structure

```
context/
├── research.txt              # General research context (always loaded)
├── forecast.txt               # General forecast context (always loaded)
├── politics/
│   ├── research.txt           # Politics-specific research context
│   └── forecast.txt           # Politics-specific forecast context
├── geopolitics/
│   ├── research.txt           # Geopolitics-specific research context
│   └── forecast.txt           # Geopolitics-specific forecast context
├── economics/
│   ├── research.txt           # Economics-specific research context
│   └── forecast.txt           # Economics-specific forecast context
└── technology/
    ├── research.txt           # Technology-specific research context
    └── forecast.txt           # Technology-specific forecast context
```

### Implementation

**Step 1: Extract categories from question**

```python
def _get_question_categories(self, question: MetaculusQuestion) -> list[str]:
    """
    Extract category slugs from question's api_json.
    Returns list of category slugs (e.g., ['politics', 'geopolitics']).
    """
    try:
        categories = question.api_json.get("projects", {}).get("category", [])
        return [cat.get("slug") for cat in categories if cat.get("slug")]
    except Exception:
        return []
```

**Step 2: Load category-specific context**

```python
def _load_category_context(self, context_type: str, category_slug: str) -> str:
    """
    Load context for a specific category.
    context_type: 'research' or 'forecast'
    category_slug: e.g., 'politics', 'geopolitics'
    """
    file_path = f"context/{category_slug}/{context_type}.txt"
    return self._load_context_file(file_path)
```

**Step 3: Merge contexts**

```python
def _get_research_context(self, question: MetaculusQuestion) -> str:
    """
    Get merged research context: general + category-specific.
    """
    context_parts = []
    
    # Load general research context
    if self._research_context:
        context_parts.append(self._research_context)
    
    # Load category-specific contexts
    categories = self._get_question_categories(question)
    for category_slug in categories:
        category_context = self._load_category_context("research", category_slug)
        if category_context:
            context_parts.append(f"[{category_slug.title()} Context]\n{category_context}")
    
    return "\n\n".join(context_parts) if context_parts else ""
```

**Step 4: Update `run_research()`**

```python
async def run_research(self, question: MetaculusQuestion) -> str:
    async with self._concurrency_limiter:
        # ... existing code ...
        
        # Get merged research context
        research_context_section = ""
        merged_context = self._get_research_context(question)
        if merged_context:
            research_context_section = f"\n\nAdditional Research Guidelines:\n{merged_context}\n"
        
        # ... rest of code ...
```

**Step 5: Update forecast methods similarly**

```python
def _get_forecast_context(self, question: MetaculusQuestion) -> str:
    """
    Get merged forecast context: general + category-specific.
    """
    context_parts = []
    
    # Load general forecast context
    if self._forecast_context:
        context_parts.append(self._forecast_context)
    
    # Load category-specific contexts
    categories = self._get_question_categories(question)
    for category_slug in categories:
        category_context = self._load_category_context("forecast", category_slug)
        if category_context:
            context_parts.append(f"[{category_slug.title()} Context]\n{category_context}")
    
    return "\n\n".join(context_parts) if context_parts else ""
```

### Advantages

- ✅ **Simple and intuitive**: Easy to understand file structure
- ✅ **Easy to manage**: Each category has its own directory
- ✅ **Scalable**: Easy to add new categories (just create new directory)
- ✅ **No parsing needed**: Direct file reads
- ✅ **Version control friendly**: Clear diffs per category
- ✅ **Flexible**: Can have different contexts for research vs forecast per category

### Disadvantages

- ⚠️ **More files**: Many files to manage
- ⚠️ **Directory structure**: Need to create directories for each category

### Example Usage

**Question with categories: `["politics", "geopolitics"]`**

Loaded contexts:
1. `context/research.txt` (general)
2. `context/politics/research.txt` (politics-specific)
3. `context/geopolitics/research.txt` (geopolitics-specific)


### Implementation

**Implementation Priority:**

1. ✅ Extract categories from `api_json`
2. ✅ Create `_get_question_categories()` method
3. ✅ Create `_load_category_context()` method
4. ✅ Create `_get_research_context()` and `_get_forecast_context()` methods
5. ✅ Update `run_research()` to use merged context
6. ✅ Update all forecast methods to use merged context

---

## Implementation Details

### Category Detection

```python
def _get_question_categories(self, question: MetaculusQuestion) -> list[str]:
    """
    Extract category slugs from question's api_json.
    Returns list of category slugs (e.g., ['politics', 'geopolitics']).
    """
    try:
        categories = question.api_json.get("projects", {}).get("category", [])
        slugs = [cat.get("slug") for cat in categories if cat.get("slug")]
        logger.debug(f"Question categories: {slugs}")
        return slugs
    except Exception as e:
        logger.warning(f"Failed to extract categories: {e}")
        return []
```

### Context Merging Strategy

**Order of precedence:**
1. General context (always first)
2. Category-specific contexts (in order they appear in categories array)

**Format in prompt:**
```
Additional Research Guidelines:
[General context here]

[Politics Context]
[Politics-specific context here]

[Geopolitics Context]
[Geopolitics-specific context here]
```

### Error Handling

- If category directory doesn't exist: Skip it (graceful fallback)
- If category context file doesn't exist: Skip it (graceful fallback)
- If `api_json` is missing: Use only general context
- If categories array is empty: Use only general context

### File Naming Convention

- **Directory names**: Use category slugs exactly as they appear (lowercase, hyphenated)
- **File names**: `research.txt` and `forecast.txt` (consistent)
- **Case sensitivity**: Match category slugs exactly (usually lowercase)

---

## Example File Structure (Option 1)

```
context/
├── research.txt
├── forecast.txt
├── politics/
│   ├── research.txt
│   └── forecast.txt
├── geopolitics/
│   ├── research.txt
│   └── forecast.txt
└── economics/
    ├── research.txt
    └── forecast.txt
```

**Example content:**

`context/politics/research.txt`:
```
- Focus on election cycles and polling data
- Look for policy announcements and legislative changes
- Monitor political party dynamics
```

`context/geopolitics/research.txt`:
```
- Look for similar events and patterns in the years preceding World War I, World War II, and the dissolution of the Soviet Union
- Monitor international alliances and treaty changes
- Watch for military build-ups and border tensions
```

---

## Migration Path

1. **Phase 1**: Implement category detection and file loading
2. **Phase 2**: Create category directories for existing categories (politics, geopolitics)
3. **Phase 3**: Move category-specific content from general context to category files
4. **Phase 4**: Test with various questions to ensure correct category detection
5. **Phase 5**: Add more categories as needed

---

## Testing

Test cases:
1. Question with no categories → Only general context
2. Question with one category → General + one category context
3. Question with multiple categories → General + multiple category contexts
4. Category directory doesn't exist → Skip gracefully
5. Category file doesn't exist → Skip gracefully
6. `api_json` missing → Only general context
