---
name: "course-exam-study"
description: "Turn a course's lecture slides, tutorials and solutions into styled notes + cheat sheet PDFs and/or exam-format practice tests (chapterwise mastery, full-syllabus, or study-then-test)."
---

# Course exam study kit

Use this skill when the user wants to revise a university course from its lecture slides, tutorial questions and tutorial solutions: generating textbook-style notes and a cheat sheet, and/or practising with tests that mirror the real exam.

Follow the steps **in order**. Do not start reading, teaching or building until steps 1–3 are done. Follow the **Context budget** rules throughout: course uploads are large, and the user may be on a small plan.

---

## Step 1: Get the files

Ask the user for the course files, then wait for them before doing anything else. Offer these ways to provide them, in this order of preference (keep the message short):

1. **Connect the course folder (Claude desktop app, best option).** In the desktop app, **+ → Add folder** and pick the course folder. Nothing is uploaded up front: you list the folder and stage only the files needed, when needed. This avoids the per-message upload limit *and* saves context. If remote-device tools (`mcp__remote-devices__*`) are present, you can also request access to a folder with `device_request_folder_access` after asking the user which folder it is.
2. **One zip file.** Put everything in a single `.zip` and upload it (it must be under the file-size limit). You extract it into its own directory.
3. **Merged PDFs.** Combine files, ideally one PDF per week or chapter (lecture + tutorial + solution), e.g. with Preview on Mac or ilovepdf. Per-chapter merges are better than one huge PDF, because only the current chapter needs reading.
4. **Several messages.** Upload in batches; you acknowledge each batch and wait until they say they're done.

Also tell them in one line that the files cost context once read (a zip of 40 PDFs costs the same as 40 PDFs), so:

- if they plan to go chapter by chapter (study-then-test or chapterwise tests), they can provide **just the first chapter's lecture, tutorial and solution now** and the rest when they reach each chapter (with a connected folder this is automatic);
- for notes or a full-syllabus test the whole course is needed, but they can leave out files that aren't examinable (e.g. a lecture marked out of scope) and duplicates (e.g. a live-lecture deck that repeats a recorded one).

The full list, when everything is wanted:

- all examinable lecture slides;
- all tutorial question sheets **and** their solutions;
- past papers or sample quiz/exam questions, if they have any;
- any revision/review lecture or announcement that states the examinable scope;
- the course code and name.

When files arrive (or the folder is connected), acknowledge them in one or two lines: count, and anything that looks missing, e.g. tutorials without solutions. Don't open them yet.

**Handling the sources:**

- *Connected folder:* list it with `device_list_dir` (recursive) to map files to chapters, then stage files with `device_stage_files` only when that chapter is being read. Never stage the whole folder at once.
- *Zip:* extract into a fresh directory of its own (e.g. `unzip -q course.zip -d src_zip/`), list the contents, and treat the extracted files exactly like uploads.
- *Merged PDFs:* use the per-page inventory (below) to find where each lecture, tutorial and solution starts, and work with page ranges (`pdftotext -f N -l M`).

## Step 2: Ask what they want to do

Use AskUserQuestion (single choice) with these four options:

1. **Chapterwise tests**: one exam-format test per chapter, no teaching; move on only at 100%.
2. **Full-syllabus test**: one mock exam covering the whole examinable scope.
3. **Notes + cheat sheet**: two styled PDFs (textbook notes and a 2–4 page cheat sheet), no tests.
4. **Study then test, chapter by chapter**: teach a chapter, test it, move on only at 100%.

In the same AskUserQuestion call (separate question), ask for the examinable scope if the files don't state it, e.g. "Lectures weeks 1–12, tutorials 2–13; Security lecture not examinable".

If they pick mode 2 or 3 but provided only part of the course, ask for the rest now.

## Step 3: If any testing option was picked (1, 2 or 4), ask for the test format

Do the cheap inventory pass (see "Reading the material") so you can confirm the chapter list, then ask these before writing any test. Use AskUserQuestion multiple-choice prompts where possible (max 4 questions per call; use two calls if needed) and plain text for the rest:

