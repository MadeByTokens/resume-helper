---
description: Start adversarial resume development loop with three agents
allowed-tools: Write, Read, Glob, Grep, Edit, Bash, Task, TodoWrite, AskUserQuestion, Skill
---

# Resume Development Loop

You are orchestrating an adversarial three-agent loop to develop a compelling AND honest resume.

## Agents

- **Resume Writer** (Advocate): Creates compelling resume content
- **Interviewer** (Skeptic): Reviews from hiring manager perspective
- **Coach** (Mediator): Synthesizes feedback, ensures honesty

## Step 1: Parse User Input

Extract from user's command:
- `experience_path`: Path to candidate's experience/background file (REQUIRED)
- `--job` or `--jd`: Path to job description file (OPTIONAL)
- `--max-iterations`: Maximum loop iterations (DEFAULT: 5)
- `--max-pages`: Maximum page length - 1, 2, or 3 (DEFAULT: 1, requires confirmation)
- `--output`: Output path for final resume (DEFAULT: ./resume_final.md)
- `--format`: Resume format - traditional, modern, ats (DEFAULT: ats)

Example commands:
```
/resume-loop "my_experience.md"
/resume-loop "experience.md" --job "job_description.md"
/resume-loop "exp.md" --job "jd.md" --max-pages 2
/resume-loop "exp.md" --job "jd.md" --max-iterations 3 --output "./my_resume.md"
```

## Step 2: Validate Input Files

### Experience File (required)

1. Display: "📄 Loading experience from: `<experience_path>`"
2. Attempt to read the file using the Read tool
3. **If the file does not exist or cannot be read:**
   - Display: "❌ Experience file not found: `<experience_path>`"
   - Display: "Please check the path and try again."
   - **STOP** - do not continue
4. **If the file is empty:**
   - Display: "❌ Experience file is empty: `<experience_path>`"
   - **STOP** - do not continue
5. Display: "✅ Experience loaded (`<N>` characters)"

### Job Description File (optional)

If `--job` was provided:

1. Display: "📄 Loading job description from: `<job_path>`"
2. Attempt to read the file using the Read tool
3. **If the file does not exist or cannot be read:**
   - Display: "❌ Job description file not found: `<job_path>`"
   - Display: "Please check the path and try again."
   - **STOP** - do not continue
4. **If the file is empty:**
   - Display: "❌ Job description file is empty: `<job_path>`"
   - **STOP** - do not continue
5. Display: "✅ Job description loaded (`<N>` characters)"

## Step 2.5: Confirm Page Limit

Page-to-word conversion:
- 1 page = 450 words maximum
- 2 pages = 900 words maximum
- 3 pages = 1350 words maximum

### If `--max-pages` was NOT explicitly provided:

1. Display:
   ```
   ════════════════════════════════════════════════════
   PAGE LIMIT CONFIGURATION
   ════════════════════════════════════════════════════

   No --max-pages specified. The default is 1 page (~450 words).

   Recommended page limits:
     1 page  = ~450 words  (recommended for <10 years experience)
     2 pages = ~900 words  (recommended for 10-20 years experience)
     3 pages = ~1350 words (recommended for executives/academics)

   ════════════════════════════════════════════════════
   ```

2. Use AskUserQuestion tool to ask:
   - Question: "Would you like to proceed with the 1-page default, or specify a different limit?"
   - Options: "1 page (default)", "2 pages", "3 pages"

3. Parse response and set:
   - maxPages = 1, 2, or 3 based on selection
   - maxWords = 450, 900, or 1350 respectively

4. Display confirmation:
   ```
   ✅ Using [N]-page limit (~[WORDS] words maximum)
   ```

### If `--max-pages` was explicitly provided:

1. Validate the value is 1, 2, or 3
2. If invalid:
   - Display: "❌ Invalid --max-pages value: `<value>`. Must be 1, 2, or 3."
   - **STOP** - do not continue
3. Set maxWords based on maxPages (450, 900, or 1350)
4. Display: "✅ Page limit set: [N] pages (~[WORDS] words maximum)"

## Step 3: Check for Existing Loop

Read `.resume-state.json` if it exists:
- If `active: true`, inform user and ask if they want to continue or start fresh
- If starting fresh, back up old state file

## Step 4: Initialize State

Create `.resume-state.json`:

```json
{
  "active": true,
  "iteration": 0,
  "phase": "INITIALIZED",
  "candidateInput": {
    "experiencePath": "<path>",
    "experience": "<content>",
    "jobDescriptionPath": "<path or null>",
    "jobDescription": "<content or null>"
  },
  "options": {
    "maxIterations": 5,
    "maxPages": 1,
    "maxWords": 450,
    "format": "ats",
    "outputPath": "./resume_final.md"
  },
  "currentResume": null,
  "history": [],
  "lastVerdict": null,
  "interviewQuestions": [],
  "startedAt": "<ISO timestamp>",
  "completedAt": null,
  "stoppedReason": null
}
```

## Step 5: Main Loop

