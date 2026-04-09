# Good Deck

An [Agent Skill](https://agentskills.io) that helps AI coding assistants build polished, single-file HTML slide decks — no frameworks, no build step.

## What it does

Good Deck turns your content (outlines, notes, stats) into a self-contained `index.html` presentation with:

- Keyboard navigation (arrows, space, G to jump)
- Animated slide transitions
- CSS theming via variables
- Built-in practice mode with talk track (P key)
- Speaker avatars and Slide 0 (navigation info)
- Optional Flask wrapper for password-protected hosting

## Install

Requires [Node.js](https://nodejs.org/).

```bash
npx skills add https://github.com/ajot/good-deck --skill good-deck
```

## Usage

Once installed, trigger the skill by asking your AI assistant to create a presentation, slide deck, or slides. Or invoke it directly:

- **Claude Code:** `/good-deck`
- **Codex:** `$good-deck`

### Try it

Paste this prompt after installing to see it in action:

> /good-deck Create a 7-slide presentation titled "Why Side Projects Matter" for a developer meetup audience. Use a dark theme. Include these points:
> 1. Title slide with subtitle "Lessons from building in public"
> 2. The myth of the "perfect idea"
> 3. Three reasons side projects accelerate your career (use a card grid)
> 4. Real-world stats: 60% of YC companies started as side projects
> 5. The compound effect of shipping small things
> 6. Common pitfalls and how to avoid them
> 7. Closing slide: "Just start. Ship it. Iterate."

## Compatibility

Works with any AI coding assistant that supports the [Agent Skills](https://agentskills.io) format:

- Claude Code
- GitHub Copilot
- Codex
- Gemini CLI
- Cursor

## License

MIT
