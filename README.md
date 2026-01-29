# wp-accessibility-claude-skill

WCAG 2.1 AA accessibility enforcement for WordPress themes covering skip links, landmarks, color contrast, form accessibility, screen reader support, keyboard navigation, and focus management.

## Installation

### Claude Code Plugin Marketplace

```bash
/plugin install https://github.com/xonack/wp-accessibility-claude-skill
```

### Manual Installation

Copy the skill file to your Claude Code skills directory:

```bash
mkdir -p ~/.claude/skills/wp-accessibility
cp skills/wp-accessibility/SKILL.md ~/.claude/skills/wp-accessibility/SKILL.md
```

## Usage

Once installed, the skill activates automatically when working on relevant WordPress tasks. You can also invoke it directly:

```
/wp-accessibility
```

## Structure

```
wp-accessibility-claude-skill/
├── README.md
├── LICENSE
├── .claude-plugin/
│   └── plugin.json
└── skills/
    └── wp-accessibility/
        └── SKILL.md
```

## Contributing

PRs welcome. Please follow the [Agent Skills specification](https://agentskills.io/specification) for SKILL.md format.

## License

MIT
