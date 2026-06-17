# Token Efficiency - Project Patterns

This file contains project organization patterns, task management, background process management, MANIFEST system, and efficient file operations. See [SKILL.md](SKILL.md) for core rules.

---

## Analysis File Organization

### Problem
Large files (notebooks, logs, data dumps) consume excessive tokens when loaded for context.

### Solution: Separate Analysis Files

Create `analysis_files/` directory with individual markdown files for each analysis component:

```
project/
├── analysis_files/
│   ├── MANIFEST.md
│   ├── Method.md
│   └── figures/
│       ├── 01_figure_name.md
│       ├── 02_figure_name.md
│       └── ...
├── notebooks/ (keep for code execution)
└── figures/ (actual PNG/PDF files)
```

**Token Savings**:
- Before: ~1,135,000 tokens (full notebooks)
- After: ~22,000 tokens (analysis files)
- **Savings: 98% reduction**

### What to Include in Analysis Files

Each analysis file should contain **only publication-ready text**:
- Description (formatted as legend)
- Analysis framework and interpretation
- Statistical methods and results
- Mechanistic explanations
- Context from other metrics

**DO NOT include**:
- TODOs or placeholders
- Draft notes or scratch work
- Code or computational details
- Incomplete sections

---

## Task Management Patterns

### TodoWrite for Sequential File Processing

When processing multiple files sequentially:

**Efficient pattern**:
```
1. Create todos for all files at start
2. Mark ONLY ONE as in_progress at a time
3. Complete the file
4. Mark as completed IMMEDIATELY (don't batch)
5. Mark next file as in_progress
6. Repeat
```

**Why this matters**:
- **Provides clear progress tracking** for multi-step tasks
- **Prevents skipping files** or losing track of position
- **Shows user real-time progress** as files are completed
- **Maintains focus** on one file at a time
- **Enables context resumption** if session is interrupted

**Token efficiency benefit**:
- Minimal token cost (~100-200 tokens per update)
- Massive value in maintaining task context and preventing rework
- User can see progress without asking "what's done?"

**Anti-pattern**:
- Don't mark multiple todos as completed at once
- Don't skip updating todos between files
- Don't have multiple in_progress tasks simultaneously

---

## Managing Long-Running Background Processes

### Best Practices for Background Tasks

When running scripts that take time, properly manage background processes:

**1. Run in background** with appropriate tools

**2. Document the process** in status files:
```markdown
## Background Processes
- Script: comprehensive_search.py
- Process ID: Available via process tools
- Status: Running (~6% complete)
- How to check: Use process output tool
```

**3. Kill cleanly** before session end

**4. Design scripts to be resumable**:
- Check for existing output files (skip if present)
- Load existing results and append new ones
- Save progress incrementally (not just at end)
- Track completion status in structured format

### Avoiding Unnecessary Polling

**Problem**: Repeatedly checking background process output wastes tokens when results aren't ready yet.

**Token-efficient pattern**:
1. Start process in background, capture ID
2. Inform user once: "Running in background, ID: XXXXX"
3. Don't repeatedly check output unless user asks
4. User can interrupt and say "check later" - acknowledge and stop polling

**Token savings**: Avoiding 15-20 repeated checks (200 tokens each) = ~3000-4000 tokens saved per long-running process.

---

## Repository Organization for Long Projects

### Problem
Data enrichment and analysis projects generate many intermediate files that clutter the root directory.

### Solution: Organize Early and Often

**Create dedicated subfolders at project start:**
```bash
mkdir -p scripts/ logs/ tables/
```

**Organization strategy:**
- `scripts/` - All analysis and processing scripts
- `logs/` - All execution logs from script runs
- `tables/` - Intermediate results, old versions, and archived data
- Root directory - Only main working dataset and current outputs

**Benefits:**
- Reduces cognitive load when scanning directory
- Makes git status cleaner and more readable
- Easier to exclude intermediate files from version control
- Faster file navigation with autocomplete
- Professional project structure for collaboration

