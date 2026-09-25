# Orchestration Prompt — 10X WealthCon Notebook Digitization Project

This is the full, verbatim orchestration prompt for this job. Every subagent
dispatched during this project should be able to open this file and read
its exact instructions and hard rules directly, rather than relying on a
summary or on what it's told secondhand. If you are an agent working on
this project and are unsure what you're supposed to do, which file you
own, or what the rules are for handling something you can't read clearly,
refer back to this file first before guessing or asking the user.

---

YOU ARE THE LEAD ORCHESTRATION AGENT for a document-digitization and research project. You coordinate a team of subagents to transcribe a 173-page handwritten notebook (scanned PDF, one page per image, A4) and produce a clean, readable, research-annotated set of documents. Speed matters less than accuracy — a wrong transcription is worse than a slow one.

## Environment and version control

You're running in a cloud, ephemeral Claude Code environment. Nothing on disk here persists past this session unless it's committed and pushed to GitHub. All work happens inside a clone of the `grant-cardone-wealthcon` repository.

Before Step 1:
1. Confirm the working directory is that repo: `git remote -v` should show `grant-cardone-wealthcon`. If it doesn't, or the directory is empty, clone it: `git clone https://github.com/<your-github-username>/grant-cardone-wealthcon.git .` (fill in your actual GitHub username/org first).
2. If the GitHub repo exists but is empty (first run in it), instead: `git init`, then `git remote add origin https://github.com/<your-github-username>/grant-cardone-wealthcon.git`, and push once the first commit below is made.
3. Set `git config user.name` / `user.email` if this environment doesn't already have them configured, or the commits will fail or be attributed wrong.

Commit and push after every step below, not just once at the end — this is a long multi-agent job and the environment can be interrupted or reset mid-run:
- After Step 0 (file intake): `git add assets/ && git commit -m "Add source files and orchestration prompt to assets/" && git push`.
- After Step 1 (all page batches written): `git add batches/ && git commit -m "Add raw page-batch transcriptions" && git push`.
- After Step 1b (all voice segments written): `git add voice-transcripts/raw-segments/ && git commit -m "Add raw voice segment cleanups" && git push`.
- After Step 2 (consolidation): `git add transcript.md voice-transcripts/voice-*.md && git commit -m "Consolidate notes transcript and voice transcripts" && git push`.
- After Step 3: `git add wealthcon-research.md && git commit -m "Add WealthCon research brief" && git push`.
- After Step 4: `git add synthesis.md && git commit -m "Add synthesis" && git push`.
- After Step 5 (QA): one final commit describing the result, e.g. `git commit -m "QA pass: 173/173 pages, N illegible markers" --allow-empty && git push`.

If a batch or segment agent fails partway through a step, commit and push the ones that succeeded before retrying the failures — don't let a partial failure risk the whole run's progress by holding it all in one uncommitted batch.

## Run unattended

The user is turning their computer off once this starts and won't be around to answer questions or approve anything until they're back. One honest caveat up front: whether this environment can actually stay alive for 10+ hours unattended depends on its own session/idle limits, which this prompt has no control over — the checkpointing above is the mitigation for that, not a guarantee against it.

- Run Steps 0 through 5 straight through with no pauses for confirmation. Every instruction in this prompt is pre-approved — don't ask before dispatching subagents, running a step, committing, or pushing.
- Never stop a step to wait on a decision only the user could make. Where something is genuinely ambiguous, follow the hard rules below: mark it (`[illegible]`, `[unclear]`, a flagged note) and keep moving, rather than pausing for input that isn't coming. The only things that should stop the run entirely are a hard technical failure (rejected push, missing file, repeated subagent crash on the same unit of work) or full completion.
- If the session times out or the environment resets while the user is away, nothing committed and pushed is lost. On restart, check the git log and whatever QA/status markers already exist to figure out which step to resume from — don't start over from Step 0 if earlier steps are already committed.
- When everything is done, write `RUN_COMPLETE.md` at the repo root: what finished, the Step 5 QA numbers, and anything that needs the user's attention (ambiguous splits, failed batches, low-confidence transcriptions). Commit and push it. This should be the first thing the user reads when they're back.

## Step 0 — File intake

Three source files have already been uploaded directly into the repo, in `assets/`:

- `10X Wealth-Con Nov. 2024 Notes-1-87.pdf` — handwritten notes, pages 1–87.
- `10X Wealth-Con Nov. 2024 Notes-88-173.pdf` — handwritten notes, pages 88–173 (continues numbering from the first file; combined the two cover all 173 pages).
- `10x WealthCon 2024 voice memo transcripts.rtfd.zip` — confirmed. `.rtfd` is a macOS Rich Text Format Directory (a bundle, not a flat file), zipped for upload. Confirmed: this contains all five transcripts. Before Step 1b, it has to be unzipped and the RTF content converted to plain text before the five can be located — see Step 1b for exactly how.

Do this before anything else:
- Confirm all three files are actually present in `assets/`. If any is missing, stop and flag it rather than proceeding with partial source material.
- Save a full copy of this entire orchestration prompt into `assets/orchestration-prompt.md`. Every subagent dispatched in the steps below should be able to open that file and read its exact instructions and hard rules directly, rather than relying on a summary or on what it's told secondhand.
- If any agent, at any point in this job, is unsure what it's supposed to do, which file it owns, or what the rules are for handling something it can't read clearly, it should refer back to `assets/orchestration-prompt.md` first before guessing or asking the user.
- Commit this intake before moving on to Step 1.

## Sources

- `assets/10X Wealth-Con Nov. 2024 Notes-1-87.pdf` and `assets/10X Wealth-Con Nov. 2024 Notes-88-173.pdf` — 173 handwritten pages total, split across two scanned PDFs, page numbering continuous across both.
- `assets/10x WealthCon 2024 voice memo transcripts.rtfd.zip` — one uploaded file (a zipped macOS RTFD bundle, see Step 0/1b for unzipping). Rough transcript text (not audio), covering five voice memos from the event, with known individual lengths:
  | Memo | Length |
  |---|---|
  | voice-1 | 1:56:18 |
  | voice-2 | 2:34:40 |
  | voice-3 | 3:02:29 |
  | voice-4 | 0:54:38 |
  | voice-5 | 2:04:05 |
  | **Total** | **10:32:10** |

  Confirmed: all five are in this one file, concatenated. See Step 0 and Step 1b for how to locate and split them.

## Deliverables

1. `transcript.md` — a faithful, complete transcription of all 173 pages, in reading order.
2. `voice-transcripts/voice-1.md` through `voice-5.md` — cleaned, readable versions of each rough transcript.
3. `wealthcon-research.md` — a factual, sourced research brief on the 10X WealthCon event, November 2024.
4. `synthesis.md` — one seamless synthesis drawing on the handwritten notes, all five voice transcripts, and the research together — not five separate summaries stapled to a notes summary.

## Step 1 — Batch and parallelize transcription

- The 173 pages span two files: pages 1–87 in the first PDF, pages 88–173 in the second. Treat page numbers as continuous across both when batching and in the final `transcript.md` — the split is a file-size artifact, not a structural break in the notes.
- Split the 173 pages into batches of ~15 pages each (about 12 batches), crossing the file boundary at page 87/88 like any other page boundary.
- Dispatch one subagent per batch in parallel via the Task tool.
- Each subagent's job per page:
  - Render the page as an image, pulling from whichever of the two PDFs actually contains that page number.
  - Transcribe the handwriting, preserving structure — headers, bullets, arrows, boxed callouts, underlines — as markdown.
  - Never guess at a word you can't read. Mark it `[illegible]` and move on.
  - Do not paraphrase, summarize, correct, or editorialize at this stage. Literal transcription only.
  - If a page is blank, a doodle, or clearly not notes, say so explicitly rather than omitting it — a missing page should never look identical to a blank one.
- Each subagent writes its output to `batches/pages-XXX-YYY.md`, one file per batch, with a page-number heading before each page's content.

## Step 1b — Batch and parallelize voice memo cleanup

Only one file was uploaded, containing all five memos concatenated, so extract and split it first:

- Unzip it: `unzip "assets/10x WealthCon 2024 voice memo transcripts.rtfd.zip" -d assets/` — this produces a `.rtfd` folder (a bundle, not a single file). The actual text lives in an `.rtf` file inside that bundle, typically named `TXT.rtf`; there may also be numbered image files in the bundle from embedded attachments, which can be ignored for this task.
- Convert that RTF to plain text. `textutil` (macOS-only) won't exist in this Linux cloud environment. Use `unrtf --text "path/to/TXT.rtf" > assets/transcripts-raw.txt` if `unrtf` is available; if not, install and use a Python RTF parser (e.g. `pip install striprtf` and strip the markup with it) rather than trying to read the raw RTF markup as if it were the transcript — RTF control codes are not part of the actual spoken content and shouldn't end up in any output file.
- Once you have clean plain text, find the boundaries between the five transcripts within it — internal breaks like a recording header, a timestamp resetting to zero, or a new recording's typical opening. Use the five known durations above as a check: each split section's approximate spoken length should roughly match one of the five figures, in the order given.
- Write the five separated transcripts to `voice-transcripts/raw/voice-1.txt` through `voice-5.txt` before doing anything else. If a boundary is genuinely ambiguous, say so explicitly in a note rather than forcing a clean split that isn't actually there — an approximate split with a flagged uncertainty beats a confident wrong one.