```
while iteration < maxIterations AND lastVerdict != "READY":
    iteration++

    === PHASE 1: WRITING ===
    Update state: phase = "WRITING"

    Invoke Resume Writer agent via Task tool:
    - Provide: candidate experience, job description (if any), coach feedback (if any)
    - DO NOT provide: interviewer's raw feedback
    - Agent type: resume-helper:resume-writer

    Save Writer's output to state.currentResume

    === PHASE 2: REVIEWING ===
    Update state: phase = "REVIEWING"

    Invoke Interviewer agent via Task tool:
    - Provide: current resume only, job description (if any)
    - DO NOT provide: candidate's raw experience, writer's notes
    - Agent type: resume-helper:interviewer

    Save Interviewer's review to state.history[iteration].interviewerReview

    === PHASE 2.5: ANALYSIS ===
    Update state: phase = "ANALYZING"

    Run analysis skills BEFORE invoking Coach (these run in plugin context with proper file access):

    1. Invoke `analyze-vague-claims` skill:
       - Args: resume_file_path=<absolute path to current resume>
       - Capture output as vague_claims_results

    2. Invoke `analyze-buzzwords` skill:
       - Args: resume_file_path=<absolute path to current resume>
       - Capture output as buzzwords_results

    3. Invoke `check-ats-compatibility` skill:
       - Args: resume_file_path=<absolute path>, job_description_path=<path or "none">, max_pages=<N>
       - Capture output as ats_results

    4. If iteration > 1 or previous verdict was NEEDS_GROUNDING, also invoke `suggest-quantification` skill:
       - Args: resume_file_path=<absolute path to current resume>
       - Capture output as quantification_results

    Combine all results into analysis_results string for Coach.

    === PHASE 3: COACHING ===
    Update state: phase = "COACHING"

    Invoke Coach agent via Task tool:
    - Provide: EVERYTHING - candidate input, resume, interviewer review, history, AND analysis_results
    - Agent type: resume-helper:coach

    Parse Coach's verdict and feedback
    Update state.lastVerdict

    If verdict == "READY":
        Break loop

    If verdict == "BLOCKED":
        Ask user for additional information via AskUserQuestion (HIGH priority questions)
        Add their response to candidateInput

    If Coach output contains MEDIUM or HIGH priority questions (even if not BLOCKED):
        Present questions to user via AskUserQuestion
        Frame as: "The Coach has some questions that would strengthen your resume:"
        Add their responses to candidateInput for next iteration
        (User can skip questions they don't want to answer)

    Save iteration to history
```

## Step 6: Finalize

When loop completes (READY verdict or max iterations):

1. Update state:
   ```json
   {
     "active": false,
     "phase": "COMPLETE",
     "completedAt": "<ISO timestamp>",
     "stoppedReason": "ready" | "max_iterations"
   }
   ```

2. Write final resume to output path

3. Write interview prep document to `interview_prep.md`

4. Display summary:
   ```
   ═══════════════════════════════════════════════════
   RESUME DEVELOPMENT COMPLETE
   ═══════════════════════════════════════════════════

   Iterations: X
   Final Verdict: [verdict]

   Output Files:
   - Resume: [output path]
   - Interview Prep: interview_prep.md

   Key Improvements Made:
   - [Summary of changes across iterations]

   Interview Questions to Prepare:
   - [Top questions from Interviewer]
   ═══════════════════════════════════════════════════
   ```

## Agent Invocation Templates

### Resume Writer Task

```
You are the Resume Writer agent. Create/improve a resume for this candidate.

## Candidate Experience
<experience content>

## Target Job Description (if provided)
<job description or "Not provided - create a general-purpose resume">

## Previous Coach Feedback (if iterating)
<coach feedback or "This is the first iteration">

## Current Resume (if iterating)
<current resume or "Create initial resume">

Follow your agent instructions to produce an updated resume.
```

### Interviewer Task

```
You are the Interviewer agent. Review this resume as a hiring manager would.

## Resume to Review
<current resume>

## Job Description (if provided)
<job description or "General review - no specific role">

Follow your agent instructions to produce a thorough review.
```

### Coach Task

```
You are the Coach agent. Synthesize the Writer's resume and Interviewer's review.

## Candidate's Original Input
<experience content>

## Target Job Description
<job description or "Not provided">

## Current Resume (from Writer)
<current resume>

## Interviewer's Review
<interviewer review>

## Iteration History
<previous iterations summary>

## Page/Word Limit
Maximum: [maxPages] pages ([maxWords] words)

## Analysis Results

The following analysis was performed on the current resume:

### Vague Claims Analysis
<vague_claims_results>

### Buzzwords Analysis
<buzzwords_results>

### ATS Compatibility
<ats_results>

### Quantification Suggestions (if available)
<quantification_results or "Not run this iteration">

---

Follow your agent instructions to:
1. Incorporate the analysis results above into your Tool Analysis Results table
2. Verify claims against candidate input
3. Translate interviewer concerns into writer guidance
4. **ALWAYS include 2-3+ questions for the candidate** with priority levels (HIGH/MEDIUM/LOW)
   - Even when the resume is improving, there's always something that could be more specific
5. Issue a verdict: READY, NEEDS_STRENGTHENING, NEEDS_GROUNDING, or BLOCKED
   - Do NOT issue READY if high-severity vague claims remain, ATS score < 80, or resume exceeds page limit
```

## Error Handling

- If any agent fails, save state and inform user
- If candidate input is insufficient, enter BLOCKED state and ask for more info
- If max iterations reached without READY, inform user of remaining concerns

## Important Notes

1. **Information Isolation**: Maintain strict separation - Interviewer never sees raw candidate input
2. **State Persistence**: Always update `.resume-state.json` after each phase
3. **User Communication**: Keep user informed of progress between iterations
4. **Honesty First**: If Coach flags honesty concerns, prioritize those over polish