**Token efficiency impact:**
- Cleaner `ls` outputs (fewer lines to process)
- Easier to target specific directories with Glob
- Reduced cognitive overhead when navigating
- Faster file location with autocomplete

---

## Project Navigation Patterns

### MANIFEST System for Large Projects

**Problem**: Large projects require reading many files for orientation, consuming 15,000-23,000 tokens per session startup.

**Solution**: Create MANIFEST.md files (lightweight project indexes) that provide complete context in 2,000-2,500 tokens.

#### Token Impact

- **Without MANIFESTs**: 15,000-23,000 tokens for project orientation
- **With MANIFESTs**: 2,000-2,500 tokens for complete context
- **Savings**: 85-90% reduction in session startup cost

#### Implementation

1. **Create MANIFEST.md in each major directory** (root, data/, figures/, scripts/, documentation/)
2. **Include essential sections**:
   - Quick Reference (entry points, key outputs, dependencies)
   - File Inventory (with descriptions, sizes, purposes)
   - Workflow Dependencies (how files relate)
   - Notes for Resuming Work (status, next steps, issues)
   - Metadata (tags, environment)

3. **Target token counts**:
   - Root MANIFEST: 1,000-2,000 tokens
   - Subdirectory MANIFEST: 500-1,000 tokens

#### Session Workflow

**Start session**:
```bash
# Read MANIFESTs for context (2,000 tokens vs 15,000)
cat MANIFEST.md                # Project overview
cat figures/MANIFEST.md        # If working on figures
```

#### When to Use

- Projects with 3+ notebooks
- Multiple data files and directories
- Many generated figures (10+)
- Long-term research projects
- Team collaboration

#### Benefits

- **For Claude**: 85-90% fewer tokens for session startup
- **For Users**: Quick work resumption without reading files
- **For Teams**: Faster onboarding, shared understanding

---

## Efficient File Operations

### Moving Multiple Files Safely

When moving files where some might not exist, use loops with file existence checks:

**Inefficient (fails on missing files):**
```bash
mv file1.py file2.py file3.py destination/  # Fails if any missing
```

**Efficient (handles missing files gracefully):**
```bash
for f in file1.py file2.py file3.py; do
  if [ -f "$f" ]; then
    mv "$f" destination/ && echo "Moved $f"
  fi
done
```

### File Reorganization

When reorganizing project files into directories:

**Efficient approach:**
```bash
# Create all directories first
mkdir -p figures data tests notebooks docs archives

# Move files by pattern (fast, no file reads needed)
mv fig*.png figures/
mv *.json *.tsv *.csv data/
mv test_*.py tests/
mv *.ipynb notebooks/
mv *.md docs/
```

**Token savings**: For reorganizing 50+ files, this approach uses ~1K tokens vs. 5-10K tokens if reading files to determine categorization.

### Validating Against Large Reference Files

**Pattern**: Need to validate data against large reference file

**Inefficient** (reading reference repeatedly):
```python
# Costs ~500 tokens per read
reference_data = parse_reference('large_file.dat')
# Then validate each config...
```

**Efficient** (one-time extraction, reuse):
```python
# Read once, extract needed data, save to small file
import re
with open('large_reference.dat') as f:
    content = f.read()
items = set(re.findall(r'pattern', content))

# Save for reuse (~50 tokens vs 500)
with open('extracted_items.txt', 'w') as f:
    f.write('\n'.join(sorted(items)))

# Future validations use the small file
items = set(open('extracted_items.txt').read().splitlines())
```

**Token savings**: 90% (500 -> 50 tokens per validation)

### Safe Directory Removal

**Always use `rmdir` instead of `rm -rf` when removing directories you expect to be empty:**

```bash
# Good - safe, fails if directory not empty
rmdir old-folder/

# Avoid - dangerous, silently removes everything
rm -rf old-folder/
```

**Benefits:**
- Prevents accidental deletion of files
- Alerts you if directory unexpectedly contains files
- Forces you to handle hidden files

**When to use rm -rf:**
- Only when explicitly removing directories with content
- With user confirmation first
- Never as default for cleanup
