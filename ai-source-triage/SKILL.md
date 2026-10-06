---
name: ai-source-triage
description: Triage, dedupe, and digest an AI video or article into a personal sources tracker. Use when the user drops a YouTube or article link, or says "triage this", "is this worth watching", or "add this to my AI sources".
---

# AI source triage

You cannot keep up with all AI content, but you can filter it. Most of it is mediocre, repeated, or noise. This skill rejects fast and spends attention only on the small share that is great. It runs on one source at a time. Cheap checks come first. Stop as soon as a source fails.

## Setup

The skill needs one **tracker**: a document or markdown file the user owns that holds three things.

- A **Sources table**: number, date added, linked title, type, author, why it matters, status.
- A **Skip list** of channels and authors that have failed triage three times.
- A **Digests** section with one dated block per kept source.

On first run, ask the user where the tracker lives, or offer to create one. Read it at the start of every run. It is the comparison set for the delta check.

Also ask once what the user does not care about (for example, "faceless content automation" or "crypto"). Record those under a **Not interested** heading in the tracker and treat them as automatic skips.

## Step 1: Skip list

Identify the channel or author. For YouTube, the oEmbed endpoint `https://www.youtube.com/oembed?url=<video url>&format=json` returns the title and channel. If the channel is on the Skip list, say so in one line and stop.

## Step 2: Get the content

**Articles and text pages:** fetch the page directly.

**YouTube videos:** many cloud environments cannot fetch YouTube transcripts. Use whichever route is available, in this order.

1. **A browser on the user's machine** (a browser extension or built-in browser tool). Open the video in a new tab. Pause and mute it. Expand the description and click "Show transcript". Then wait: the panel can take 20 to 30 seconds to fill. Poll until `ytd-transcript-segment-renderer` elements appear inside the expanded transcript panel, and do not give up before 40 seconds. Read the timestamp and text of each segment, then close the tab. Do not request the caption track URL directly. It tends to return empty.
2. **A local command line**, if you have a shell on the user's machine: `yt-dlp --skip-download --write-auto-subs --sub-langs en <url>` saves the captions as a file.
3. **Fallback: NotebookLM or a pasted transcript.** Tell the user plainly that you could not read the video, and give them these prompts to run in a NotebookLM notebook that holds their kept sources. Continue from what they paste back.
   - Triage: "In three lines: what are they actually building or talking about here, who is it for, and what here is not already covered by my other sources? End with KEEP or SKIP."
   - Delta: "Compared to my other sources, what does this one say that none of them do? List only the net-new techniques or claims."
   - Extract: the note-taker prompt at the end of this file.

Summarizing tools shorten the prompts they quote. When working from their output, ask the user to screenshot any prompt they plan to reuse word for word.

## Step 3: Triage

Answer in three lines: what they are actually building or talking about, who it is for, and what is not already in the tracker. End with KEEP or SKIP.

SKIP by default when the content matches the Not interested list, is a thin repeat of what the tracker already holds, or is mostly a product pitch. On SKIP, record the failure against the channel. Three failures puts the channel on the Skip list. Then stop.

## Step 4: Delta

For a KEEP, list only the net-new techniques or claims compared to the digests in the tracker, each with its timestamp. An empty list means reject and stop. A short list tells the user which minutes to watch.

Report triage and delta, then pause. The user watches the net-new parts at 2x and decides whether the source is worth keeping. Continue to Step 5 only when they say to keep it. If they ask you to go straight through, skip the pause.

## Step 5: Extract

Write the digest from the source only. Add no outside knowledge. Do not merge separate techniques.

- **Thesis:** the core argument in 2 to 3 sentences.
- **Do this:** a table of techniques with columns Technique, How, Why. "How" carries the exact steps, settings, and prompt wording as the source states them, with timestamps for videos.
- **Avoid:** each anti-pattern and the reason given.
- **Treat with caution:** anything hedged, version-specific, or likely to go stale. Flag advice that overlaps with something the speaker sells. Label figures as the speaker's claims.

Leave out sponsor segments and giveaways. Quote only the short prompts and commands needed to use a technique, and summarize everything else in your own words. Never paste the transcript or long passages from it.

## Step 6: File

In the tracker:

1. Add a row to the Sources table with today's date and status "Digested".
2. Add the digest as a new dated block under Digests.

Then tell the user in two or three lines what was added and what was net-new.

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
