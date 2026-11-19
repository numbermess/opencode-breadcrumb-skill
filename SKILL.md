# Breadcrumb Journaling Skill

Automated journal entry creation for OpenCode sessions using the Panopticon breadcrumb system. Creates structured markdown entries with optional file attachments, integrated via git hooks and session triggers.

## What It Does

This skill orchestrates automatic and manual journal entry creation:

- **Session tracking**: Log what you worked on at start/end of coding sessions
- **Commit documentation**: Automatically capture commit messages and changed files  
- **Manual checkpoints**: Create ad-hoc entries during complex work
- **Error logging**: Document failures, debugging attempts, recovery steps
- **Attachment support**: Include screenshots, logs, PDFs in journal entries

## Setup (One-Time)

### 1. Install the Skill

Add this URL to your OpenCode config (`~/.config/opencode/opencode.jsonc`):

```jsonc
{
  "skills": [
    "https://raw.githubusercontent.com/numbermess/opencode-breadcrumb-skill/main/SKILL.md"
  ]
}
```

### 2. Configure the Panopticon API (Optional but Recommended)

The skill works best when connected to a running breadcrumb API server:

**Option A - Use centralized chopper config:**  
If you're on the LAN with access to `chopper-01.local`, it serves defaults automatically via `/opencode-config`. Just ensure your OpenCode client fetches from there.

**Option B - Manual setup:**
```bash
# Clone panopticon repo (or just copy hook scripts)
git clone https://github.com/numbermess/panopticon.git ~/projects/panopticon

# Start the API server:
cd ~/projects/panopticon/api && go build -o breadcrumb-api . && ./breadcrumb-api &

# Set environment variable in your shell or OpenCode config:
export PANOPTICON_API_URL="http://localhost:8080/api/breadcrumb"
```

### 3. Enable Automatic Hooks (Optional)

**Git post-commit hook:**  
Add this to `.git/hooks/post-commit` in any project where you want automatic commit logging:

```bash
#!/bin/bash
# Post-commit breadcrumb - logs completed work automatically
curl http://localhost:8080/api/breadcrumb \
  --data-urlencode "title=$(git log -1 --pretty=format:'%s')" \
  --data-urlencode "content=Commit $(git log -1 --pretty=format:'%h')\n\n$(git diff-tree --no-commit-id --name-only -r HEAD | sed 's/^- //g' | xargs printf '- %%s\\n')" || true
```

Make it executable: `chmod +x .git/hooks/post-commit`

**Session-based triggers:**  
Add to your project's `.opencode.jsonc`:
```jsonc
{
  "hooks": {
    "session-start": "./panopticon/hooks/context-mode/breadcrumb-hook.sh manual \"Session started\"",
    "commit-post": "./panopticon/hooks/context-mode/breadcrumb-commit.sh",
    "session-end": "./panopticon/hooks/context-mode/breadcrumb-hook.sh session-end"
  }
}
```

## Usage Patterns

### Manual Breadcrumb Creation (via this skill)

When you want to log something manually, just ask:

> "Create a manual breadcrumb entry titled 'Debugging auth flow' with this summary of what I just figured out..."

The skill will call the API automatically and create a journal entry. If there are attachments in context (screenshots, logs), it can include them too.

### With Attachments

To add files to your journal entries:

```bash
# Using the CLI helper script directly:
./scripts/post-breadcrumb.sh \
  --attach /path/to/screenshot.png \
  "Error Analysis" "" <(echo "Here's what I found...")

# Or ask OpenCode to create one with attachments if files are in context.
```

### Automatic Triggers (if configured)

- **Session start**: Logs when you begin working  
- **After commits**: Captures commit message and changed files automatically
- **Session end**: Summarizes the session (optional, can be disabled)

## Journal Entry Structure

Entries are stored in a self-contained folder structure:

```
{journal-root}/repos/{repository-name}/
├── YYYY-MM-DD-title-slug-breadcrumb.md    # Main markdown entry  
└── attachments/
    └── YYYY-MM-DD-title-slug-breadcrumb/  # Attachment folder for this entry
        ├── screenshot.png
        └── error-log.txt
```

The markdown file includes an "## Attachments" section listing all uploaded files with relative paths.

## Configuration (Environment Variables)

Set these before starting OpenCode or in your project's `.opencode.jsonc`:

- `PANOPTICON_API_URL` - Breadcrumb API endpoint  
  Default: `http://localhost:8080/api/breadcrumb`
- `BREADCRUMB_API_KEY` - Authentication token if required (optional)
- `JOURNAL_ENABLED` - Set to `"false"` to temporarily disable automatic logging

## CLI Helper Scripts

If you clone the panopticon repo, these scripts are available in `/scripts/`:

**post-breadcrumb.sh:** Create journal entries from command line:
```bash
# Basic usage (JSON mode):  
echo "Entry content" | ./scripts/post-breadcrumb.sh "My Title" [repository] < /dev/null

# With attachments:
./scripts/post-breadcrumb.sh \
  --attach screenshot.png \
  --attach error.log \
  "Debugging session" "" <(cat notes.md)
```

**test-attachments.sh:** Verify API functionality with sample entries.

## Troubleshooting

**Skill not available in OpenCode:**  
Check that the URL is correct and accessible: `curl https://raw.githubusercontent.com/numbermess/opencode-breadcrumb-skill/main/SKILL.md`

**API calls failing:**  
Verify the server is running: `curl http://localhost:8080/api/breadcrumb -X POST --data '{"title":"test","content":"testing"}'`

**Attachments not appearing in journal:**  
- Confirm files exist at specified paths
- Check file permissions on attachment directory
- Review API logs for upload errors (`journal-api.log` or console output)

## Related Resources

- **Panopticon repo**: https://github.com/numbermess/panopticon (full implementation with hooks, docs, examples)
- **API documentation**: See `api/README.md` in panopticon repo  
- **Integration guide**: See `RIBU1C_INTEGRATION.md` for LAN-wide deployment

## Version History

This skill is versioned alongside the Panopticon breadcrumb system. Check the main repository's commit history for updates and new features.
