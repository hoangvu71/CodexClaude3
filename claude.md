# AgenticWriting: Autonomous Book Creation Pipeline

**Purpose:** Complete autonomous pipeline for creating fiction books (series or standalone) from concept to finished chapters.

**This document:** Instructions for Claude to orchestrate the entire pipeline without prior context. Follow sequentially.

---

## 📊 CURRENT PIPELINE STATUS

**Project:** Kissing the Cryptographer Was Never Part of the Authentication Protocol
**Type:** Standalone Book (30 chapters: Act 1: 10ch, Act 2: 12ch, Act 3: 8ch)
**Branch:** claude/continue-pipeline-work-01NBf5qJSLuu2TJfKedJHrKX
**Last Updated:** 2025-11-16

### Progress Tracker

- ✅ **Phase 1: Foundation Building** - COMPLETE
  - Created: logline.md, book-spine.md
  - Location: `./Kissing the Cryptographer Was Never Part of the Authentication Protocol/`

- 🔄 **Phase 2: Outline Creation** - IN PROGRESS
  - Status: About to invoke outline-architect
  - Expected: Bible + 30 chapter outlines

- ⏳ **Phase 3: QA Validation** - PENDING

- ⏳ **Phase 4: Outline Critique Loop** - PENDING

- ⏳ **Phase 5: Context Guide Creation** - PENDING

- ⏳ **Phase 6: Chapter Writing + QA** - PENDING
  - Expected: 10 pairs (30 chapters ÷ 3)

- ⏳ **Phase 7: Beta Reading** - PENDING

- ⏳ **Phase 8: Beta Reader Fixes** - PENDING

---

## Quick Start: What This Pipeline Does

1. **Foundation** → Creates story concept and structure
2. **Outline** → Creates comprehensive chapter-by-chapter outlines
3. **QA Validation** → Validates outline-architect completed properly
4. **Outline Critique Loop** → Critiques outlines, fixes issues, repeats until clean
5. **Context Guide** → Maps chapter relationships for intelligent context loading
6. **Writing + QA (Paired)** → Writes 3 chapters, validates, repeats (3000+ words each)
7. **Beta Reading** → Simulates human reader, identifies issues
8. **Fixing** → Systematically fixes all beta-reader issues

---

## Pipeline Phases

### Phase 1: Foundation Building

**Agent:** `foundation-builder`

**When to invoke:** User provides initial concept (genre/trope/premise)

**CRITICAL - Research Inspiration Sources:**

When the user mentions a **specific work** as inspiration (book title, anime, manga, series, etc.), you MUST instruct the foundation-builder to research it via WebSearch first.

**Why this matters:**
- Captures nuanced details from the actual source material
- Understands specific tone, character dynamics, and appeal factors
- Avoids relying on your secondhand knowledge which may miss key elements

**Bad invocation (relying on your interpretation):**
```
prompt: "Create a series inspired by [Work Title] - an isekai series where..."
```

**Good invocation (instructs agent to research):**
```
prompt: "IMPORTANT: First, use WebSearch to research '[Exact Work Title]' -
understand its plot, themes, protagonist, companions, tone, and what makes it
appealing.

Then create a NEW [series/standalone] inspired by these elements, not a retelling.
Capture the key elements that make the original work successful while creating
a completely original story.

[User's additional requirements if any]"
```

**Invocation:**
```
Use Task tool with subagent_type="foundation-builder":
prompt: "[See above - include WebSearch instruction if inspiration work mentioned]"
description: "Create foundation files"
subagent_type: "foundation-builder"
```

**Expected outputs:**
- `./logline.md`
- `./series-spine.md` OR `./book-spine.md`

**Verification:**
```bash
ls ./logline.md ./series-spine.md 2>/dev/null || ls ./logline.md ./book-spine.md 2>/dev/null
```

**Next:** Proceed to Phase 2

---

### Phase 2: Outline Creation

**Agent:** `outline-architect`

**When to invoke:** After foundation files exist

**Prerequisites check:**
```bash
# Check spine type
ls ./series-spine.md 2>/dev/null && echo "SERIES"
ls ./book-spine.md 2>/dev/null && echo "STANDALONE"

# Check for existing work (resume detection)
ls -d ./Outlines/Book\ *\ Outlines 2>/dev/null || ls -d ./Outlines/Standalone 2>/dev/null
```

