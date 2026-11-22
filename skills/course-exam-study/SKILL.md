---
name: "course-exam-study"
description: "Turn a course's lecture slides, tutorials and solutions into textbook-style notes and a LaTeX cheat sheet (PDFs), and/or exam-format practice tests (chapterwise mastery, full-syllabus, or study-then-test)."
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
3. **Notes + cheat sheet**: two styled PDFs (textbook-style notes and a LaTeX cheat sheet sized to the allowed help sheet), no tests.
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
- If worked solutions are handwritten or images without extractable text, solve those problems yourself and verify every answer with code (two independent methods where possible) instead of rendering many images.
- For anything computational (schedules, traces, tables, numeric answers), write a small script or simulator and check it reproduces **every** lecture and tutorial answer exactly. If it disagrees with an official answer, find out why (usually a tie-breaking or convention difference, or a slide typo), adopt the course's convention, and record it in the digest.

---

## Mode 3: Notes PDF + cheat sheet

### What the notes must feel like

A self-contained **textbook**, not a commentary on the course materials.

- **No source references.** Never write "Tutorial 7 Problem 3", "Lecture 5", "slide 12", "from the tutorial", "the lecturer's example", "this array from Tut 3", or mention slide typos, errata or disagreements with official answers. Restate every piece of data in full where it is used. Report errata and disagreements to the user **in chat** when delivering, never in the PDF.
- **Practice problems become examples.** Every tutorial problem, lecture example and past-paper question in scope becomes an `example` with a short descriptive title ("Unit tangent of a cycloid", "Independent Set ≤p Vertex Cover"), never a source label.
- **Explain every term.** Define each term the first time it is used, in a `definition` (or inline in one clause for minor terms). Say what every symbol and variable means (e.g. what n and k are in O(n + k)). Don't assume prior course vocabulary.
- **Teach, don't list.** For each concept give, where they help: an `intuition` box with an analogy or plain-language picture, a small concrete example, and a figure.
- **Processes step by step.** Every algorithm, method or proof technique gets a `method` box with numbered steps, then at least one `example` that **traces it on concrete data**, with labelled steps ("Step 1 (derivative)") and a trace table where state changes (arrays, queues, distances, partial sums).
- **Common errors.** Put typical mistakes, misconceptions, true/false traps and "why the obvious approach fails" in `pitfall` boxes next to the concept they concern.
- **Not wordy.** Short paragraphs (≤ 4 lines), one idea each. Prefer a formula, table or numbered list over a long sentence. Solutions show every step, but each step is one or two lines.

### Environment system (use consistently)

Write each environment as a pandoc fenced div. The build script numbers them per chapter (Definition 1.1, Example 1.2, Theorem 1.3, …, one shared counter) and adds the label line; figures are numbered separately (Figure 1.1, …). Don't write numbers or labels yourself.

| Class | Use for | Colour |
|---|---|---|
| `definition` | every new term or notation | lavender, purple bar |
| `theorem`, `lemma`, `proposition`, `corollary` | results to remember; follow with a `proof` div when the proof is examinable or instructive | light blue, blue bar |
| `proof` | proofs (inline "Proof." label, ends with ∎) | no box |
| `method` | numbered procedure / algorithm / recipe | warm grey bar |
| `example` | solved problems; start with **Problem.**, then **Step k (what).** lines, end with the answer in bold | green bar |
| `intuition` | analogies, plain-language pictures | pink bar |
| `pitfall` | common errors, misconceptions, traps (label "Common error") | amber bar |
| `remark` | short asides (unboxed) | none |
| `summary` | chapter-end bullet summary (exactly one per chapter, last) | lavender, full border |

Syntax: `::: {.definition name="Regular parametrization"}` … `:::` (the `name` becomes the italic title in brackets; omit it if there is no natural name). Plain `::: intuition` works for untitled ones. Never start a div with a heading. Don't nest environments. In proofs, don't start a paragraph with "(1)" (pandoc turns it into a list); write **Part 1.** instead.

Chapter layout: `# Chapter title` → one `::: lead` paragraph (what the chapter covers and what the exam asks) → `##` sections in teaching order → `summary`. Use `###` only for real sub-sections. Final optional chapter: an appendix of course conventions (notation, tie-breaking rules, orientation conventions) and the exam format, with no errata or source talk.

### Maths, figures, tables

