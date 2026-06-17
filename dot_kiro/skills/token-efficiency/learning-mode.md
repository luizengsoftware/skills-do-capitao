# Token Efficiency - Learning Mode Strategies

This file contains detailed strategies for learning mode: strategic file selection, targeted pattern learning, and broad repository exploration. See [SKILL.md](SKILL.md) for core rules.

---

## Strategic File Selection for Learning Mode

When entering learning mode, **first determine if this is broad exploration or targeted learning**, then apply the appropriate strategy.

### Learning Mode Types

**Type 1: Broad Exploration** - "Help me understand this codebase", "How is this organized?"
-> Use repository-based strategies below (identify type, read key files)

**Type 2: Targeted Pattern Learning** - "How do I implement X?", "Show me examples of Y"
-> Use targeted concept search (see Targeted Pattern Learning section below)

---

## General Exploration vs Targeted Learning

**When user says -> Use this approach:**

| User Request | Approach | Strategy |
|--------------|----------|----------|
| "Help me understand this codebase" | **General Exploration** | Identify repo type -> Read key files |
| "How is this project organized?" | **General Exploration** | Read docs -> Entry points -> Architecture |
| "Show me how to implement X" | **Targeted Learning** | Search for X -> Read examples -> Extract pattern |
| "How does feature Y work?" | **Targeted Learning** | Grep for Y -> Find best examples -> Explain |
| "What patterns are used here?" | **General Exploration** | Read core files -> Identify patterns |
| "How do I use API method Z?" | **Targeted Learning** | Search for Z usage -> Show examples |

---

## Targeted Pattern Learning

When user asks about a **specific technique or pattern**, use this focused approach instead of broad exploration.

### Targeted Learning Workflow

**STEP 1: Identify the Specific Concept**

Extract the key concept from user's question:
```
User: "How do variable number of outputs work in Galaxy wrappers?"
-> Concept: "variable number of outputs" OR "dynamic outputs"
-> Context: "Galaxy tool wrappers"
-> File types: ".xml" (Galaxy tool wrappers)
```

**STEP 2: Search for Examples**

Use targeted searches to find relevant code:

```bash
# For error handling patterns
grep -r "try:\|except\|raise" --include="*.py" -l | xargs grep -l "class.*Error"
```

**STEP 3: Rank and Select Examples**

**Selection criteria (in priority order):**

1. **Documentation/Comments** - Files with good comments explaining the pattern
2. **Simplicity** - Simpler examples are better for learning
3. **Recency** - Recent code shows current best practices
4. **Multiple variations** - Show different approaches if they exist

**STEP 4: Read Examples Fully**

Read 2-3 selected examples **completely** to understand the pattern.

**STEP 5: Extract and Explain the Pattern**

After reading examples, explain:
1. **The core pattern** - How it works conceptually
2. **Required elements** - What's needed to implement it
3. **Common variations** - Different ways to use it
4. **Common pitfalls** - What to avoid
5. **Best practices** - Recommended approach

---

### When to Use Targeted Learning

**Use targeted learning when user:**
- Asks "how do I..." about specific feature
- Requests "show me examples of X"
- Wants to learn specific pattern/technique
- Has focused technical question
- References specific concept/API/feature

**Don't use for:**
- "Understand this codebase" (use broad exploration)
- "What does this project do?" (use documentation reading)
- "Debug this error" (use debugging mode, not learning mode)

---

### Key Principles for Targeted Learning

1. **Search first, read second**
   - Use grep to find relevant examples
   - Rank by quality/simplicity/recency
   - Then read selected examples fully

2. **Read 2-3 examples, not 20**
   - Simple example (minimal working code)
   - Standard example (common usage)
   - Complex example (advanced features) - optional

3. **Extract the pattern**
   - Don't just show code, explain the pattern
   - Highlight key elements and structure
   - Show variations and alternatives

4. **Provide context**
   - Where this pattern is used
   - When to use it vs alternatives
   - Common pitfalls and best practices

