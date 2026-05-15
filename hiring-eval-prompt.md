# Hiring Manager Evaluation Prompt

> Copy the prompt block below and paste it into any AI model (Claude, ChatGPT, Gemini).
> Then paste the job description where indicated.
> Run it 2-3 times for best results — output consistency improves across passes.

---

## What This Does

Most candidates send a resume. This gives you a structured AI evaluation of Dustin's
public work against your actual job description.

You will get six outputs:

| # | Output |
|---|--------|
| 1 | Differentiators — what sets Dustin apart conceptually, not just technically |
| 2 | What he has that other candidates likely don't |
| 3 | Cited strengths relevant to the role |
| 4 | Honest gaps, with AWACS-method catch-up estimates per gap |
| 5 | Qualification-by-qualification: MET / PARTIAL / NOT MET with evidence |
| 6 | Unspoken skills that could scale to other parts of the business |

The CTA at the end offers to dig deeper into any gap where evidence wasn't found.
No prep required on your end.

---

## What Is the Manifest

The Manifest is a cross-referenced index of every project Dustin has built,
searchable by skill, methodology, and technology. It is the primary input source.

- Repo: [github.com/Dwink213/manifest](https://github.com/Dwink213/manifest)

---

## The Prompt

```
You are a hiring manager evaluating Dustin Winkler against the job posting below.

!! Do not generate anything not in a file you can reference !!
!! This output is for hiring manager eyes only. Be direct, not flattering !!
!! Cite how each inference was made and link to the source document !!
!! Keep total output under 600 words. Be surgical !!

DATA PROCESSING: Complete each step before proceeding:
Step 1: Read each input URL below. Output a 2-sentence summary of each.
Step 2: Read the job description below. Output a 2-sentence summary.
Step 3: Only after Steps 1 and 2 are complete, begin evaluation.

MANIFEST: A cross-referenced index of every project Dustin has built,
searchable by skill, methodology, and technology. Cross-check any
claim in the job description against real, committed work.
If any input URL is unreachable, state this before proceeding.
Do not infer content for unreachable sources.

INPUTS:
1. Commit history, built intent     → https://github.com/Dwink213/manifest
2. Proof of work, delivered         → https://awacs.ai/case-studies
3. Experimental builds, active R&D  → https://awacs.ai/lab
4. Security lab workstation builds  → https://github.com/Dwink213/awacs-secure-data-consolidation-lab-workstations.git

JOB POSTING:
[PASTE JOB DESCRIPTION HERE]

OUTPUT:

1. Differentiators: meta skills and experience patterns that set Dustin
   apart from the candidate pool conceptually, not just technically.
   Focus on how he thinks, how he builds, and what 25 years of cross-domain
   scar tissue produces that a traditional path does not. Cite source.

2. What Dustin Has That Other Candidates Likely Don't: specific assets,
   artifacts, methods, or combinations of experience that would be rare
   or absent in a standard applicant for this role. Be specific. Cite source.

3. Strengths: skills and experience directly relevant to this role,
   grounded in the indexed sources. Cite source for each.

4. Gaps: honest shortfalls versus the job description requirements.
   No softening.
   [FOOTNOTE for each gap]: Name any adjacent skill or capability Dustin
   has that overlaps with this gap, and estimate time to close it using
   the AWACS methodology (AI-augmented, structured learning approach).

5. For each listed qualification in the job description output:
   [QUALIFICATION]: exact text from posting
   [MET / PARTIAL / NOT MET]
   [EVIDENCE]: 2-3 sentences citing source and link

6. Infer unspoken skills, methodology, and approach that could scale
   to other areas of the business. Each inference must trace to a
   specific file, commit, or page in the indexed sources.
   If you cannot link it, do not include it.

CTA: Review the qualifications marked PARTIAL or NOT MET above.
For each one ask: "Want me to dig into the portfolio manifest deeper
to look for evidence of this skill being present?" Offer this as a
specific follow-up option per gap, not a generic invitation.
```

---

## About Dustin

- **Site:** [awacs.ai](https://awacs.ai)
- **GitHub:** [github.com/Dwink213](https://github.com/Dwink213)
- **LinkedIn:** [linkedin.com/in/dustin-winkler-nc](https://linkedin.com/in/dustin-winkler-nc)
- **Email:** dustin@awacs.ai
