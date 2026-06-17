# Token Efficiency - Examples and Cost Calculations

This file contains extensive token savings examples with before/after comparisons and cost calculations. See [SKILL.md](SKILL.md) for core rules.

---

## Token Savings Examples for Bash Commands

### Example 1: Update 10 config files

Wasteful approach:
```bash
Read: config1.yaml  # 5K tokens
Edit: config1.yaml
Read: config2.yaml  # 5K tokens
Edit: config2.yaml
# ... repeat 10 times = 50K tokens
```

Efficient approach:
```bash
Bash: for f in config*.yaml; do sed -i '' 's/old/new/g' "$f"; done
# Token cost: ~100 tokens for command, 0 for file content
```

**Savings: 49,900 tokens (99.8%)**

### Example 2: Copy configuration

Wasteful approach:
```bash
Read: template_config.yaml  # 10K tokens
Write: project_config.yaml  # 10K tokens
# Total: 20K tokens
```

Efficient approach:
```bash
Bash: cp template_config.yaml project_config.yaml
# Token cost: ~50 tokens
```

**Savings: 19,950 tokens (99.75%)**

### Example 3: Append log entry

Wasteful approach:
```bash
Read: application.log  # 50K tokens (large file)
Write: application.log  # 50K tokens
# Total: 100K tokens
```

Efficient approach:
```bash
Bash: echo "[$(date)] Log entry" >> application.log
# Token cost: ~50 tokens
```

**Savings: 99,950 tokens (99.95%)**

---

## High-Level Token Savings Examples

### Example 1: Status Check
**Scenario:** User asks "What's the status of my application?"

**Wasteful approach (50K tokens):**
```bash
Read: /var/log/app.log  # 40K tokens
Bash: systemctl status myapp  # 10K tokens
```

**Efficient approach (3K tokens):**
```bash
Bash: systemctl status myapp --no-pager | head -20  # 1K tokens
Bash: tail -50 /var/log/app.log  # 2K tokens
```
**Savings: 94%**

---

### Example 2: Debugging Errors
**Scenario:** User says "My script is failing, help debug"

**Wasteful approach (200K tokens):**
```bash
Read: debug.log  # 150K tokens
Read: script.py  # 30K tokens
Read: config.json  # 20K tokens
```

**Efficient approach (8K tokens):**
```bash
Bash: tail -100 debug.log  # 3K tokens
Bash: grep -i "error\|traceback" debug.log | tail -50  # 2K tokens
Grep: "def main" script.py  # 1K tokens
Read: script.py (offset: 120, limit: 50)  # 2K tokens (just the failing function)
```
**Savings: 96%**

---

### Example 3: Code Review
**Scenario:** User asks "Review this codebase"

**Wasteful approach (500K tokens):**
```bash
Read: file1.py
Read: file2.py
Read: file3.py
Read: file4.py
# ... reads 20+ files
```

**Efficient approach (20K tokens):**
```bash
Bash: find . -name "*.py" | head -30  # 1K
Bash: cloc .  # Lines of code summary - 1K
Bash: grep -r "^class " --include="*.py" | head -20  # 2K
Bash: grep -r "^def " --include="*.py" | wc -l  # 1K
Read: main.py (limit: 100)  # 3K
Read: README.md  # 5K
Grep: "TODO\|FIXME\|XXX" -r .  # 2K
# Then ask user what specific areas to review
```
**Savings: 96%**

---

## Task Tool Token Savings

**Inefficient approach (many tool calls, large context)**:
```python
# Direct grep through many files
Grep(pattern="some_pattern", path=".", output_mode="content")
# Followed by multiple Read calls to understand context
Read("file1.py")
Read("file2.py")
# Followed by more Grep calls for related patterns
Grep(pattern="related_pattern", path=".", output_mode="content")
# Results in dozens of tool calls and accumulating context
```

**Efficient approach (single consolidated response)**:
```python
# Use Task tool with Explore subagent
Task(
    subagent_type="Explore",
    description="Research how Galaxy API works",
    prompt="""Explore the codebase to understand how Galaxy API calls are made.
    I need to know:
    - Which files contain API call patterns
    - How authentication is handled
    - Common error handling patterns
    Return a summary with file locations and key patterns."""
)
```

**Token savings**:
- Task tool: ~5-10K tokens for consolidated response
- Direct exploration: ~30-50K tokens (many tool calls + context accumulation)
- **Savings: 70-80%** for exploratory searches
