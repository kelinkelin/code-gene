# Code Gene Skill

Reusable Codex skill for verification-first code generation discipline.

## Install From GitHub

After this repository is uploaded to GitHub, ask Codex:

```text
Install the code-gene skill from https://github.com/<your-user>/code-gene-skill/tree/main/skills/code-gene
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
    └── agents/
        └── openai.yaml
```

