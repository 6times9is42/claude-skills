---
name: resume-writer
description: >
  Craft a polished, role-tailored resume in the Jake Ryan LaTeX template format and compile it to
  a downloadable PDF. Trigger this skill whenever the user wants to: write or update a resume,
  tailor a resume to a specific role or company, create a resume from scratch, improve or reformat
  an existing CV/resume, or asks anything like "make me a resume", "update my CV for a SWE role",
  "help me apply to [company]", "create a resume for a PM job", "redo my resume in a clean format",
  or "I want to apply to [role/company], can you help with my resume". Always use this skill — even
  for partial requests like "just fix my skills section" or "add my new project to my resume".
---

# Resume Writer

Produces a clean, one-page PDF resume in the Jake Ryan LaTeX template — the most widely recognised
clean resume format — tailored specifically to the user's target role and company type.

The LaTeX preamble (macros, packages, formatting commands) lives in:
`assets/jake_ryan_preamble.tex` — read it before writing any LaTeX.

Role-tailoring guidance (what to emphasise, how to write bullets, section ordering per role) lives in:
`references/role_tailoring.md` — read it before drafting any content.

---

## Overview of the Flow

The skill has three distinct paths. Always ask the user which one they need before doing anything else.

```
PATH A — CREATE      New resume from scratch
PATH B — UPDATE      Improve or add to an existing resume
PATH C — TAILOR      Optimise an existing resume for a specific job posting
                     └─ invokes agents/recruiter.md for analysis before rewriting
```

---

## Step 1 — Path Selection

Open every conversation with this single question:

> "What would you like to do?
> - **A) Create** — build a new resume from scratch
> - **B) Update** — improve or add to an existing resume
> - **C) Tailor** — optimise your resume for a specific job you're applying to"

Wait for the user's choice. Do not proceed until you have it.

**→ If A (Create):** go to Step 2b (Collection from Scratch)
**→ If B (Update):** go to Step 2a (Extraction), then ask what they want to change or add
**→ If C (Tailor):** go to Step 1C (Recruiter Path) below — do NOT go to Step 2 yet

---

## Step 1C — Recruiter Path (Path C only)

Read `agents/recruiter.md` in full before proceeding.

This path runs in two phases:

**Phase 1 — Analysis**
First, you need the resume. Ask:
> "Great — please upload your current resume (PDF or DOCX) or paste its contents directly."

Extract the resume content using the same method as Step 2a. Then hand off to the recruiter agent persona defined in `agents/recruiter.md`. The recruiter will:
1. Ask for company name, role title, and full job description (in a single message)
2. Run the full analysis protocol from `agents/recruiter.md`
3. Deliver the structured output: match score, top 5 missing keywords, 3 red flags, and overall take

Do not skip the analysis or jump straight to rewriting. The whole value of Path C is that the rewrite is *informed* by a rigorous audit. The user needs to see the diagnosis before they see the treatment.

**Phase 2 — Targeted Rewrite**
After delivering the analysis, follow the Tailoring Protocol defined in `agents/recruiter.md`:
- Add missing keywords where genuinely supported by the candidate's background
- Remove or reframe the identified red flags
- Reorder content to surface the most JD-relevant material first
- Tighten all bullets to CAR format with vocabulary matched to this specific JD

Then proceed to Step 4 (LaTeX Generation) to compile the updated resume.

If a critical JD requirement is completely absent from the candidate's background, state it clearly after the rewrite rather than silently omitting it.

---

## Step 2a — Extraction from Uploaded Resume (Paths B and C)

When the user uploads a file:
- For **PDF**: use `bash_tool` to extract text with `pdftotext` or read it via Python if needed.
- For **DOCX**: use `bash_tool` with `python-docx` or `catdoc` to extract plain text.
- For **text pasted directly**: use it as-is.

Once extracted, mentally parse the resume into sections:
- Personal info (name, email, phone, LinkedIn, GitHub)
- Education entries
- Experience entries  
- Project entries
- Skills

Then confirm with the user: summarise what you found, flag anything that seems missing or unclear, and ask what they'd like to add, change, or remove. Example:

> "Got it — I can see your NTU education, your internship at Mahindra.AI, and 3 projects. A few things I want to confirm: [list gaps]. And what would you like to add or change for this application?"

Do not proceed to generation until you have confirmed the content with the user.

---

## Step 2b — Collection from Scratch (Path A only)

Collect content through conversation — not a long form dump. Ask in logical groups:

**Message 1** — Personal info:
> "Let's start with the basics: full name, phone number, email, LinkedIn URL, and GitHub URL (if you have one)."

