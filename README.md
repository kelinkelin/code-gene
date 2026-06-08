# code-gene

Reusable Codex skill for verification-first code generation discipline.

## Install From GitHub

Ask Codex:

```text
Install the code-gene skill from https://github.com/kelinkelin/code-gene/tree/main/skills/code-gene
```

Restart Codex after installation so the skill is discovered.

## Manual Install

Clone the repository, then copy the skill folder into your Codex skills directory:

```bash
mkdir -p "${CODEX_HOME:-$HOME/.codex}/skills"
cp -R skills/code-gene "${CODEX_HOME:-$HOME/.codex}/skills/"
```

Restart Codex, then invoke it with:

```text
/code-gene
```

or:

```text
use code-gene
```

## Repository Layout

```text
skills/
└── code-gene/
    ├── SKILL.md
    ├── references/
    │   ├── code-style.md
    │   ├── delivery-tooling.md
    │   ├── java-backend.md
    │   ├── logging.md
    │   └── review.md
    └── agents/
        └── openai.yaml
```
