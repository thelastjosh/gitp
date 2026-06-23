# gitp — git + prompt

**Version:** 0.1

gitp is a minimal extension to git. Every commit may carry one or more prompts — the natural-language instructions that produced the change. Standard git workflows are unchanged; gitp only adds optional metadata keyed by commit OID.

## Principles

1. **Git-compatible.** A gitp repository is a normal git repository. Any git client works without modification.
2. **Opt-in.** Commits without prompts behave exactly like plain git commits.
3. **Append-only metadata.** Prompts are attached after (or at) commit time and keyed by the resulting commit SHA.
4. **Local-first.** Prompt storage lives under `.git/gitp/`. Nothing is rewritten into commit objects or messages.

## Storage

```
.git/gitp/
  prompts/
    <40-char-hex-sha>.json
```

Each file corresponds to one commit. Filename is the full lowercase hex OID of the commit.

### Prompt record (JSON)

```json
{
  "version": 1,
  "commit": "a1b2c3d4e5f6...",
  "prompts": [
    {
      "text": "Add a login form with email and password fields",
      "at": "2026-06-23T12:00:00Z"
    }
  ]
}
```

| Field | Required | Description |
|-------|----------|-------------|
| `version` | yes | Schema version. Currently `1`. |
| `commit` | yes | Full 40-char hex OID. Must match the filename. |
| `prompts` | yes | Ordered list of prompt entries. |
| `prompts[].text` | yes | The prompt text. |
| `prompts[].at` | no | ISO-8601 UTC timestamp of when the prompt was recorded. |

Multiple prompts on one commit represent a sequence of instructions (e.g. follow-up refinements in the same session).

## Operations

### Attach at commit time

When creating a commit, record one or more prompts alongside it:

```
gitp commit -p "Add login form" -m "feat: add login"
gitp commit -p "prompt one" -p "prompt two" -m "fix: validation"
GITP_PROMPT="from the environment" gitp commit -m "chore: cleanup"
```

Equivalent to `git commit` for all unrecognized flags. After a successful commit, prompts are written to `.git/gitp/prompts/<new-sha>.json`.

### Attach to an existing commit

```
gitp attach <sha> -p "retroactive prompt"
```

Appends to any existing prompt record for that commit.

### Read prompts

```
gitp prompts              # prompts for HEAD
gitp prompts <sha>        # prompts for a specific commit
gitp log --prompts        # git log with prompts appended per commit
```

### Pass-through

Any subcommand not implemented by gitp is forwarded to git:

```
gitp status
gitp diff
gitp push
```

## Environment

| Variable | Description |
|----------|-------------|
| `GITP_PROMPT` | Default prompt text for the next `gitp commit` (single prompt). |

## Distribution

Prompt files are **not** part of the git object database. They are not pushed or pulled by default `git push` / `git fetch`.

To share prompts across clones, copy or sync `.git/gitp/prompts/` explicitly, or use a future transport (e.g. a dedicated ref or git notes namespace). v0.1 leaves transport unspecified.

## Compatibility

- Implementations MUST NOT modify commit objects, trees, or blobs.
- Implementations MUST NOT require prompts for any git operation.
- Unknown fields in prompt records SHOULD be preserved on read/write for forward compatibility.