**Message 2** — Education:
> "Tell me about your education — university name, degree, GPA (optional), graduation date, and any awards, scholarships, or relevant coursework worth mentioning."

**Message 3** — Experience:
> "Any internships, jobs, or research positions? For each one: company/org, your role, dates, location, and what you worked on. Bullet points or even rough notes are fine — I'll polish them."

**Message 4** — Projects:
> "What projects are you proud of? For each: project name, what it does, tech stack, dates, and any standout results (metrics, users, accuracy scores, etc.)."

**Message 5** — Skills + wrap-up:
> "Finally: programming languages and frameworks you're comfortable with, any certifications, and anything else you want on there (interests, publications, competitions)?"

After collecting everything, give a quick summary and confirm before generating.

---

## Step 3 — Tailoring (all paths)

Before writing a single line of LaTeX, read `references/role_tailoring.md`.

**For Path C specifically:** the tailoring decisions here are driven by the recruiter's analysis from Step 1C — not general role guidance. The missing keywords and red flags identified by the recruiter are your primary rewrite targets. Use role_tailoring.md for vocabulary and CAR guidance, but let the JD be the compass.

Based on the target role and company type:

1. **Choose section order** — the table in role_tailoring.md gives recommended orderings. New grads with strong projects often benefit from Projects before Experience.

2. **Decide what to include/exclude** — ruthlessly cut anything not relevant to the target role. A quant fund resume should not feature a web portfolio unless it's quantitative.

3. **Write or rewrite every bullet using the CAR framework** (defined in full in role_tailoring.md):
   - **Challenge** — the problem, constraint, or opportunity that made the work non-trivial (often a brief clause)
   - **Action** — what you specifically built or decided, led by a strong active verb
   - **Result** — the quantified outcome; if the user hasn't provided a metric, ask before drafting

   All three must appear in every bullet, compressed into 1–2 lines. A bullet with no Challenge is just a task description. A bullet with no Result is an activity log. Neither belongs on a strong resume.

   If the user's raw notes are missing a component, ask:
   - No Challenge: *"What problem or constraint made this work necessary?"*
   - No Result: *"What changed after? Any numbers — accuracy, latency, users, cost saved?"*
   - No specificity: *"What was your individual contribution vs. the team's?"*

   Never invent a metric. Write the bullet without it and mark it:
   `% [ADD METRIC — ask user]`

4. **Adjust project descriptions** — reframe the same project differently per role:
   - For SWE: emphasise architecture, stack choices, performance
   - For ML: emphasise model metrics, dataset, approach
   - For quant: emphasise mathematical rigour, simulation, statistics
   - For consulting: emphasise the problem solved and business impact

---

## Step 4 — LaTeX Generation

### 4a. Read the preamble

Read `assets/jake_ryan_preamble.tex` in full before writing any LaTeX. Do not rewrite the macros
— use exactly the commands defined there: `\resumeSubheading`, `\resumeProjectHeading`,
`\resumeItem`, `\resumeSubHeadingListStart/End`, `\resumeItemListStart/End`.

### 4b. Write the full .tex file

Structure:

```latex
% Paste the full contents of assets/jake_ryan_preamble.tex here

\begin{document}

%----------HEADING----------
\begin{center}
    \textbf{\Huge \scshape [Full Name]} \\ \vspace{1pt}
    \small [Phone] $|$
    \href{mailto:[email]}{\underline{[email]}} $|$
    \href{[linkedin]}{\underline{[linkedin display]}}
    % Add GitHub if provided:
    $|$ \href{[github]}{\underline{[github display]}}
\end{center}

%-----------EDUCATION-----------
\section{Education}
  \resumeSubHeadingListStart
    \resumeSubheading
      {University Name}{City, Country}
      {Degree, GPA (if provided)}{Start -- End}
      \resumeItemListStart
        \resumeItem{Award / scholarship / relevant coursework bullet}
      \resumeItemListEnd
  \resumeSubHeadingListEnd

%-----------EXPERIENCE-----------
\section{Experience}
  \resumeSubHeadingListStart
    \resumeSubheading
      {Company Name}{Start Date -- End Date}
      {Role Title}{City, Country}
      \resumeItemListStart
        \resumeItem{Tailored bullet}
        \resumeItem{Tailored bullet}
      \resumeItemListEnd
  \resumeSubHeadingListEnd

%-----------PROJECTS-----------
\section{Projects}
    \resumeSubHeadingListStart
      \resumeProjectHeading
          {\textbf{Project Name} $|$ \emph{Tech stack}}{Date range}
          \resumeItemListStart
            \resumeItem{Tailored bullet}
          \resumeItemListEnd
    \resumeSubHeadingListEnd

%-----------TECHNICAL SKILLS-----------
\section{Technical Skills}
 \begin{itemize}[leftmargin=0.15in, label={}]
    \small{\item{
     \textbf{Languages}{: ...} \\
     \textbf{Frameworks \& Tools}{: ...} \\
     \textbf{Certifications}{: ...}
    }}
 \end{itemize}

\end{document}
```