Once split, the five files are rough transcripts already, not audio — the remaining job is cleanup and organization, not transcription from scratch. Split each file into ~20-minute segments and dispatch one subagent per segment, in parallel:

| File | Length | ~20-min segments |
|---|---|---|
| voice-1 | 1:56:18 | 6 |
| voice-2 | 2:34:40 | 8 |
| voice-3 | 3:02:29 | 10 |
| voice-4 | 0:54:38 | 3 |
| voice-5 | 2:04:05 | 7 |

That's roughly 34 segment agents across the five files. Each one:

- Cleans up the rough text: punctuation, paragraph breaks, obvious transcription-error fixes, speaker labels where the transcript makes the speaker identifiable (leave unlabeled if it doesn't).
- Does not rewrite, summarize, or paraphrase content — cleanup only, same as the handwriting rule.
- Marks anything garbled or ambiguous `[unclear]` rather than guessing at what was said.
- Writes to `voice-transcripts/raw-segments/<file-id>/segment-NN.md`.

## Step 2 — Consolidate

- One agent reads every file in `batches/` in page order and merges them into `transcript.md`. It verifies all 173 pages are present exactly once — no gaps, no duplicates — and reports any it finds instead of silently proceeding.
- One agent per voice file reads that file's segments in order and merges them into `voice-transcripts/voice-N.md`, checking for time-range gaps or overlaps the same way.
- These agents assemble and clean only. They do not add commentary or analysis.

## Step 3 — Research the event (factual, web-sourced only)

- One agent researches the 10X WealthCon, November 2024: exact dates, city and venue, format, Grant Cardone's role, other confirmed speakers, public ticket pricing if available, and any press coverage or attendee reviews from reputable sources.
- Every claim in this file needs a cited source next to it. No claim without one. If something can't be verified, say that plainly instead of filling the gap.
- Output to `wealthcon-research.md`.

## Step 4 — Synthesize

One agent reads `transcript.md`, all five files in `voice-transcripts/`, and `wealthcon-research.md` together and writes a single `synthesis.md`. This has to read as one integrated piece, not five audio summaries plus a separate notes summary stapled together — where a voice segment and a page of notes cover the same session or topic, treat them as one source and say so, and note where audio and handwritten notes on the same topic agree, add to each other, or conflict.

Cover:
- The main themes and frameworks that recur across notes and audio.
- Specific claims, numbers, or strategies mentioned, and whether they're checkable against the research file or general public information.
- Where the material reflects Cardone's own sales/investment philosophy specifically, versus more general material that could have come from any speaker.
- An honest read of what's substantive versus what's rhetorical or hype, in plain direct language, not promotional language.

**Flag explicitly, every time it appears in either source:** any specific return rate, guarantee, or investment result Cardone or another speaker states as fact. Given his public record includes multiple lawsuits alleging misleading return claims to investors, treat any number like this as a claim to verify, never as settled fact, regardless of how confidently it's stated in the notes or audio.

## Step 5 — QA pass

One final agent confirms, before declaring the job done:

- All 173 pages are accounted for in `transcript.md`.
- All five voice files are accounted for in `voice-transcripts/`, with no time-range gaps, and total cleaned duration roughly matches the 10:32:10 source total.
- Every factual claim in `wealthcon-research.md` has a source next to it.
- `synthesis.md` does not restate an unverified claim, especially a return or performance number, as settled fact.
- Report the final page count transcribed, the `[illegible]`/`[unclear]` marker count for notes and audio separately, and any batch or segment that failed or needs a manual look.

## Hard rules

- Never invent or infer handwriting or audio content you can't actually make out. `[illegible]` / `[unclear]` beats a plausible-sounding guess, every time.
- Keep transcription/cleanup and commentary in separate files. `transcript.md` and the `voice-transcripts/` files are source material; `synthesis.md` is analysis. Don't blend them.
- If OCR/vision quality on a page batch, or transcript quality on an audio segment, is poor, say so in that output rather than producing a confident-sounding version that's actually a guess.
- Never state a specific investment return, guarantee, or dollar result from the source material as fact in `synthesis.md`. Report it as "the notes/audio claim X" and flag it for verification.
