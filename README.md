# gitp

Git + prompt

A minimal protocol for attaching prompts to commits. See [gitp/PROTOCOL.md](gitp/PROTOCOL.md).

## Quick start

```bash
# Add bin/ to PATH, or symlink gitp into your PATH
export PATH="$PWD/bin:$PATH"

gitp commit -p "Add a README with quick start" -m "docs: add gitp quick start"
gitp prompts          # show prompts for HEAD
gitp log --prompts    # log with prompts
```

Prompts are stored in `.git/gitp/prompts/<sha>.json` and do not alter commit objects.
