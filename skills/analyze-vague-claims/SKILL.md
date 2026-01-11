---
name: analyze-vague-claims
description: Analyze a resume for vague or unquantified claims. Use this skill when you need to identify statements that lack specific numbers, metrics, or concrete details.
allowed-tools: Bash, Read
---

# Analyze Vague Claims

This skill analyzes a resume file to detect vague or unquantified claims that should be made more specific.

## Usage

When invoked, you will receive a resume file path. Run the analysis tool and return the results.

## Instructions

1. You will be given a `resume_file_path` parameter
2. Run the Python analysis tool using Bash:
   ```bash
   python3 tools/detect_vague_claims.py "<resume_file_path>" --json
   ```
3. Parse the JSON output and present the findings in a structured format

## Output Protocol

Return your analysis in this format:

```markdown
## Vague Claims Analysis

**Score:** [clarity_score]/100
**Total Issues:** [count] ([high] high, [medium] medium, [low] low severity)

### High Severity Issues (must fix)
| Line | Claim | Suggestion |
|------|-------|------------|
| [line] | [vague text] | [how to quantify] |

### Medium Severity Issues (should fix)
| Line | Claim | Suggestion |
|------|-------|------------|
| [line] | [vague text] | [how to quantify] |

### Low Severity Issues (nice to fix)
| Line | Claim | Suggestion |
|------|-------|------------|
| [line] | [vague text] | [how to quantify] |
```

## Error Handling

If the tool fails to run:
1. Check if the file path exists using Read tool
2. Report the error clearly
3. Suggest potential fixes (e.g., check file path)