**Invocation:**
```
Use Task tool with subagent_type="outline-architect":
prompt: "Create complete outlines. Read all existing files to resume if work already started."
description: "Create chapter outlines"
subagent_type: "outline-architect"
```

**Expected outputs:**
- Series: `./Outlines/Book [N] Outlines/` (Bible + chapter outlines per book)
- Standalone: `./Outlines/Standalone/` (Bible + chapter outlines)

**Verification:**
```bash
# Count books (series) or check standalone exists
ls -d ./Outlines/Book\ *\ Outlines/ 2>/dev/null | wc -l
ls -d ./Outlines/Standalone/ 2>/dev/null

# Count chapter outlines
find ./Outlines -name "Outline Chapter*.md" | wc -l
```

**Next:** Phase 3

---

### Phase 3: QA Validation of Outlines

**Agent:** `qa-validator`

**When to invoke:** After outline-architect completes

**Invocation:**
```
Use Task tool with subagent_type="qa-validator":
prompt: "Validate outline-architect output."
description: "Validate outline-architect"
subagent_type: "qa-validator"
```

**Expected outputs:**
- `./Outlines/QA-Reports/qa-outline-architect-[YYYY-MM-DD].md`

**Verification:**
```bash
ls ./Outlines/QA-Reports/qa-outline-architect-*.md
# Check completion status in report
```

**Next:** Phase 4

---

### Phase 4: Outline Critique Loop

**Agent:** `outline-critique` (invoked multiple times until clean)

**When to invoke:** After outline-architect validated

**CRITICAL - Loop until no issues:**

```
LOOP:
  1. Invoke outline-critique
  2. Check report: ./Outlines/Critique/outline-critique-report-Fixed-[XX]%-[date].md
  3. If issues found (XX% < 100):
     - Issues were fixed
     - Re-invoke outline-critique (repeat from step 1)
  4. If no issues found (XX% = 100 OR report shows "No issues"):
     - Exit loop
     - Proceed to Phase 5
```

**Invocation (each iteration):**
```
Use Task tool with subagent_type="outline-critique":
prompt: "Perform comprehensive critique of all outlines. Find and fix all issues."
description: "Critique and fix outlines"
subagent_type: "outline-critique"
```

**Expected outputs (per iteration):**
- `./Outlines/Critique/outline-critique-report-Fixed-[XX]%-[YYYY-MM-DD].md`
- Edits to outline files (in place)

**Verification after each iteration:**
```bash
ls ./Outlines/Critique/outline-critique-report-*.md | tail -1
grep "Fixed:" ./Outlines/Critique/outline-critique-report-*.md | tail -1
# Check if 100% or no issues found
```

**Exit condition:** Report shows no issues OR 100% fixed

**Next:** Phase 5

---

### Phase 5: Context Guide Creation

**Agent:** `context-guide`

**When to invoke:** After outlines are clean (Phase 4 complete)

**Invocation:**
```
Use Task tool with subagent_type="context-guide":
prompt: "Create context guides for all books."
description: "Generate context guides"
subagent_type: "context-guide"
```

**Expected outputs:**
- Series: `./Outlines/Book [N] Outlines/context-guide.md` (one per book)
- Standalone: `./Outlines/Standalone/context-guide.md`

**Verification:**
```bash
find ./Outlines -name "context-guide.md" | wc -l
```

**Next:** Phase 6

---

### Phase 6: Chapter Writing + QA Validation (Paired)

**Agents:** `chapter-writer` + `qa-validator` (paired invocations)

**When to invoke:** After context-guide created

**CRITICAL - Paired execution pattern:**

```
LOOP for each batch of 3 chapters:
  1. Invoke chapter-writer for chapters [N] to [N+2]
  2. Verify chapters created
  3. Invoke qa-validator for those chapters
  4. Verify QA report shows completion
  5. Repeat until all chapters complete
```

**How to determine invocations needed:**

```bash
# Series: Count chapters per book
ls ./Outlines/Book\ 1\ Outlines/Outline\ Chapter*.md | wc -l
# Result: 30 chapters → Need 10 pairs (30 ÷ 3)

# Standalone: Count chapters
ls ./Outlines/Standalone/Outline\ Chapter*.md | wc -l
# Result: 25 chapters → Need 9 pairs (round up: 25 ÷ 3 = 8.33)
```