1. Exam components: how many papers or quizzes, their weights, durations and dates?
2. Question formats per component: MCQ (how many options? single or multiple answer?), true/false with justification, short answer, long-form calculation, code tracing, proofs, diagrams?
3. Roughly how many questions per component, and how many marks per question?
4. Open- or closed-book; calculators or formula sheets allowed?
5. Which chapters and tutorials are examinable, and is anything excluded? (skip if already known)
6. Are there past papers or sample questions? If so, ask the user to provide them and match their style exactly.
7. How strictly should written answers be marked: correct final answer only, or full method and justification as the official tutorial solutions show?
8. How much time until the exam, and how many hours per day? Use this to set the pace; warn early if the plan won't fit.

For mode 2 (full-syllabus test) also ask whether the mock should be timed to the real exam length and split into the real exam's components.

Then confirm the chapter list and order, and start the chosen mode.

---

## Reading the material (all modes)

### Context budget (follow strictly)

- **Never** `cat` or Read a whole PDF, or a whole extracted text file, into context in one go. Extract each PDF to a text file on disk first (`pdftotext`, then strip blank lines and repeated slide footers or navigation lines with `grep -v`/`sed`), and print only the part you need, one lecture or tutorial at a time.
- **Cheap inventory first:** page count, words per page, and the first line (title) of each page, for every file (or every page range of a merged PDF). Use it to map chapters, match tutorials to lectures, find slides marked "not examinable", and list low-text pages. Once a slide is confirmed non-examinable, don't read it further. Skip boilerplate (title, contents, "additional materials", and recap or learning-objective slides that only repeat earlier content).
- **Images only where needed:** render to images only pages with little extractable text (roughly under 40 words) that look like diagrams, worked solutions or tables. Use contact sheets (`pdftoppm` + `montage`) at about 50–60 dpi, 9 pages per sheet. Re-render a single page at higher resolution only when a detail is unreadable. Never re-read an image you have already read.
- **Digest file:** after each lecture or tutorial, write what matters into a terse digest file on disk (definitions, formulas, examples with their official answers, conventions, non-examinable items). Afterwards work from the digest, not the PDFs.
- **Just in time:** in modes 1 and 4, read (and, for a connected folder, stage) only the current chapter's lecture, tutorial and solution before teaching or testing it. Modes 2 and 3 need the whole course, but still read it chapter by chapter, writing the digest as you go.
- **Short tool output:** use `head`, `grep` and targeted line ranges instead of dumping files; don't echo back files you just wrote; write long documents straight into files rather than into chat.
- **Running low:** if context is clearly getting tight (e.g. on a smaller plan), say so, finish the current chapter, and suggest continuing in a fresh chat with the digest file (send it to the user) and the remaining chapters' files.

### What to extract

- Read every examinable page, under the rules above, before writing about or testing that chapter. Don't rely on text extraction alone for diagram-heavy pages.
- Match each tutorial (and its solution) to its lecture chapter.
- Record every slide marked "not examinable" and any scope statement from a review lecture. The review lecture's scope overrides the user's line if they conflict (tell them).
- For anything computational (schedules, traces, tables, numeric answers), write a small script or simulator and check it reproduces **every** lecture and tutorial answer exactly. If it disagrees with an official answer, find out why (usually a tie-breaking or convention difference, or a slide typo), adopt the course's convention, and record it in the digest.

---

## Mode 3: Notes PDF + cheat sheet

### Content

**Notes PDF (textbook-style, self-contained)**

- One chapter (H1) per lecture topic, in lecture order; H2 for sections; H3 for worked examples and subsections.
- Each chapter has:
  - a short italic introduction (`::: lead`) saying what the chapter covers and which skills are tested;
  - numbered sections that explain every examinable concept in full prose, using the lecture's own definitions and terminology, with tables for comparisons;
  - `::: key` boxes for must-memorise facts and formulas, and `::: warn` boxes for common traps (especially true/false statements from the tutorials) and for the chapter's list of non-examinable slides;
  - every tutorial question restated in full and solved as a worked example (`::: solve`, starting with an H3 title), with the method, every intermediate step, and the reasoning the official solution gives. Never refer to "the tutorial" without restating the question; the PDF must make sense without the tutorial sheets;
  - the lecture's own worked examples, reproduced with full working, also as `::: solve` boxes;
  - a short chapter summary at the end (`::: key`).