- **Maths in LaTeX**: `$…$` inline, `$$…$$` on its own line for display. It is rendered to KaTeX HTML at build time (no JavaScript at print time). Built-in macros: `\R \N \Z \Q`, `\vb{x}` (bold vector), `\norm{x}`, `\abs{x}`, `\grad`, `\curl`, `\divg`, `\poly`. Use LaTeX for every formula, variable and set; plain text only for words. Don't use Unicode symbols outside `$…$` if DejaVu Serif lacks them (e.g. ✓, ∮): write `$\checkmark$`, `$\oint$`.
- **Figures**: real SVGs generated by a Python script, wherever a picture explains better than words (graphs, trees, arrays, timelines, regions of integration, state diagrams, before/after of an algorithm step). Insert as `![caption](fig/x.svg){width=NN%}`; the caption has no number (added automatically). Style: 680px-wide viewBox; DejaVu Sans 10.5–12.5px labels (escape `<`, `>`, `&` in SVG text, and validate every SVG with an XML parser); rounded rects (rx 4) with 0.6–0.8px #555 strokes; category [fill, text] pairs in this order: [#CECBF6,#3C3489] purple, [#9FE1CB,#085041] teal, [#F5C4B3,#712B13] coral, [#FAC775,#633806] amber, [#B5D4F4,#0C447C] blue, [#F4C0D1,#72243E] pink, [#C0DD97,#27500A] green, [#D3D1C7,#444441] grey; arrows/lines #444, errors #A32D2D, highlights #D85A30 (orange) and #378ADD (blue).
- **Tables** for comparisons, classifications and algorithm traces (pipe tables; LaTeX allowed in cells).

### Cheat sheet

- **Purpose**: the student copies or prints it onto the allowed help sheet, so it must be **readable at a glance**, not word soup.
- **Layout**: A4 **landscape, 3 columns** (`cheat.css`) by default. If the material is mostly long formulas or wide tables and a quick test build shows portrait 2-column is denser, switch `@page` to portrait and `column-count: 2`. Fit the page limit the user has (default 2 pages = one double-sided sheet), and **fill it with a readable font (8.5–9.5pt)** rather than leaving a page half-empty at tiny type. Rebuild and adjust font size until the content fills, but does not exceed, the limit.
- **Structure**: one `## N · Topic` strip per chapter; `###` sub-headings inside long topics; one fact per bullet (aim ≤ 1.5 lines), bold keyword first ("**Polar**: $dA = r\,dr\,d\theta$"); numbered lists for method skeletons; small tables for classifications/comparisons; `<div class="box">` (blank lines inside) for the few must-know theorems; `::: trap` for the classic exam traps. Key formulas go on their own `$$…$$` line; never let a long formula wrap mid-sentence.
- **Content**: every definition, formula, theorem condition, method skeleton, "which one is it" fact and recurring numeric result likely to be examined. No prose paragraphs, no source references.

### Build

- One-time setup in the build directory: `npm init -y && npm install katex@0.16`, then write the four bundled files below **exactly as given** (`build.py`, `katexify.js`, `notes.css`, `cheat.css`). Don't add other CSS or cover designs.
- Sources: notes in `md/NN.md` (two-digit chapter files), cheat sheet in `md/cheat.md`, figures in `fig/`.
- Build: `python3 build.py notes "<CODE> <NAME>"` (all chapters), `python3 build.py notes "<CODE> <NAME>" 03` (one chapter, keeps its real number; useful for previews), `python3 build.py cheat "<CODE> <NAME>"`.
- Pipeline: pandoc (`markdown+fenced_divs+pipe_tables+raw_html+implicit_figures+tex_math_dollars`, `--mathjax` placeholders, TOC depth 2, numbered sections) → `build.py` numbers environments and figures → `katexify.js` renders all maths with KaTeX (fails loudly on bad LaTeX) → headless Chrome prints to PDF.

### Verify, then deliver

- `katexify.js` must report no errors, and `grep -c 'class="math' <file>.html` must be 0 (all maths rendered).
- `pdffonts`: only DejaVu and KaTeX fonts. If another font appears, find the glyph (check each non-ASCII character against DejaVu with `fc-list :charset=…`) and replace it.
- Render pages to low-resolution contact sheets and check: environment colours and labels correct and numbered per chapter; no raw `$`, `\(` or LaTeX visible; figures load and are numbered; no overflowing formulas or tables; no near-empty pages; no source references (grep the Markdown for "tutorial", "lecture", "slide", "tut ", "problem set"). Zoom into single pages only where something looks wrong. Fix and rebuild.
- Send both PDFs with SendUserFile and a one-line summary of each. Report in chat (not in the PDFs) anything not covered, official answers you disagreed with, and slide errors found. If the user connected a folder, offer to save the PDFs into it.

