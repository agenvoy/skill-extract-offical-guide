# extract-offical-guide - Documentation

> Back to [README](../README.md)

## Prerequisites

- Claude Code (loads skills from `~/.claude/skills/`)
- Network access to `platform.claude.com`, `developers.openai.com`, `ai.google.dev`, and `web.archive.org`
- A target repo containing `configs/prompts/system_prompt/official_guides/` (Agenvoy by default; pass `--to` for any other repo)
- Go toolchain (the Verify phase runs `go build -tags fts5 ./...`)
- Python 3 (heading-order check in the Verify phase)

## Installation

### From Source

```bash
git clone https://github.com/pardnchiu/extract-offical-guide.git ~/.claude/skills/extract-offical-guide
```

### Verify Loading

```bash
ls ~/.claude/skills/extract-offical-guide/SKILL.md
```

Restart Claude Code and `extract-offical-guide` appears in the skill list.

## Configuration

The skill reads no environment variables; command arguments and the paths below control all behavior.

| Item | Path | Description |
|------|------|-------------|
| Source archive | `~/.claude/skills/extract-offical-guide/offical_guide/<key>.md` | Full vendor text; first line is `Retrieved <YYYY-MM-DD> from <url>` |
| Default output | `<repo>/configs/prompts/system_prompt/official_guides/` | The skill asks instead of creating it when missing |

## Usage

### Basic

Run download, base extraction, and model extraction for every key in the source registry:

```text
/extract-offical-guide
```

### Specific Models

Process only the given keys; passing keys skips Phase B by default so the always-injected base files stay untouched:

```text
/extract-offical-guide claude-opus-5-5 gpt-5.6
```

### Download or Extract Only

```text
# Refresh the archive without extracting
/extract-offical-guide gemini --download-only

# Skip download and extract from the local archive
/extract-offical-guide gpt-5.4 --extract-only
```

### Advanced

```text
# Write to another repo's output directory
/extract-offical-guide claude-sonnet-5 --to ~/Desktop/git/other-repo/prompts/official_guides

# Re-extract only _base.md and _base_unlisted.md
/extract-offical-guide --base-only
```

When an archive or output file already exists, the skill prints a diff summary and asks before overwriting; nothing is written without approval.

## CLI Reference

### Commands

| Command | Syntax | Description |
|---------|--------|-------------|
| `/extract-offical-guide` | `/extract-offical-guide [<key>...] [--to <DEST>] [--download-only] [--extract-only] [--base-only] [--skip-base]` | Download official prompting guides and extract them into model guide files |

### Flags

| Flag | Description |
|------|-------------|
| `<key>...` | Process only these keys (e.g. `claude-opus-5`, `gpt-5.4`, `gemini`); all keys when omitted |
| `--to <DEST>` | Output directory; defaults to the Agenvoy path |
| `--download-only` | Stop after download without extracting |
| `--extract-only` | Skip download and extract from the local archive |
| `--base-only` | Run Phase B only and leave model files untouched |
| `--skip-base` | Skip Phase B; the default whenever `<key>` is given |

### Source Registry

| Vendor | key | Source |
|--------|-----|--------|
| Anthropic | `claude` (overview) | `platform.claude.com/.../claude-prompting-best-practices.md` |
| Anthropic | `claude-<model>` | `prompting-claude-<model>.md` on the same path; URL taken from the overview's model-specific guidance table |
| OpenAI | `gpt-<ver>` | `developers.openai.com/api/docs/guides/latest-model/gpt-<ver>.md` |
| Google | `gemini` | `ai.google.dev/gemini-api/docs/prompting-strategies` |

When a newer family page replaces an older model's URL, the skill falls back to Wayback and records the snapshot timestamp in the archive's first line.

### Output Files

| File | Budget | Injected when |
|------|--------|---------------|
| `_base.md` | 25 lines / 4 sections | Always |
| `_vendor_<vendor>.md` | 35 lines / 5 sections | The model name contains the vendor name |
| `<key>.md` | 55 lines / 6 sections | The model matches (longest key wins, one file only) |
| `_base_unlisted.md` | 25 lines / 4 sections | Neither a model file nor a vendor file matches |

### Heading Vocabulary

Only these headings, in this order: `Acting`, `Scope`, `Instructions`, `Tools`, `Delegation`, `Grounding`, `Verification`, `Review`, `Long context`, `Long horizon`, `Progress`, `Output`, `Code`, `Other`.

### Deletion Reasons

| Reason | Criterion |
|--------|-----------|
| Duplicates `_base.md` | Same rule verbatim or near-verbatim; keep versions with added nuance |
| Duplicates or contradicts the shared layer | `system_prompt.md` §Behavioral Constraints wins |
| Contradicts a local contract | The local contract wins |
| Self-contradiction within the file | Keep the more specific rule |
| Thinking guidance | Sentences about how hard or how deeply to think |
| Process steps | Step-by-step SOPs |
| How to use a tool | Moves to that tool's description |
| Prompt-writing advice | Teaches developers to write prompts rather than steering the model |
| Named examples | Named fonts, companies, version numbers |

### Archived Keys

| Vendor | key |
|--------|-----|
| Anthropic | `claude`, `claude-fable-5`, `claude-fable-5-1`, `claude-opus-4-8`, `claude-opus-5`, `claude-opus-5-5`, `claude-sonnet-5` |
| OpenAI | `gpt-4.1`, `gpt-5`, `gpt-5.1`, `gpt-5.2`, `gpt-5.3`, `gpt-5.4`, `gpt-5.5`, `gpt-5.6`, `gpt-6`, `gpt-6-astra` |
| Google | `gemini` |