**Paired invocation pattern (example for Book 1 chapters 1-3):**

**Step 1: Write chapters**
```
Use Task tool with subagent_type="chapter-writer":
prompt: "Write chapters 1-3 for Book 1."
description: "Write Book 1 chapters 1-3"
subagent_type: "chapter-writer"
```

**Step 2: Validate chapters**
```
Use Task tool with subagent_type="qa-validator":
prompt: "Validate chapter-writer output for Book 1 chapters 1-3."
description: "Validate Book 1 chapters 1-3"
subagent_type: "qa-validator"
```

**Expected outputs per pair:**

**Chapter files (Series - Book 1, chapters 1-3):**
- `./Chapters/Book 1/Book 1 Chapter 01.md`
- `./Chapters/Book 1/Book 1 Chapter 02.md`
- `./Chapters/Book 1/Book 1 Chapter 03.md`

**QA report:**
- `./Outlines/QA-Reports/qa-chapter-writer-book-1-ch-1-3-[YYYY-MM-DD].md`

**Verification after each pair:**
```bash
# Check files created
ls ./Chapters/Book\ 1/Book\ 1\ Chapter\ 0[1-3].md

# Check word count (should be 3000+ per chapter)
wc -w ./Chapters/Book\ 1/Book\ 1\ Chapter\ 01.md

# Check QA report exists
ls ./Outlines/QA-Reports/qa-chapter-writer-*.md | tail -1
```

**Resume detection:**

Before each pair, check what chapters already exist:
```bash
# Series
ls ./Chapters/Book\ 1/*.md 2>/dev/null | sort -V | tail -1
# Returns: Book 1 Chapter 06.md → Next pair: chapters 7-9

# Standalone
ls ./Chapters/Standalone/*.md 2>/dev/null | sort -V | tail -1
# Returns: Standalone Chapter 09.md → Next pair: chapters 10-12
```

**Complete pairing sequence (Book 1 with 30 chapters):**

```
Pair 1:  Write chapters 1-3  → QA validate chapters 1-3
Pair 2:  Write chapters 4-6  → QA validate chapters 4-6
Pair 3:  Write chapters 7-9  → QA validate chapters 7-9
...
Pair 10: Write chapters 28-30 → QA validate chapters 28-30
```