### LaTeX special character escaping — CRITICAL

Always escape these characters when they appear in user content:

| Raw | LaTeX |
|-----|-------|
| `%` | `\%` |
| `&` | `\&` |
| `#` | `\#` |
| `_` | `\_` |
| `$` | `\$` |
| `~` | `\textasciitilde{}` |
| `^` | `\textasciicircum{}` |
| `{` | `\{` |
| `}` | `\}` |
| `\` | `\textbackslash{}` |
| `–` (en dash) | `--` |
| `—` (em dash) | `---` |
| `×` | `$\times$` |
| `±` | `$\pm$` |
| `>` `<` inside text | `$>$` `$<$` |

Superscripts: use `\textsuperscript{st}` etc. for 1st, 2nd, 3rd.
Math inline: wrap in `$...$` (e.g. `$p$-adic`, `$\pm$0.11\%`).

### 4c. Compile

```bash
cd /tmp && pdflatex -interaction=nonstopmode resume.tex 2>&1
```

Run **twice** to resolve internal references. Check the output line for `(1 page,` — if it says
2 pages, prune content (see role_tailoring.md → Content Pruning section) and recompile.

Save the final PDF to `/mnt/user-data/outputs/[FirstName]_Resume.pdf` and call `present_files`.

---

## Step 5 — Iterate

After presenting the PDF, invite feedback:

> "Here's your resume! Let me know if you'd like to tweak anything — wording, section order, add/remove items, or adjust any bullet points for a specific job posting."

Common iteration requests and how to handle them:
- **"Make bullet X more concise"** — trim to the single most impactful clause; keep the Result, compress the Challenge.
- **"This bullet doesn't sound right"** — check if CAR structure is intact; often the issue is a missing Challenge (reads like a task) or a missing Result (reads like an activity).
- **"Tailor this for [specific company]"** — if they haven't gone through Path C yet, route them there: run the full recruiter analysis first, then rewrite. Don't do a shallow keyword swap without the audit.
- **"Add [new project/experience]"** — ask for Challenge context, specific actions, and Result metrics before drafting; don't generate until you have all three CAR components.
- **"It's 2 pages"** — prune per the Content Pruning guide in role_tailoring.md.
- **"Change the format"** — the Jake Ryan template is fixed; explain this and offer to adjust spacing or font size (10pt is the minimum; never go below).

For each iteration: edit the `.tex` file, recompile, overwrite the output PDF, re-present.

---

## Quality Checklist

Before presenting the PDF, verify:

- [ ] Compiles to exactly **1 page**
- [ ] No LaTeX errors (only warnings are acceptable)
- [ ] All special characters properly escaped
- [ ] Every Experience and Projects bullet follows **CAR structure** (Challenge + Action + Result in one sentence)
- [ ] Every bullet starts with an **active verb**
- [ ] At least **50% of bullets** contain a quantified metric; all others are flagged for the user
- [ ] No bullet is missing its **Result** — activity-only bullets are rewritten or cut
- [ ] Bullet vocabulary matches the **target role** (checked against role_tailoring.md)
- [ ] No bullet wraps beyond **2 lines**
- [ ] Contact line includes all provided links (LinkedIn, GitHub, email, phone)
- [ ] Section order follows the recommendation in role_tailoring.md for this role
- [ ] Skills section is dense but scannable — no redundant entries
- [ ] **(Path C only)** All 5 missing keywords from recruiter analysis are now present in the resume
- [ ] **(Path C only)** All 3 red flags from recruiter analysis have been addressed or removed
- [ ] **(Path C only)** No information has been fabricated — every addition is grounded in the user's stated background

---

## Notes

- **Never fabricate information.** If a metric or detail is missing, either ask the user or write
  the bullet without the number. A bullet without a metric is better than a made-up one.
- **Font size**: default is 10pt. Use 11pt only if the content is sparse (< 60% of a page at 10pt).
  Never go below 10pt.
- **Do not change the template's visual design** (margins, fonts, titlerule, column layout). The
  Jake Ryan format is intentionally fixed — its value is in its universality.
- **GitHub username**: if not provided, omit the GitHub field rather than guessing.
- **GPA**: include only if ≥ 3.5/4.0 (or local equivalent). If the user provides it and it's below
  this, gently note the convention and let them decide.
