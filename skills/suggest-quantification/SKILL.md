---
name: suggest-quantification
description: Suggest specific questions and templates to help quantify vague resume claims. Use this skill when claims need more specific numbers, metrics, or concrete details.
allowed-tools: Bash, Read
---

# Suggest Quantification

This skill analyzes a resume to identify claims that need quantification and provides specific questions to ask the candidate along with templates for quantified rewrites.

## Usage

When invoked, you will receive a resume file path. Run the analysis tool and return the results.

## Instructions

1. You will be given a `resume_file_path` parameter
2. Run the Python analysis tool using Bash:
   ```bash
   python3 tools/quantification_helper.py "<resume_file_path>" --json
   ```
3. Parse the JSON output and present the findings in a structured format

## Output Protocol

Return your analysis in this format:

```markdown
## Quantification Suggestions

**Total Claims Found:** [count] claims needing quantification

### Claims to Quantify

#### Claim 1: "[original claim text]"
**Line:** [line number]
**Type:** [achievement/responsibility/skill]

**Questions to Ask Candidate:**
1. [specific question about numbers]
2. [specific question about scale]
3. [specific question about impact]

**Template for Rewrite:**
> [Template with placeholders like {team_size}, {percentage}, {timeframe}]

---

#### Claim 2: "[original claim text]"
**Line:** [line number]
**Type:** [achievement/responsibility/skill]

**Questions to Ask Candidate:**
1. [specific question]
2. [specific question]

**Template for Rewrite:**
> [Template with placeholders]

---

[Continue for all claims...]
```

## Usage Notes

- Use this skill when the Coach needs to formulate questions for the candidate
- The questions generated should be included in the Coach's "Questions for Candidate" section
- The templates help the Writer know what format to use once answers are provided

## Error Handling

If the tool fails to run:
1. Check if the file path exists using Read tool
2. Report the error clearly
3. Suggest potential fixes
