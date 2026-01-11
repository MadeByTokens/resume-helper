---
name: check-ats-compatibility
description: Check a resume for ATS (Applicant Tracking System) compatibility. Use this skill to verify formatting, keyword alignment with job descriptions, and page length compliance.
allowed-tools: Bash, Read
---

# Check ATS Compatibility

This skill analyzes a resume for ATS compatibility, including formatting issues, keyword matching, section detection, and page limit compliance.

## Usage

When invoked, you will receive:
- `resume_file_path` (required): Path to the resume file
- `job_description_path` (optional): Path to job description for keyword matching
- `max_pages` (optional): Page limit (1, 2, or 3) for length checking

## Instructions

1. Parse the provided parameters
2. Run the Python analysis tool using Bash:

   **With job description and page limit:**
   ```bash
   python3 tools/ats_compatibility.py "<resume_file_path>" "<job_description_path>" --max-pages <N> --json
   ```

   **With page limit only (no JD):**
   ```bash
   python3 tools/ats_compatibility.py "<resume_file_path>" --max-pages <N> --json
   ```

   **Basic check (no JD, no page limit):**
   ```bash
   python3 tools/ats_compatibility.py "<resume_file_path>" --json
   ```

3. Parse the JSON output and present the findings in a structured format

## Output Protocol

Return your analysis in this format:

```markdown
## ATS Compatibility Analysis

**ATS Score:** [score]/100
**Word Count:** [count] words
**Page Limit Status:** [X] / [Y] words ([WITHIN LIMIT / OVER BY Z words])

### Sections
- **Found:** [list of sections found]
- **Missing:** [list of required sections missing]

### Issues by Severity

#### High Severity (blocks ATS parsing)
| Category | Issue | Fix |
|----------|-------|-----|
| [category] | [issue] | [suggestion] |

#### Medium Severity (may reduce score)
| Category | Issue | Fix |
|----------|-------|-----|
| [category] | [issue] | [suggestion] |

#### Low Severity (minor improvements)
| Category | Issue | Fix |
|----------|-------|-----|
| [category] | [issue] | [suggestion] |

### Keyword Analysis (if JD provided)
**Match Rate:** [matched]/[total] ([percentage]%)

**Missing Keywords:** [list of important missing keywords]

**Found Keywords:** [list of matched keywords with context]
```

## Error Handling

If the tool fails to run:
1. Check if the file paths exist using Read tool
2. Report the error clearly
3. Suggest potential fixes