## Modes 1, 2 and 4: Testing

### Teaching each chapter (mode 4 only)

Teaching quality does not drop under time pressure: a short deadline changes the pace, never the depth. No one-sentence summaries in place of explanations.

- **Define before use.** Define every term and symbol the first time it appears (e.g. what n, m and k mean in O(n + k)), in the lecture's terminology.
- **Explain how, then show it.** For every algorithm: what problem it solves, the idea in plain words, the lecture's pseudocode, then a **step-by-step trace on one concrete example** as a table (one row per step: state before, action, state after), then why its running time is what it is (derive it, don't just state it).
- **One running example per unit.** Draw it once (inline SVG diagram) and reuse it across sections so the user builds on one picture; add a second small drawing only when a concept needs it (e.g. one graph showing all four DFS edge types).
- **Generate every trace with code** and check it against the official lecture/tutorial answers before showing it; never hand-compute a trace you present.
- **Every tutorial question fully worked**: restate it, give the idea, the algorithm, a small worked example, and the complexity.
- **Proof ideas**: for each theorem the exam can ask (e.g. cycle ⇔ back edge, cut property), give the 2–4 line argument.
- **Deliver teaching text with SendUserMessage** (sectioned with headers, tables and code blocks), interleaved with the diagram widgets in reading order; text written between tool calls is otherwise summarised and the user never sees it.
- Build an interactive simulator for every algorithmic topic and verify it reproduces every lecture and tutorial answer exactly before showing it; say it has been verified.
- End with a "high-yield facts" list (each fact with its reason), then the chapter test.
- If the user picks a unit order, follow it.

**Precompute while the user studies.** When the user asks, or when the session is long, prepare the next units in the background (one forked agent per unit) while the user works on the current one: teaching sections as markdown files, diagram/simulator/test widget HTML, and the verified answer key, saved under a per-unit folder. When the user submits a test, grade it from the stored key and present the next unit from the prepared files.

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

Write these four files into the build directory before building (mode 3), after `npm install katex@0.16`. Do not modify them.

### build.py

