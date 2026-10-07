# ai-source-triage

A Claude skill for keeping up with AI without watching everything.

Someone asked me how I keep up with AI. My first answer was "you can't." That is only partly true. You can't keep up with all of it, but you can filter it. Most AI content is mediocre, repeated, or noise, so the job is to reject fast and spend real attention only on the small share that is great.

This skill runs that filter on one video or article at a time.

## How it works

It has two parts.

**It judges the content.**

1. **Triage.** Three lines on what the source is actually about, ending in KEEP or SKIP.
2. **Delta.** What does this say that my kept sources don't? An empty list means reject.
3. **Watch.** Only now do I spend viewing time, at 2x, on the net-new parts.
4. **Extract.** Techniques with exact steps and prompts, things to avoid, and caveats. I can keep a whole source or only the parts worth keeping.
5. **File.** A dated digest goes into a tracker that feeds my own AI cheat sheet.

**It learns the creators.**

Every keep or skip is scored against the channel it came from. Three skips in a row and a channel stops being looked at. Three or more keeps with at most one skip and it becomes trusted, which skips triage. Over time the scorecard becomes a short list of who is consistently worth following.

The cheap checks come first. The scorecard and two short prompts reject most sources before I watch a minute.

**Batch mode.** Send several links at once and get one ranked table back, with overlap between them called out and a suggested watch order.

## Install

Copy the `ai-source-triage` folder (it only needs `SKILL.md`) into your Claude skills:

- **Claude Code:** `~/.claude/skills/ai-source-triage/SKILL.md`
- **Claude apps:** zip the folder and upload it under Settings, then Skills.

Then drop a link and say "triage this."

On first run it asks where your tracker lives (any doc or markdown file) and what topics you want skipped automatically. It keeps the channel scorecard in that same tracker.

## Reading YouTube

Many cloud environments can't fetch YouTube transcripts. The skill tries a browser on your machine first, then `yt-dlp` if it has a local shell, and falls back to giving you prompts for NotebookLM. The fallback prompts are in `SKILL.md` and work fine on their own if you'd rather run the process by hand.

## Status

Early. The browser route for transcripts has worked on a handful of videos, and the channel scorecard and batch mode are new, so expect rough edges.

## License

MIT