5. **Confirm understanding**
   - Ask if user needs specific variation
   - Offer to show related patterns
   - Check if explanation answered their question

---

## Broad Repository Exploration

When entering broad exploration mode, **first identify the repository context**, then apply the appropriate exploration strategy.

### STEP 1: Identify Repository Type

**Repository type indicators:**

1. **Tool Library / Recipe Collection** (bioconda, tool collections)
   - Multiple independent directories at same level
   - Each subdirectory is self-contained

2. **Monolithic Application** (single integrated codebase)
   - One main entry point
   - Hierarchical module structure
   - Shared dependencies and utilities

3. **Framework / SDK** (extensible system)
   - Core framework + plugins/extensions
   - Base classes and interfaces

4. **Example / Template Repository**
   - Multiple example implementations
   - Each directory shows different pattern

---

### STEP 2: Apply Context-Specific Strategy

#### Strategy A: Tool Library / Recipe Collection

**Files to read (in order):**
1. `CONTRIBUTING.md` or submission guidelines -> Learn required structure
2. Recent tool/recipe -> Current best practices
3. Well-established tool/recipe -> Proven patterns
4. Template or example -> Base structure

#### Strategy B: Monolithic Application

**Files to read (in order):**
1. `README.md` -> Overview and architecture
2. Entry point (`main.py`, `run_all.py`) -> Execution flow
3. Core orchestrator/manager -> Main logic
4. Most-imported utility module -> Common patterns
5. One domain-specific module -> Implementation details

#### Strategy C: Framework / SDK

**Files to read (in order):**
1. Architecture documentation -> Design philosophy
2. Base/core classes -> Fundamental abstractions
3. Simple plugin/extension -> How to extend
4. Complex plugin/extension -> Advanced patterns

#### Strategy D: Example / Template Repository

**Files to read (in order):**
1. `examples/README.md` -> Overview of examples
2. Basic example -> Minimal working pattern
3. Advanced example -> Full-featured pattern
4. Compare differences -> Learn progression

---

### STEP 3: Execution Strategy Template

**For ANY repository type, use this workflow:**

```bash
# PHASE 1: Context Discovery (always token-efficient)
ls -la  # Repository structure
cat README.md  # Overview
ls -la .github/ docs/ | head -20  # Find documentation
cat CONTRIBUTING.md 2>/dev/null | head -50  # Submission guidelines

# PHASE 2: Identify Type (ask user if unclear)

# PHASE 3: Strategic Reading (based on type)
# Read 2-5 key files fully
# Grep for patterns across remaining files

# PHASE 4: Summarize and Confirm
```

---

### File Selection Priorities (General Rules)

**Priority 1: Documentation**
```bash
README.md, CONTRIBUTING.md, docs/architecture.md
# These explain intent, not just implementation
```

**Priority 2: Entry Points**
```bash
# Monolithic: main.py, app.py, run.py, __main__.py
# Library: Most recent example in collection
```

**Priority 3: Core Components**
```bash
# Most imported modules
grep -r "import" | cut -d: -f2 | sort | uniq -c | sort -rn

# "Manager", "Controller", "Orchestrator", "Core", "Base"
find . -name "*manager*" -o -name "*core*" -o -name "*base*"
```

**Priority 4: Representative Examples**
```bash
# Recent files (current best practices)
ls -lt directory/ | head -5

# Medium complexity (not too simple, not too complex)
wc -l **/*.py | sort -n | awk 'NR > 10 && NR < 20'
```

**Priority 5: Active Development Areas**
```bash
# Git history (if available)
git log --name-only --since="1 month ago" --pretty=format: | sort | uniq -c | sort -rn
```

---

### Key Principle for Learning Mode

**Balance understanding with efficiency:**
- Read 2-5 **strategic** files fully (based on context)
- Use grep/head/tail for **pattern discovery** across many files
- **Ask user** which aspect to focus on after initial exploration
- **Summarize** findings before reading more

**Don't:**
- Read 20+ files sequentially without strategy
- Read files without understanding their role
- Ignore repository context and documentation
