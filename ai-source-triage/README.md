# ai-source-triage

A Claude skill for keeping up with AI without watching everything.

Someone asked me how I keep up with AI. My first answer was "you can't." That is only partly true. You can't keep up with all of it, but you can filter it. Most AI content is mediocre, repeated, or noise, so the job is to reject fast and spend real attention only on the small share that is great.

This skill runs that filter on one video or article at a time.

## How it works

1. **Skip list.** Channels that have failed three times never get looked at again.
2. **Triage.** Three lines on what the source is actually about, ending in KEEP or SKIP.
3. **Delta.** What does this say that my kept sources don't? An empty list means reject.
4. **Watch.** Only now do I spend viewing time, at 2x, on the net-new parts.
5. **Extract.** Thesis, techniques with exact steps and prompts, things to avoid, and caveats.
6. **File.** A dated digest goes into a tracker that feeds my own AI cheat sheet.

The cheap checks come first. A skip list and two short prompts reject most sources before I watch a minute.

## Install

Copy the `ai-source-triage` folder (it only needs `SKILL.md`) into your Claude skills:

- **Claude Code:** `~/.claude/skills/ai-source-triage/SKILL.md`
- **Claude apps:** zip the folder and upload it under Settings, then Skills.

Then drop a link and say "triage this."

On first run it asks where your tracker lives (any doc or markdown file) and what topics you want skipped automatically.

## Reading YouTube

Many cloud environments can't fetch YouTube transcripts. The skill tries a browser on your machine first, then `yt-dlp` if it has a local shell, and falls back to giving you prompts for NotebookLM. The fallback prompts are in `SKILL.md` and work fine on their own if you'd rather run the process by hand.

## Status

Early. The process is one I've used by hand for a while. The browser route for transcripts is new and lightly tested, so expect rough edges.

## License

MIT