```python
import re, subprocess, sys, pathlib, html as H
B = pathlib.Path(__file__).parent
KATEX_CSS = B / 'node_modules' / 'katex' / 'dist' / 'katex.min.css'

# Textbook environments: class -> (label, numbered)
ENVS = {'definition': ('Definition', True), 'theorem': ('Theorem', True), 'lemma': ('Lemma', True),
        'proposition': ('Proposition', True), 'corollary': ('Corollary', True), 'example': ('Example', True),
        'method': ('Method', True), 'intuition': ('Intuition', False), 'pitfall': ('Common error', False),
        'remark': ('Remark', False), 'summary': ('Chapter summary', False), 'proof': ('Proof', False)}

def number(html, chapter_offset=0):
    """Number environments and figures per chapter (h1): Definition 1.1, Example 1.2, ..., Figure 1.1."""
    out, pos, ch, k, fig = [], 0, chapter_offset, 0, 0
    pat = re.compile(r'<h1(?! class="title")[^>]*>|<div class="(%s)"([^>]*)>|<figcaption[^>]*>' % '|'.join(ENVS))
    for m in pat.finditer(html):
        out.append(html[pos:m.end()]); pos = m.end()
        t = m.group(0)
        if t.startswith('<h1'):
            ch += 1; k = 0; fig = 0
        elif t.startswith('<figcaption'):
            fig += 1; out.append('<span class="fignum">Figure %d.%d</span> ' % (ch, fig))
        else:
            cls, attrs = m.group(1), m.group(2)
            label, numbered = ENVS[cls]
            nm = re.search(r'(?:data-)?name="([^"]*)"', attrs)
            head = label
            if numbered:
                k += 1; head += ' %d.%d' % (ch, k)
            if cls == 'proof':   # put "Proof." inline at the start of the first paragraph
                lead = re.match(r'\s*<p>', html[pos:])
                if lead:
                    out.append(html[pos:pos + lead.end()]); pos += lead.end()
                out.append('<span class="envname">Proof.</span> ')
                continue
            name = (' <span class="envtitle">(%s)</span>' % nm.group(1)) if nm else ''
            out.append('<p class="envhead"><span class="envname">%s</span>%s</p>' % (head, name))
    out.append(html[pos:])
    return ''.join(out)

def postprocess(html, chapter_offset=0):
    html = re.sub(r'<section id="([^"]*)" class="(%s)"([^>]*)>(.*?)</section>' % '|'.join(ENVS), r'<div class="\2" id="\1"\3>\4</div>', html, flags=re.S)
    html = re.sub(r'<colgroup>.*?</colgroup>', '', html, flags=re.S)
    return number(html, chapter_offset)

def build(mdfiles, out, css, title, subtitle, toc=True, chapter_offset=0):
    src = ''
    for f in mdfiles:
        src += (B / f).read_text() + '\n\n'
    (B / 'tmp.md').write_text(src)
    h = B / (out + '.html')
    args = ['pandoc', str(B / 'tmp.md'), '-f', 'markdown+fenced_divs+pipe_tables+raw_html+implicit_figures+tex_math_dollars',
            '-t', 'html5', '-s', '--mathjax', '--css', str(KATEX_CSS), '--css', str(B / css),
            '--metadata', 'title=' + title, '--metadata', 'subtitle=' + subtitle, '-o', str(h)]
    if toc: args += ['--toc', '--toc-depth=2', '--number-sections']
    if chapter_offset: args += ['--number-offset=%d' % chapter_offset]
    subprocess.run(args, check=True)
    t = h.read_text()
    t = postprocess(t, chapter_offset) if toc else re.sub(r'<colgroup>.*?</colgroup>', '', t, flags=re.S)
    h.write_text(t)
    subprocess.run(['node', str(B / 'katexify.js'), str(h)], check=True)
    subprocess.run(['google-chrome', '--headless=new', '--no-sandbox', '--disable-gpu', '--no-pdf-header-footer',
                    '--print-to-pdf=' + str(B / (out + '.pdf')), 'file://' + str(h)], check=True, capture_output=True)
    print('built', out)

if __name__ == '__main__':
    # usage: python3 build.py notes "MH2100 Calculus III" [NN ...]   (NN = only these chapter files, e.g. 01 09)
    #        python3 build.py cheat "MH2100 Calculus III"
    which = sys.argv[1]; course = sys.argv[2] if len(sys.argv) > 2 else 'Course'
    only = sys.argv[3:]
    slug = course.split()[0]
    if which == 'notes':
        files = sorted(str(p.relative_to(B)) for p in (B / 'md').glob('[0-9][0-9].md') if not only or p.stem in only)
        offset = int(pathlib.Path(files[0]).stem) - 1 if only else 0   # a single-chapter build keeps its real number
        suffix = ('_Ch' + '_'.join(only)) if only else ''
        build(files, slug + '_Notes' + suffix, 'notes.css', course, 'Course notes with worked examples', chapter_offset=offset)
    else:
        build(['md/cheat.md'], slug + '_CheatSheet', 'cheat.css', course, 'Formula and concept sheet', toc=False)
```

### katexify.js

```javascript
// Replace pandoc's MathJax-style math spans with server-rendered KaTeX HTML.
// usage: node katexify.js file.html   (edits in place)
const fs = require('fs'), path = require('path');
const katex = require(path.join(__dirname, 'node_modules', 'katex'));
const f = process.argv[2];
let html = fs.readFileSync(f, 'utf8');
const unesc = s => s.replace(/&lt;/g, '<').replace(/&gt;/g, '>').replace(/&quot;/g, '"').replace(/&#39;/g, "'").replace(/&amp;/g, '&');
const macros = { '\\R': '\\mathbb{R}', '\\N': '\\mathbb{N}', '\\Z': '\\mathbb{Z}', '\\Q': '\\mathbb{Q}',
  '\\vb': '\\mathbf{#1}', '\\norm': '\\left\\lVert #1 \\right\\rVert', '\\abs': '\\left\\lvert #1 \\right\\rvert',
  '\\dd': '\\,\\mathrm{d}#1', '\\pd': '\\frac{\\partial #1}{\\partial #2}', '\\grad': '\\nabla', '\\curl': '\\operatorname{curl}',
  '\\divg': '\\operatorname{div}', '\\poly': '\\operatorname{poly}' };
let errors = 0;
html = html.replace(/<span\s+class="math (inline|display)">\\[\(\[]([\s\S]*?)\\[\)\]]<\/span>/g, (m, kind, tex) => {
  try { return katex.renderToString(unesc(tex), { displayMode: kind === 'display', throwOnError: true, macros, strict: 'ignore' }); }
  catch (e) { errors++; console.error('KaTeX error:', e.message.split('\n')[0], '| in:', unesc(tex).slice(0, 80));
    return katex.renderToString(unesc(tex), { displayMode: kind === 'display', throwOnError: false, macros, strict: 'ignore' }); }
});
html = html.replace(/<script[^>]*mathjax[\s\S]*?<\/script>/gi, '');
fs.writeFileSync(f, html);
if (errors) { console.error(errors + ' math error(s)'); process.exitCode = 1; }
```

