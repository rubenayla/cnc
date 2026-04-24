<!-- reference — read when relevant -->

# resources/

Binary assets for this project. Keep this separate from `docs/` (which is markdown only) so `docs/` can stay a clean pool of short, greppable text.

```
resources/
├── photos/          on-site photos
└── manuals/         vendor PDFs (Fagor, Holke, INTZA, WinDNC)
```

See each subfolder's `README.md` for what lives there and the naming convention.

## What doesn't belong here

- Markdown notes → `docs/` or `.agents/notes.md`
- Debugging session evidence (WinDNC logs, dated screenshots, "what we tried") → `logs/YYYY-MM-DD_*/`
- Temp scratch files → `/tmp/`, never the repo