**For Series:** When starting Book 2+, use same paired pattern:
```
Pair 1: Write chapters 1-3 for Book 2 → QA validate
Pair 2: Write chapters 4-6 for Book 2 → QA validate
...
```
(chapter-writer automatically reads previous book's last chapter for continuity)

**Next:** Phase 7 (after ALL chapters for a book are written and validated)

---

### Phase 7: Beta Reading

**Agent:** `beta-reader`

**When to invoke:** After all chapters written and validated for a book (Phase 6 complete)

**Invocation (Series):**
```
Use Task tool with subagent_type="beta-reader":
prompt: "Beta read Book 1."
description: "Beta read Book 1"
subagent_type: "beta-reader"
```

**Invocation (Standalone):**
```
Use Task tool with subagent_type="beta-reader":
prompt: "Beta read standalone book."
description: "Beta read standalone"
subagent_type: "beta-reader"
```

**Expected outputs:**
- Series: `./beta-reader/beta-reader-book-[N]-chapter-[NN].md` (one per chapter)
- Standalone: `./beta-reader/beta-reader-standalone-chapter-[NN].md` (one per chapter)

**Verification:**
```bash
# Count reports vs chapters
ls ./beta-reader/beta-reader-book-1-*.md 2>/dev/null | wc -l
ls ./Chapters/Book\ 1/*.md 2>/dev/null | wc -l
```

**For Series:** Repeat for each book (Book 1, Book 2, Book 3, etc.)

**Next:** Phase 8

---

### Phase 8: Beta Reader Fixes

**Agent:** `beta-reader-fixer`

**When to invoke:** After beta-reader completes for a book

**Invocation (Series):**
```
Use Task tool with subagent_type="beta-reader-fixer":
prompt: "Fix all beta-reader issues for Book 1."
description: "Fix Book 1 beta issues"
subagent_type: "beta-reader-fixer"
```

**Invocation (Standalone):**
```
Use Task tool with subagent_type="beta-reader-fixer":
prompt: "Fix all beta-reader issues for standalone book."
description: "Fix standalone beta issues"
subagent_type: "beta-reader-fixer"
```

**Expected outputs:**
- `./beta-reader-fixes/beta-reader-fix-report-[book-name]-[YYYY-MM-DD].md`
- Edits to chapter files (in place)

**Verification:**
```bash
ls ./beta-reader-fixes/beta-reader-fix-report-*.md
grep "Fixed:" ./beta-reader-fixes/beta-reader-fix-report-*.md
```

**For Series:** Repeat for each book (Book 1, Book 2, Book 3, etc.)

**Next:** Pipeline complete

---

## Complete Workflow Examples

### Example 1: Standalone Book (Full Pipeline)

**User request:** "Create a psychological thriller standalone book"

**Your orchestration:**

```
1. Phase 1: Invoke foundation-builder
   - Creates logline.md + book-spine.md

2. Phase 2: Invoke outline-architect
   - Creates Bible + 25 chapter outlines

3. Phase 3: Invoke qa-validator for outline-architect
   - Validates outlines complete

4. Phase 4: Invoke outline-critique loop
   - Iteration 1: Find issues, fix
   - Iteration 2: Find issues, fix
   - Iteration 3: No issues found → exit loop

5. Phase 5: Invoke context-guide
   - Creates context-guide.md

6. Phase 6: Invoke chapter-writer + qa-validator (paired) 9 times
   - Pair 1: Write chapters 1-3 → QA validate
   - Pair 2: Write chapters 4-6 → QA validate
   - Pair 3: Write chapters 7-9 → QA validate
   - Pair 4: Write chapters 10-12 → QA validate
   - Pair 5: Write chapters 13-15 → QA validate
   - Pair 6: Write chapters 16-18 → QA validate
   - Pair 7: Write chapters 19-21 → QA validate
   - Pair 8: Write chapters 22-24 → QA validate
   - Pair 9: Write chapter 25 → QA validate

7. Phase 7: Invoke beta-reader
   - Creates 25 issue reports

8. Phase 8: Invoke beta-reader-fixer
   - Fixes all issues

Done! Book complete.
```

---

### Example 2: Series (3 Books, Full Pipeline)

**User request:** "Create a fantasy series" (defaults to trilogy/3 books)

**Your orchestration:**

```
1. Phase 1: Invoke foundation-builder
   - Creates logline.md + series-spine.md

2. Phase 2: Invoke outline-architect
   - Creates Book 1 Bible + 30 chapters
   - Creates Book 2 Bible + 28 chapters
   - Creates Book 3 Bible + 32 chapters

3. Phase 3: Invoke qa-validator for outline-architect
   - Validates outlines complete

4. Phase 4: Invoke outline-critique loop
   - Iteration 1: Find issues across ALL books, fix
   - Iteration 2: Find issues, fix
   - Iteration 3: No issues found → exit loop

5. Phase 5: Invoke context-guide
   - Creates context-guide.md for all books

6. Phase 6a: Write Book 1 chapters (paired)
   - Pair 1: Write chapters 1-3 → QA validate
   - Pair 2: Write chapters 4-6 → QA validate
   - Pair 3: Write chapters 7-9 → QA validate
   - ... (continue pairs)
   - Pair 10: Write chapters 28-30 → QA validate

7. Phase 7a: Invoke beta-reader for Book 1
   - Creates 30 issue reports

8. Phase 8a: Invoke beta-reader-fixer for Book 1
   - Fixes all Book 1 issues

9. Phase 6b: Write Book 2 chapters (paired)
   - Pair 1: Write chapters 1-3 → QA validate (reads Book 1's last chapter)
   - Pair 2: Write chapters 4-6 → QA validate
   - Pair 3: Write chapters 7-9 → QA validate
   - ... (continue pairs)
   - Pair 10: Write chapters 28 → QA validate

10. Phase 7b: Invoke beta-reader for Book 2
    - Creates 28 issue reports

11. Phase 8b: Invoke beta-reader-fixer for Book 2
    - Fixes all Book 2 issues

12. Phase 6c: Write Book 3 chapters (paired)
    - Pair 1: Write chapters 1-3 → QA validate (reads Book 2's last chapter)
    - Pair 2: Write chapters 4-6 → QA validate
    - Pair 3: Write chapters 7-9 → QA validate
    - ... (continue pairs)
    - Pair 11: Write chapters 31-32 → QA validate

13. Phase 7c: Invoke beta-reader for Book 3
    - Creates 32 issue reports

14. Phase 8c: Invoke beta-reader-fixer for Book 3
    - Fixes all Book 3 issues

Done! Series complete.
```

---

## Decision Trees

### How to Determine Series vs Standalone?

```bash
# Check spine file
if [ -f ./series-spine.md ]; then
    echo "SERIES"
    # Extract book count from series-spine.md
elif [ -f ./book-spine.md ]; then
    echo "STANDALONE"
fi
```

### How to Resume Work in Progress?

**Foundation exists but no outlines:**
```bash
ls ./logline.md && ! ls -d ./Outlines 2>/dev/null
# → Start from Phase 2 (outline-architect)
```

**Outlines exist but no chapters:**
```bash
ls -d ./Outlines && ! ls -d ./Chapters 2>/dev/null
# → Start from Phase 4 (chapter-writer)
```

**Some chapters exist:**
```bash
# Find last written chapter
ls ./Chapters/Book\ 1/*.md | sort -V | tail -1
# Returns: Book 1 Chapter 12.md
# → Resume from Phase 4: "Write chapters 13-15 for Book 1"
```

**Chapters complete but no beta-reader reports:**
```bash
ls -d ./Chapters && ! ls -d ./beta-reader 2>/dev/null
# → Start from Phase 5 (beta-reader)
```

**Beta reports exist but no fixes:**
```bash
ls -d ./beta-reader && ! ls -d ./beta-reader-fixes 2>/dev/null
# → Start from Phase 6 (beta-reader-fixer)
```

### How Many Chapters Per Book?

```bash
# Series Book 1
ls ./Outlines/Book\ 1\ Outlines/Outline\ Chapter*.md | wc -l

# Standalone
ls ./Outlines/Standalone/Outline\ Chapter*.md | wc -l
```

### How Many chapter-writer Invocations Needed?

```python
import math
chapter_count = 30  # from above
invocations_needed = math.ceil(chapter_count / 3)
# Result: 10 invocations
```

---

## Error Handling

### If foundation-builder fails:
- Check: Does user input contain enough information?
- Check: Is WebSearch working (if market research needed)?
- Ask user for more details

### If outline-architect fails:
- Check: Do foundation files exist?
- Check: Are foundation files valid (have content)?
- Re-invoke with explicit instructions

### If chapter-writer fails:
- Check: Do outlines exist for requested chapters?
- Check: Does Bible file exist?
- Check: If Book 2+, does previous book's last chapter exist?
- Verify chapter numbers in prompt match available outlines

### If beta-reader fails:
- Check: Do chapters exist in `./Chapters/`?
- Check: Are chapters readable (not empty)?
- Re-invoke with correct book number

### If beta-reader-fixer fails:
- Check: Do beta-reader reports exist?
- Check: Are reports readable (not empty)?
- Re-invoke with correct book number

---

## File Structure Reference

**After complete pipeline:**

```
./
├── logline.md                                    # Phase 1
├── series-spine.md OR book-spine.md              # Phase 1
│
├── Outlines/                                     # Phase 2
│   ├── Book 1 Outlines/
│   │   ├── Book 1 Bible.md
│   │   ├── context-guide.md                      # Phase 5
│   │   └── Outline Chapter 01.md ... [NN].md
│   ├── Book 2 Outlines/                          # (series only)
│   │   ├── Book 2 Bible.md
│   │   ├── context-guide.md                      # Phase 5
│   │   └── Outline Chapter 01.md ... [NN].md
│   ├── Standalone/                               # (standalone only)
│   │   ├── Book Bible.md
│   │   ├── context-guide.md                      # Phase 5
│   │   └── Outline Chapter 01.md ... [NN].md
│   ├── Critique/                                 # Phase 4
│   │   └── outline-critique-report-*.md
│   └── QA-Reports/                               # Phase 3 (outlines), Phase 6 (chapters)
│       └── qa-*.md
│
├── Chapters/                                     # Phase 6
│   ├── Book 1/
│   │   └── Book 1 Chapter 01.md ... [NN].md
│   ├── Book 2/                                   # (series only)
│   │   └── ...
│   └── Standalone/                               # (standalone only)
│       └── Standalone Chapter 01.md ... [NN].md
│
├── beta-reader/                                  # Phase 7
│   └── beta-reader-book-1-chapter-01.md ... [NN].md
│
└── beta-reader-fixes/                            # Phase 8
    └── beta-reader-fix-report-*.md
```

---

## Key Principles for Autonomous Execution

1. **Always check what exists before invoking agents**
   - Use `ls` and file checks
   - Resume from correct phase
   - Don't duplicate work

2. **Pair chapter-writer with QA validation**
   - Write 3 chapters, then validate those 3 chapters
   - Calculate total pairs needed (total chapters ÷ 3)
   - Track progress (which chapters written and validated)
   - Continue pairs until all chapters complete

3. **Process series sequentially**
   - Complete Book 1 fully (write+QA pairs → beta → fix)
   - Then Book 2 (write+QA pairs → beta → fix)
   - Then Book 3 (write+QA pairs → beta → fix)

4. **Verify outputs after each phase**
   - Check files created
   - Check file counts match expectations
   - Check content exists (not empty files)

5. **Use Task tool for all agent invocations**
   - Specify subagent_type correctly
   - Provide clear, complete prompts
   - Include context (book number, chapter range)

6. **Handle resume gracefully**
   - Agents have built-in resume capability
   - Check existing work before invoking
   - Don't re-invoke for already-complete work

7. **Track state throughout pipeline**
   - Know which phase you're in
   - Know what's complete, what's pending
   - Communicate progress to user

---

## Agent Tool Availability

| Agent | Read | Write | Edit | Bash | Glob | Grep | TodoWrite | WebSearch |
|-------|------|-------|------|------|------|------|-----------|-----------|
| foundation-builder | ✓ | ✓ | | ✓ | | | | ✓ |
| outline-architect | ✓ | ✓ | | ✓ | | | | |
| outline-critique | ✓ | ✓ | ✓ | ✓ | | | ✓ | |
| chapter-writer | ✓ | ✓ | | ✓ | | | | |
| beta-reader | ✓ | ✓ | | ✓ | | | | |
| beta-reader-fixer | ✓ | ✓ | ✓ | ✓ | | | ✓ | |
| qa-validator | ✓ | ✓ | ✓ | ✓ | | | | |
| project-mover | ✓ | ✓ | | ✓ | ✓ | | | |

---

## Final Checklist: Is Pipeline Complete?

**Standalone Book:**
- [ ] Foundation files exist (`logline.md`, `book-spine.md`)
- [ ] Outlines complete (Bible + all chapter outlines)
- [ ] All chapters written (count matches outline count)
- [ ] Beta-reader reports exist (one per chapter)
- [ ] Beta-reader fixes complete (report shows 100%)
- [ ] All files in expected locations

**Series (per book):**
- [ ] Foundation files exist (`logline.md`, `series-spine.md`)
- [ ] Outlines complete for ALL books
- [ ] Book 1: All chapters written + beta-read + fixed
- [ ] Book 2: All chapters written + beta-read + fixed
- [ ] Book 3: All chapters written + beta-read + fixed
- [ ] (etc. for all books in series)
- [ ] All files in expected locations

**When complete, inform user:**
```
"Pipeline complete!

[Standalone/Series] book finished:
- [N] chapters written ([X] words total)
- [N] beta-reader reports generated
- [N] issues identified and fixed

Files located in:
- Outlines: ./Outlines/
- Chapters: ./Chapters/
- Reports: ./beta-reader/, ./beta-reader-fixes/

Next steps:
- Review final chapters in ./Chapters/
- Run project-mover to archive (optional)
- Export/publish as needed"
```

---

## Notes for Future Claude

- **You are orchestrating, not executing directly** - Use Task tool to invoke specialized agents
- **Each agent is autonomous** - They read their own inputs, create their own outputs
- **Your job: sequence and verify** - Invoke in correct order, check outputs, proceed to next phase
- **Resume is built-in** - Agents detect existing work, don't duplicate
- **Series = sequential books** - Complete each book fully before moving to next
- **chapter-writer + QA paired** - Write 3 chapters, validate, repeat until all chapters complete
- **Communicate progress** - Keep user informed of current phase and completion status

---

**End of autonomous orchestration instructions**