### notes.css

```css
@page { size: A4; margin: 18mm 18mm 18mm 18mm; @bottom-center { content: counter(page); font-family: "DejaVu Sans"; font-size: 8pt; color: #888; } }
html { font-family: "DejaVu Serif", Georgia, serif; font-size: 10.3pt; line-height: 1.55; color: #1d1d1b; }
body { max-width: none; margin: 0; padding: 0; }
/* cover + contents */
header#title-block-header { text-align: center; padding: 75mm 0 30mm; page-break-after: always; }
header .title { font-family: "DejaVu Sans", sans-serif; font-size: 30pt; font-weight: bold; color: #26215C; margin: 0; }
header .subtitle { font-family: "DejaVu Sans", sans-serif; font-size: 13pt; color: #534AB7; margin-top: 10px; }
nav#TOC { page-break-after: always; font-family: "DejaVu Sans", sans-serif; }
nav#TOC::before { content: "Contents"; display: block; font-size: 20pt; font-weight: bold; color: #26215C; margin-bottom: 10px; }
nav#TOC ul { list-style: none; padding-left: 0; } nav#TOC ul ul { padding-left: 22px; font-size: 9.5pt; color: #444; }
nav#TOC > ul > li { margin-top: 8px; font-weight: bold; } nav#TOC ul ul li { font-weight: normal; margin: 1px 0; }
nav#TOC a { color: inherit; text-decoration: none; }
/* headings */
h1 { font-family: "DejaVu Sans", sans-serif; color: #26215C; font-size: 21pt; border-bottom: 3px solid #534AB7; padding-bottom: 6px; margin: 0 0 8px; page-break-before: always; }
h2 { font-family: "DejaVu Sans", sans-serif; color: #3C3489; font-size: 13.5pt; margin: 22px 0 6px; page-break-after: avoid; }
h3 { font-family: "DejaVu Sans", sans-serif; color: #534AB7; font-size: 11pt; margin: 14px 0 4px; page-break-after: avoid; }
.header-section-number { color: #7F77DD; margin-right: 8px; }
/* text */
p { margin: 5px 0 7px; text-align: justify; hyphens: auto; }
ul, ol { margin: 4px 0 8px; padding-left: 22px; } li { margin: 2px 0; }
strong { color: #111; } em { color: #26215C; }
a { color: #3C3489; }
code { font-family: "DejaVu Sans Mono", monospace; font-size: 8.8pt; background: #f2f1ec; padding: 0 3px; border-radius: 3px; }
pre { background: #f5f4ef; border-left: 3px solid #7F77DD; padding: 7px 10px; font-size: 8.6pt; line-height: 1.4; white-space: pre-wrap; page-break-inside: avoid; border-radius: 0 4px 4px 0; }
pre code { background: none; padding: 0; }
.katex { font-size: 1.08em; } .katex-display { margin: 7px 0; overflow: hidden; }
/* chapter opener */
div.lead { font-size: 10.6pt; color: #3C3489; font-style: italic; border-left: 4px solid #AFA9EC; padding: 4px 12px; margin: 6px 0 14px; }
/* textbook environments: one colour per kind, label line on top */
div.definition, div.theorem, div.lemma, div.proposition, div.corollary, div.example, div.method, div.intuition, div.pitfall, div.summary
  { border-left: 4px solid; border-radius: 0 6px 6px 0; padding: 5px 12px 6px; margin: 12px 0; page-break-inside: avoid; }
div.example { page-break-inside: auto; }
p.envhead { margin: 0 0 3px; text-align: left; font-family: "DejaVu Sans", sans-serif; font-size: 9.6pt; }
.envname { font-weight: bold; } .envtitle { font-weight: normal; font-style: italic; }
div.definition { background: #EEEDFE; border-color: #534AB7; } div.definition .envname { color: #3C3489; }
div.theorem, div.lemma, div.proposition, div.corollary { background: #EAF2FC; border-color: #378ADD; }
div.theorem .envname, div.lemma .envname, div.proposition .envname, div.corollary .envname { color: #0C447C; }
div.example { background: #E1F5EE; border-color: #1D9E75; } div.example .envname { color: #085041; }
div.method { background: #F4F3EE; border-color: #888780; } div.method .envname { color: #444441; }
div.intuition { background: #FBEFF4; border-color: #D4537E; } div.intuition .envname { color: #72243E; }
div.pitfall { background: #FAEEDA; border-color: #EF9F27; } div.pitfall .envname { color: #633806; }
div.summary { background: #EEEDFE; border: 1pt solid #AFA9EC; border-left: 4px solid #534AB7; }
div.summary .envname { color: #26215C; }
div.remark { margin: 8px 0; } div.remark p.envhead { display: inline; } div.remark .envname { color: #534AB7; }
div.proof { margin: 4px 0 10px; } div.proof .envname { font-style: italic; font-weight: bold; color: #0C447C; font-family: "DejaVu Sans", sans-serif; font-size: 9.6pt; }
div.proof > p:last-child::after { content: " ∎"; color: #0C447C; }
/* inside a worked example: "Solution" and "Step" labels */
div.example p strong:first-child { font-family: "DejaVu Sans", sans-serif; font-size: 9.4pt; color: #085041; }
div.example hr { border: none; border-top: 1px dashed #5DCAA5; margin: 6px 0; }
/* tables */
table { border-collapse: collapse; width: 100%; margin: 8px 0 12px; font-size: 9pt; page-break-inside: avoid; font-family: "DejaVu Sans", sans-serif; }
th { background: #EEEDFE; color: #26215C; text-align: left; padding: 5px 7px; border: 0.5pt solid #bbb; }
td { padding: 4px 7px; border: 0.5pt solid #ccc; vertical-align: top; }
div.definition table, div.example table, div.theorem table { background: #fff; }
/* figures */
figure { margin: 10px 0 14px; text-align: center; page-break-inside: avoid; }
figure img { max-width: 100%; }
figcaption { font-family: "DejaVu Sans", sans-serif; font-size: 8.6pt; color: #555; margin-top: 3px; }
.fignum { font-weight: bold; color: #3C3489; }
```

