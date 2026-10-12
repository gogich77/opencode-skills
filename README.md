# opencode-skills

HTTP skill catalog for [OpenCode](https://opencode.ai). Every OpenCode instance
that points at this repo gets the same skills.

## Catalog layout

```
skills/
├── index.json            <- catalog manifest (read by OpenCode)
└── mermaid/
    └── mermaid.md        <- skill content, flat file name = skill ID
```

OpenCode downloads each file from `<base-url>/<name>/<file>`, so the catalog
base URL is:

```
https://raw.githubusercontent.com/<YOUR-USER>/opencode-skills/main/skills/
```

(Replace `<YOUR-USER>` with your GitHub user/org. If your default branch is
`master`, use `master` instead of `main`.)

## Use it in an OpenCode instance

Add the catalog URL to the `skills` array in `opencode.json`
(global `~/.config/opencode/opencode.json` or a project `opencode.json`):

```json
{
  "$schema": "https://opencode.ai/config.json",
  "skills": [
    "https://raw.githubusercontent.com/<YOUR-USER>/opencode-skills/main/skills/"
  ]
}
```

Restart OpenCode (or start a new session). The `mermaid` skill is then loaded
on demand, exactly like a local skill.

## Add a new skill

1. Create `skills/<name>/<name>.md` with YAML frontmatter (`name`, `description`).
2. Add an entry to `skills/index.json`.
3. Commit and push.

## Update an existing skill

1. Edit the file.
2. **Bump its `version` in `index.json`** — OpenCode caches by version and
   will not refetch a skill whose version did not change.
3. Commit and push.

## Notes

- Flat file name `mermaid.md` gives the skill the ID `mermaid`. A file named
  `SKILL.md` in a catalog gets the ID `SKILL`, so always use `<name>.md`.
- Paths in `files` must be relative and same-origin (they are, here).
- `raw.githubusercontent.com` serves these files publicly; keep secrets out
  of this repo.
