---
name: ai-source-triage
description: Triage, dedupe, and digest AI videos and articles into a personal sources tracker, and keep a scorecard of which channels are worth following. Use when the user drops one or more YouTube or article links, or says "triage this", "is this worth watching", or "add this to my AI sources".
---

# AI source triage

You cannot keep up with all AI content, but you can filter it. Most of it is mediocre, repeated, or noise. This skill rejects fast, spends attention only on the small share that is great, and learns which creators are consistently worth the time. Cheap checks come first. Stop as soon as a source fails.

## Setup

The skill needs one **tracker**: a document or markdown file the user owns. It holds four things.

- A **Sources table**: number, date added, linked title, type, author, why it matters, status.
- A **Channel scorecard**: channel or author, keeps, skips, and a status of Trusted, Neutral, or Skip.
- A **Not interested** list of topics that are automatic skips.
- A **Digests** section with one dated block per kept source.

On first run, ask the user where the tracker lives, or offer to create one, and ask what topics they do not care about. Read the tracker at the start of every run. Its digests are the comparison set for the delta check.

## Channel scorecard rules

Every verdict is scored against the channel it came from.

- A SKIP adds one to the channel's skips. A KEEP, full or partial, adds one to its keeps.
- **Skip status:** three skips in a row. A keep resets the streak.
- **Trusted status:** three or more keeps with at most one skip.
- Everything else is Neutral.
- The user can override any status at any time. Record an override as a note on that row and do not recompute over it.

## Step 1: Check the scorecard

Identify the channel or author. For YouTube, the oEmbed endpoint `https://www.youtube.com/oembed?url=<video url>&format=json` returns the title and channel.

- **Skip:** say so in one line and stop.
- **Trusted:** skip Step 3 and go straight from Step 2 to the delta check.
- **Neutral or new:** run every step.

## Step 2: Get the content

**Articles and text pages:** fetch the page directly.

**YouTube videos:** many cloud environments cannot fetch YouTube transcripts. Use whichever route is available, in this order.

1. **A browser on the user's machine** (a browser extension or built-in browser tool).
   1. Open the video in a new tab. Pause and mute it. Do not let it play.
   2. Wait for the page to finish rendering, about ten seconds. The description must be present before the next step works.
   3. Expand the description and click "Show transcript".
   4. Bring the tab to the foreground. The transcript panel does not load in a background tab. Taking a screenshot of the tab is enough to do this.
   5. Poll for `ytd-transcript-segment-renderer` elements inside the expanded panel. Use several short checks, each well under 30 seconds, because one long script call can time out. Allow up to a minute in total.
   6. Read the page text. Once the panel has loaded, a single page-text read returns the description, the chapter list, and the full transcript with timestamps.
   7. Close the tab.

   Do not request the caption track URL directly. It tends to return empty.
2. **A local command line**, if you have a shell on the user's machine: `yt-dlp --skip-download --write-auto-subs --sub-langs en <url>` saves the captions as a file.
3. **Fallback: NotebookLM or a pasted transcript.** Tell the user plainly that you could not read the video, and give them these prompts to run in a NotebookLM notebook that holds their kept sources. Continue from what they paste back.
   - Triage: "In three lines: what are they actually building or talking about here, who is it for, and what here is not already covered by my other sources? End with KEEP or SKIP."
   - Delta: "Compared to my other sources, what does this one say that none of them do? List only the net-new techniques or claims."
   - Extract: the note-taker prompt at the end of this file.

Summarizing tools shorten the prompts they quote. When working from their output, ask the user to screenshot any prompt they plan to reuse word for word.

## Step 3: Triage

Answer in three lines: what they are actually building or talking about, who it is for, and what is not already in the tracker. End with KEEP or SKIP.

SKIP by default when the content matches the Not interested list, is a thin repeat of what the tracker already holds, or is mostly a product pitch. On SKIP, update the scorecard and stop.

## Step 4: Delta

For a KEEP, list only the net-new techniques or claims compared to the digests in the tracker, each with its timestamp or section. An empty list means reject, score it as a skip, and stop. A short list tells the user which minutes to watch.

Report triage and delta, then pause and ask one question: keep all of it, keep only some parts, or skip? The user watches the net-new parts at 2x and decides. If they ask you to go straight through, keep everything in the delta list without pausing.

## Step 5: Extract

Extract only the parts the user chose. Write from the source only. Add no outside knowledge. Do not merge separate techniques.

- **Thesis:** the core argument in 2 to 3 sentences. Leave it out on a partial keep if the kept parts do not need it.
- **Do this:** a table of techniques with columns Technique, How, Why. "How" carries the exact steps, settings, and prompt wording as the source states them, with timestamps for videos.
- **Avoid:** each anti-pattern and the reason given.
- **Treat with caution:** anything hedged, version-specific, or likely to go stale. Flag advice that overlaps with something the speaker sells. Label figures as the speaker's claims.

Leave out sponsor segments and giveaways. Quote only the short prompts and commands needed to use a technique, and summarize everything else in your own words. Never paste the transcript or long passages from it.

## Step 6: File

In the tracker:

1. Add a row to the Sources table with today's date. Set the status to "Digested", or to "Partly digested:" followed by the parts kept.
2. Add the digest as a new dated block under Digests. On a partial keep, open the block with one line saying what was kept and what was left out on purpose.
3. Update the Channel scorecard and recompute the channel's status.

Then tell the user in two or three lines what was added, what was net-new, and any change in the channel's status.

## Batch mode

When the user sends more than one link, run Steps 1 to 4 on each, then report once.

- One table, best first: source, channel and its status, verdict, and the net-new items with timestamps.
- Call out overlap between the sources in the batch. When two cover the same ground, say which one covers it better and which minutes of the other are still worth watching.
- Recommend a watch order.
- Pause once for the whole batch. The user answers keep, partial, or skip for each.

Compare each source against the tracker and against the other sources in the batch, so the second video on a topic is not credited for what the first already said.

## Upkeep

AI advice goes stale quickly. When the user asks for an audit, or when a digest is more than about two months old, review the digests and flag anything outdated or conflicting. For each, recommend update, merge, or delete. Change nothing until the user signs off.

## Note-taker prompt (for the fallback route)

```
Act as my note-taker for a personal AI playbook. From this source, extract:

1. THESIS: The core argument in 2-3 sentences.
2. TECHNIQUES: Every distinct technique, tip, or workflow, numbered. For each one give:
   - What it is (one sentence)
   - Exact steps, settings, or prompt wording used, as specifically as the source states them
   - Why it works, per the speaker
   - When to use it and when not to
3. THINGS TO AVOID: Every anti-pattern or mistake called out, and the reason given.
4. TOOLS AND FEATURES: Every product, feature, or setting named, with what it was used for.
5. EXAMPLES: Concrete examples or demos shown, briefly.
6. CAVEATS: Anything the speaker hedged on, or that looks version-specific or likely to go stale.

Rules: Be complete rather than brief. Do not merge separate techniques. Only include what is actually in the source, and do not add outside knowledge. Include timestamps where available.
```