### cheat.css

```css
@page { size: A4 landscape; margin: 7mm 7mm 8mm 7mm; @bottom-right { content: counter(page) "/" counter(pages); font-family: "DejaVu Sans"; font-size: 6.5pt; color: #888; } }
html { font-family: "DejaVu Sans", Arial, sans-serif; font-size: 9.3pt; line-height: 1.38; color: #1a1a1a; }
body { margin: 0; padding: 0; max-width: none; column-count: 3; column-gap: 5mm; column-rule: 0.5pt solid #d8d6ef; }
header#title-block-header { column-span: all; border-bottom: 1.5pt solid #534AB7; margin-bottom: 4px; padding-bottom: 2px; }
header .title { font-size: 13pt; color: #26215C; margin: 0; display: inline; }
header .subtitle { font-size: 8pt; color: #534AB7; display: inline; margin-left: 8px; }
/* one box per topic */
h2 { font-size: 10.4pt; color: #fff; background: #534AB7; padding: 1.5px 5px; margin: 5px 0 2px; border-radius: 2px; break-after: avoid; }
h3 { font-size: 9.4pt; color: #3C3489; margin: 3px 0 1px; break-after: avoid; border-bottom: 0.4pt solid #CECBF6; }
p { margin: 1px 0 2px; }
ul, ol { margin: 1px 0 3px; padding-left: 13px; } li { margin: 0 0 2px; break-inside: avoid; }
strong { color: #26215C; }
.katex { font-size: 1.0em; } .katex-display { margin: 2px 0; } .katex-display > .katex { font-size: 1.0em; }
table { border-collapse: collapse; width: 100%; margin: 2px 0 3px; break-inside: avoid; font-size: 8.8pt; }
th { background: #EEEDFE; color: #26215C; text-align: left; padding: 1px 3px; border: 0.4pt solid #c9c6ea; }
td { padding: 1px 3px; border: 0.4pt solid #ddd; vertical-align: top; }
div.box { background: #EEEDFE; border-left: 2.5pt solid #534AB7; padding: 1px 4px; margin: 2px 0; break-inside: avoid; }
div.trap { background: #FAEEDA; border-left: 2.5pt solid #EF9F27; padding: 1px 4px; margin: 2px 0; break-inside: avoid; }
div.trap::before { content: "Trap  "; font-weight: bold; color: #633806; }
code { font-family: "DejaVu Sans Mono", monospace; font-size: 6.7pt; background: #f0efea; padding: 0 2px; }
```