- End with an appendix listing conventions, tie-breaking rules and any slide errata, and one summarising non-examinable or supplementary material.
- Write each chapter's Markdown file straight to disk from the digest, one chapter at a time; don't paste chapter text into chat.

**Cheat sheet PDF**

- 2–4 pages, A4, two columns, built with `cheat.css`; title + subtitle via pandoc metadata; no TOC.
- One `## N · Chapter name` header per chapter (renders as a purple strip). Dense bullets with bold keywords only: **no tables, no prose paragraphs**.
- Include every definition, formula, rule of thumb, comparison and "which one is it" fact likely to appear in MCQs; the method skeleton for every calculation type (e.g. "Need = Max − Allocation; Work += Allocation"); the recurring numeric results from lecture and tutorial examples; and the course's tie-breaking conventions.

### Style and build (must match exactly)

- Write the three files in the "Bundled files" section below into the build directory **exactly as given**. Don't add your own CSS, colour scheme, cover design, part-divider pages or running headers.
- Pipeline: Markdown → pandoc (`markdown+fenced_divs+pipe_tables+raw_html+implicit_figures`, `-s`, `--toc --toc-depth=2 --number-sections`) → HTML → headless Chrome `--print-to-pdf`, via `build.py`. Notes sources go in `md/NN.md` (two-digit chapter files); the cheat sheet in `md/cheat.md`.
- Cover: only the pandoc title + subtitle ("<COURSE CODE> <COURSE NAME>" / "Complete chapter notes with solved examples").
- Class mapping (pandoc fenced divs / raw HTML):
  - intro → `::: lead` · must-memorise → `::: key` · traps → `::: warn` · solved examples → `::: solve`;
  - numeric tables → `<table class="num">` (build.py auto-tags tables whose cells are mostly numeric);
  - step-by-step trace tables → `<table class="pr">` with `class="new"` on cells that just changed and `class="flt"` on event/fault cells, plus a `<p class="verdict">` under each trace with the final result. Write these as HTML fragments in `frag/NAME.html` and insert them with `{{frag:NAME}}`.
