---
name: analyze-buzzwords
description: Analyze a resume for overused buzzwords and corporate jargon. Use this skill to identify empty phrases that should be replaced with concrete, specific language.
allowed-tools: Bash, Read
---

# Analyze Buzzwords

This skill analyzes a resume file to detect overused buzzwords and corporate jargon that reduce clarity and impact.

## Usage

When invoked, you will receive a resume file path. Run the analysis tool and return the results.

## Instructions

1. You will be given a `resume_file_path` parameter
2. Run the Python analysis tool using Bash:
   ```bash
   python3 tools/detect_buzzwords.py "<resume_file_path>" --json
   ```
3. Parse the JSON output and present the findings in a structured format

## Output Protocol

Return your analysis in this format:

```markdown
## Buzzword Analysis

**Clarity Score:** [score]/100
**Total Buzzwords Found:** [count] ([high] high, [medium] medium, [low] low severity)

### High Severity (empty jargon - replace immediately)
| Line | Buzzword | Category | Suggested Alternative |
|------|----------|----------|----------------------|
| [line] | [word/phrase] | [category] | [concrete alternative] |

### Medium Severity (overused - consider replacing)
| Line | Buzzword | Category | Suggested Alternative |
|------|----------|----------|----------------------|
| [line] | [word/phrase] | [category] | [concrete alternative] |

### Low Severity (acceptable but watch frequency)
| Line | Buzzword | Category | Suggested Alternative |
|------|----------|----------|----------------------|
| [line] | [word/phrase] | [category] | [concrete alternative] |
```

## Error Handling

If the tool fails to run:
1. Check if the file path exists using Read tool
2. Report the error clearly
3. Suggest potential fixes (e.g., check file path)
