# EthosAI™ Brand Status

## Current Status

EthosAI™ is a trademark application pending before the German Patent and
Trade Mark Office (DPMA). The repository records the filing date as
2026-03-23. Until the official registration is available and approved,
all public-facing materials must use `EthosAI™` or the fallback
`EthosAI (trademark application pending)`.

## Public Notice

Use this notice in public README footers, package metadata, NOTICE files,
and mirror repositories:

```text
EthosAI™ is a trademark application pending before the German Patent and Trade Mark Office.
The EthosAI™ trademark is owned by Robert Alexander Massinger.
This open-source license does not grant trademark rights.
```

## Registered-Symbol Approval Gate

The `®` symbol and public claims that the mark is already registered are
blocked while the status is `application-pending`.

Before any public release may switch from `™` to `®`, all of the following
must exist and be reviewed:

- Official DPMA registration proof PDF.
- SHA-256 proof file for the PDF.
- Legal/compliance review evidence.
- Product owner approval evidence.
- `doc/legal/trademark/status.json` updated from `application-pending` to
  `registered` in the same reviewed change.

The current configured evidence paths are listed in
`doc/legal/trademark/status.json`. Do not create placeholder evidence for
this gate; absence of the official PDF means the public mark remains `™`.

## Automation

Run the trademark status gate before public mirror release work:

```powershell
.venv\Scripts\python scripts\check_trademark_status.py
```

The public-repos dry run also calls this gate for every mirror.