- Pandoc gotcha: a fenced div whose first element is a heading becomes `<section class="solve">`, which the CSS doesn't style. `build.py` post-processes it back to `<div class="solve">`; check every solved example renders as a green box.
- Maths: no LaTeX/MathML. Write formulas as plain HTML with `<sup>`/`<sub>`; never output a literal `^` or `_`. Put each display formula on its own line (`<br>` between alternatives) so justified text doesn't stretch it.
- Fonts: DejaVu Serif body, DejaVu Sans headings and tables, DejaVu Sans Mono code (set by the CSS). No other fonts. Avoid glyphs DejaVu Serif lacks (e.g. ✗) so Chrome doesn't fall back to another font; check with `pdffonts`.
- Figures: generate real SVG figures wherever a picture explains better than words (timelines, state/sequence diagrams, Gantt charts, memory layouts, graphs, data structures, algorithm traces, plots), generated from the verification scripts where possible. Insert as `![Figure N.M: caption](fig/x.svg){width=NN%}` (short italic caption under each). Style:
  - 680px-wide viewBox; DejaVu Sans 11–12px labels; rounded rects (rx 4) with 0.6–0.8px #555 strokes;
  - category [fill, text] pairs, used in this order: [#CECBF6,#3C3489] purple, [#9FE1CB,#085041] teal, [#F5C4B3,#712B13] coral, [#FAC775,#633806] amber, [#B5D4F4,#0C447C] blue, [#F4C0D1,#72243E] pink, [#C0DD97,#27500A] green, [#D3D1C7,#444441] grey;
  - arrows/lines #444, errors/violations #A32D2D, highlight lines #D85A30 (orange) and #378ADD (blue).
- Build: `python3 build.py notes "<CODE> <NAME>"` and `python3 build.py cheat "<CODE> <NAME>"`.

### Verify, then deliver

- Render pages to low-resolution contact sheets and check against the spec: purple headings (#26215C / #3C3489 / #534AB7), lavender key boxes, green solved-example boxes, amber warning boxes, lavender table headers, figures in the palette, no broken or overflowing figures or tables, no literal `^`, no empty pages, only DejaVu fonts embedded. Zoom into single pages only where something looks wrong. Fix and rebuild until it matches.
- Send both PDFs with SendUserFile and a one-line summary of each, plus anything you couldn't cover, any official answers you disagreed with, and any slide errors found. If the user connected a folder, also offer to save the PDFs into it.

---

## Modes 1, 2 and 4: Testing

### Teaching each chapter (mode 4 only)

- Explain every examinable concept in detail, in the lecture's terminology, using analogies and examples.
- Walk through every tutorial question for that chapter as a fully solved example.
- Use inline visuals (diagrams, charts, interactive widgets) where they help. For every algorithmic topic, build an interactive simulator or calculator the user can use to check their hand-worked answers. First verify that it reproduces every lecture and tutorial answer exactly, and say it has been verified.
- End with a "high-yield facts" summary, then the chapter test.

### Pacing rules

- **Mode 1 (chapterwise tests) and mode 4 (study then test):** one test per chapter, in lecture order. The user moves on only after scoring 100%. Under 100%: give a new, focused retest on the missed topics (never the same questions again), and repeat until 100%. When a chapter is passed, ask for the next chapter's files if they haven't been provided yet (with a connected folder, just stage them).
- **Mode 2 (full-syllabus test):** one mock exam across the whole scope, matching the real exam's components, question counts, marks and (if requested) timing. After grading, offer a focused retest on the weakest topics.
- Keep a running list of the user's recurring mistakes (in the digest file) and bring them back into later tests and a final cumulative retest.
- If time is running short, say so plainly and offer options (e.g. a condensed pace for concept-heavy chapters, or retests deferred to the end), but let the user decide whether to relax the 100% rule.

### Test design (the most important part)

- Mirror the real exam: use the formats, question counts and marking style the user described, scaled to one chapter (modes 1 and 4) or to the full paper (mode 2).
- Cover every topic in scope. Include questions in the style of the tutorials but with new numbers and new scenarios, never the exact tutorial question.
- MCQ quality rules:
  - every option must be similar in length, detail and tone, so the correct one never stands out by being the longest or most precise;
  - distractors must be plausible: build them from real misconceptions, off-by-one errors, swapped definitions, and properties that belong to a different algorithm or concept in the same chapter;
  - spread the correct letters evenly across the options, with no patterns;
  - no "all of the above" or "none of the above" unless the real exam uses them.
- Short-answer and calculation questions must have a single, unambiguous answer. State every convention needed (tie-breaking, initial state, units). Compute the answer key beforehand with code, and check that the question has exactly one correct answer.
- Present the test as an interactive widget (Visualizer `show_widget`): radio buttons for MCQs and text boxes for written answers, which won't submit until everything is answered. On submit, send the answers back to chat (`sendPrompt`). Don't put the answer key in the widget. Keep the widget code compact (questions in a data array, one shared render loop). Tell the user not to use the simulators on the test questions before submitting. If no widget tool is available, give the test as numbered plain text and ask for answers in one message.

### Grading

- Grade strictly against the chosen marking style. For each wrong or incomplete answer, give the correct answer, why the choice was wrong, and the exact method or justification the marker expects. Keep feedback on correct answers to a tick.
- For written answers, flag missing steps or justifications even when the final answer is right, and say whether a strict marker would deduct marks.
- Report the score, then apply the pacing rules above.

---

## Bundled files

Write these three files into the build directory before building (mode 3). Do not modify `notes.css` or `cheat.css`.

### notes.css

```css
@page { size: A4; margin: 18mm 17mm 18mm 17mm; @bottom-center { content: counter(page); } }
html { font-family: "DejaVu Serif", Georgia, serif; font-size: 10.4pt; line-height: 1.5; color: #1d1d1b; }
body { max-width: none; margin: 0; padding: 0; }
header#title-block-header { text-align: center; padding: 70mm 0 30mm; page-break-after: always; }
header .title { font-family: "DejaVu Sans", sans-serif; font-size: 30pt; font-weight: bold; color: #26215C; margin: 0; }
header .subtitle { font-family: "DejaVu Sans", sans-serif; font-size: 13pt; color: #534AB7; margin-top: 10px; }
nav#TOC { page-break-after: always; font-family: "DejaVu Sans", sans-serif; }
nav#TOC::before { content: "Contents"; display: block; font-size: 20pt; font-weight: bold; color: #26215C; margin-bottom: 10px; }
nav#TOC ul { list-style: none; padding-left: 0; } nav#TOC ul ul { padding-left: 22px; font-size: 9.5pt; color: #444; }
nav#TOC > ul > li { margin-top: 8px; font-weight: bold; } nav#TOC ul ul li { font-weight: normal; margin: 1px 0; }
nav#TOC a { color: inherit; text-decoration: none; }
h1 { font-family: "DejaVu Sans", sans-serif; color: #26215C; font-size: 21pt; border-bottom: 3px solid #534AB7; padding-bottom: 6px; margin-top: 0; page-break-before: always; }
h2 { font-family: "DejaVu Sans", sans-serif; color: #3C3489; font-size: 14pt; margin-top: 20px; margin-bottom: 6px; page-break-after: avoid; }
h3 { font-family: "DejaVu Sans", sans-serif; color: #534AB7; font-size: 11.5pt; margin-top: 14px; margin-bottom: 4px; page-break-after: avoid; }
.header-section-number { color: #7F77DD; margin-right: 8px; }
p { margin: 5px 0 7px; text-align: justify; hyphens: auto; }
ul, ol { margin: 4px 0 8px; padding-left: 22px; } li { margin: 2px 0; }
code { font-family: "DejaVu Sans Mono", monospace; font-size: 8.8pt; background: #f2f1ec; padding: 0 3px; border-radius: 3px; }
pre { background: #f5f4ef; border-left: 3px solid #7F77DD; padding: 7px 10px; font-size: 8.6pt; line-height: 1.38; overflow: hidden; white-space: pre-wrap; page-break-inside: avoid; border-radius: 0 4px 4px 0; }
pre code { background: none; padding: 0; }
table { border-collapse: collapse; width: 100%; margin: 8px 0 12px; font-size: 9pt; page-break-inside: avoid; font-family: "DejaVu Sans", sans-serif; }
th { background: #EEEDFE; color: #26215C; text-align: left; padding: 5px 7px; border: 0.5pt solid #bbb; }
td { padding: 4px 7px; border: 0.5pt solid #ccc; vertical-align: top; }
table.num td, table.num th { text-align: center; }
table.pr { width: auto; font-family: "DejaVu Sans Mono", monospace; font-size: 8.5pt; }
table.pr td, table.pr th { text-align: center; padding: 2px 5px; min-width: 18px; }
table.pr td.new { background: #B5D4F4; font-weight: bold; } table.pr td.hand { background: #FAC775; } table.pr td.flt { color: #A32D2D; font-weight: bold; }
p.verdict { font-family: "DejaVu Sans", sans-serif; font-weight: bold; color: #0F6E56; margin-top: -4px; }
figure { margin: 10px 0 14px; text-align: center; page-break-inside: avoid; }
figure img { max-width: 100%; }
figcaption { font-family: "DejaVu Sans", sans-serif; font-size: 8.6pt; color: #555; margin-top: 3px; font-style: italic; }
blockquote { margin: 8px 0; padding: 6px 12px; background: #f6f6f3; border-left: 3px solid #B4B2A9; color: #333; page-break-inside: avoid; }
blockquote p { margin: 3px 0; }
div.lead { font-size: 10.8pt; color: #3C3489; font-style: italic; border-left: 4px solid #AFA9EC; padding: 4px 12px; margin: 6px 0 12px; }
div.key { background: #EEEDFE; border: 1pt solid #AFA9EC; border-radius: 6px; padding: 6px 12px; margin: 10px 0; page-break-inside: avoid; }
div.warn { background: #FAEEDA; border: 1pt solid #EF9F27; border-radius: 6px; padding: 6px 12px; margin: 10px 0; page-break-inside: avoid; }
div.key p:first-child, div.warn p:first-child { margin-top: 2px; }
div.solve { background: #E1F5EE; border: 1pt solid #5DCAA5; border-radius: 6px; padding: 6px 12px; margin: 10px 0; }
strong { color: #111; }
a { color: #3C3489; }
```

### cheat.css

```css
@page { size: A4; margin: 9mm 9mm 10mm 9mm; @bottom-center { content: counter(page); font-size: 7pt; } }
html { font-family: "DejaVu Sans", Arial, sans-serif; font-size: 7.25pt; line-height: 1.32; color: #1a1a1a; }
body { margin: 0; padding: 0; max-width: none; column-count: 2; column-gap: 7mm; column-rule: 0.5pt solid #ccc; }
header#title-block-header { column-span: all; text-align: left; border-bottom: 2pt solid #534AB7; margin-bottom: 5px; padding-bottom: 3px; }
header .title { font-size: 14pt; color: #26215C; margin: 0; display: inline; }
header .subtitle { font-size: 8.5pt; color: #534AB7; display: inline; margin-left: 10px; }
h2 { font-size: 9.5pt; color: #fff; background: #534AB7; padding: 2px 6px; margin: 6px 0 3px; border-radius: 3px; break-after: avoid; }
ul { margin: 0 0 3px; padding-left: 11px; } li { margin: 0 0 1.6px; break-inside: avoid; }
strong { color: #26215C; }
code { font-family: "DejaVu Sans Mono", monospace; font-size: 6.8pt; background: #f0efea; padding: 0 2px; }
```

### build.py

```python
import re, subprocess, sys, pathlib
B = pathlib.Path(__file__).parent

def postprocess(html):
    # numeric tables -> class="num"; trace tables come from frag/ already as class="pr"
    def tag_table(m):
        t = m.group(0)
        if 'class="pr"' in t: return t
        cells = re.findall(r'<td[^>]*>(.*?)</td>', t, flags=re.S)
        plain = [re.sub(r'<[^>]+>', '', c).strip() for c in cells]
        num = sum(1 for c in plain if re.fullmatch(r'[\d.,/ ()×%–−+=≈µsBkKMbpsms∞✓~-]*', c) and re.search(r'\d|∞|✓', c))
        if cells and num / len(cells) >= 0.6:
            t = t.replace('<table', '<table class="num"', 1)
        return t
    html = re.sub(r'<table.*?</table>', tag_table, html, flags=re.S)
    # pandoc turns a fenced div that starts with a heading into <section class="solve">; restore the div
    html = re.sub(r'<section id="([^"]*)" class="solve"[^>]*>(.*?)</section>', r'<div class="solve" id="\1">\2</div>', html, flags=re.S)
    html = re.sub(r'<colgroup>.*?</colgroup>', '', html, flags=re.S)   # let the stylesheet size columns
    return html

def build(mdfiles, out, css, title, subtitle, toc=True):
    src = ''
    for f in mdfiles:
        t = (B / f).read_text()
        t = re.sub(r'\{\{frag:(\w+)\}\}', lambda m: '\n```{=html}\n' + (B / 'frag' / (m.group(1) + '.html')).read_text() + '\n```\n', t)
        src += t + '\n\n'
    (B / 'tmp.md').write_text(src)
    args = ['pandoc', str(B / 'tmp.md'), '-f', 'markdown+fenced_divs+pipe_tables+raw_html+implicit_figures', '-t', 'html5', '-s',
            '--css', str(B / css), '--metadata', 'title=' + title, '--metadata', 'subtitle=' + subtitle, '-o', str(B / (out + '.html'))]
    if toc: args += ['--toc', '--toc-depth=2', '--number-sections']
    subprocess.run(args, check=True)
    h = B / (out + '.html'); h.write_text(postprocess(h.read_text()))
    subprocess.run(['google-chrome', '--headless=new', '--no-sandbox', '--disable-gpu', '--no-pdf-header-footer',
                    '--print-to-pdf=' + str(B / (out + '.pdf')), 'file://' + str(h)], check=True, capture_output=True)
    print('built', out)

if __name__ == '__main__':
    # usage: python3 build.py notes|cheat "SC2008 Computer Networks"
    which = sys.argv[1]
    course = sys.argv[2] if len(sys.argv) > 2 else 'Course'
    slug = course.split()[0]
    if which == 'notes':
        files = sorted(str(p.relative_to(B)) for p in (B / 'md').glob('[0-9][0-9].md'))
        build(files, slug + '_Notes', 'notes.css', course, 'Complete chapter notes with solved examples')
    else:
        build(['md/cheat.md'], slug + '_CheatSheet', 'cheat.css', course + ' — Cheat Sheet', 'Concepts and formulas for quick revision', toc=False)
```