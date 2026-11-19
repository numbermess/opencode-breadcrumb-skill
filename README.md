# OpenCode Breadcrumb Journaling Skill

Automated journal entry creation skill for OpenCode sessions using the Panopticon breadcrumb system.

## Quick Start

Add this URL to your `~/.config/opencode/opencode.jsonc`:

```jsonc
"skills": [
  "https://raw.githubusercontent.com/numbermess/opencode-breadcrumb-skill/main/SKILL.md"
]
```

See **SKILL.md** for full documentation and setup instructions.

## What It Does

- Creates structured markdown journal entries from OpenCode sessions  
- Supports file attachments (screenshots, logs, PDFs)
- Integrates with git hooks for automatic commit logging
- Session-based triggers (start/end checkpoints)

## Repository Structure

```
├── SKILL.md          # Main skill instructions and documentation
└── README.md         # This file - quick reference
```

## Related Projects

- **Panopticon**: Full breadcrumb system implementation with API, hooks, CLI tools  
  https://github.com/numbermess/panopticon
  
- **Rib u1c-router**: Centralized OpenCode config serving for LAN clients (includes breadcrumb defaults)  
  https://github.com/numbermess/ribu1c-router

## License

MIT - Free to use and modify. See LICENSE file if needed.
