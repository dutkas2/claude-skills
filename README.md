# Claude Code Skills

Custom skills for [Claude Code](https://claude.ai/code) by Shane Dutka.

## Skills

| Skill | Description |
|-------|-------------|
| [superpixels](./superpixels/) | Design and build remarkable, memorable micro-interactions for web projects. Inspired by Glen Allsopp's Superpixels framework. |

## Installation

To install a skill, copy the skill folder into your Claude Code skills directory:

```bash
# Copy a single skill
cp -r superpixels ~/.claude/skills/superpixels

# Or clone the whole repo and symlink
git clone https://github.com/dutkas2/claude-skills.git ~/claude-skills
ln -s ~/claude-skills/superpixels ~/.claude/skills/superpixels
```

Then use it in Claude Code by typing `/superpixels` (or whatever the skill name is).

## Adding New Skills

Each skill is a folder containing a `SKILL.md` file with YAML frontmatter:

```
skill-name/
  SKILL.md    # Skill definition with name, description, and instructions
```

## License

MIT
