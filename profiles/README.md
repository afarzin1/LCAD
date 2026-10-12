# Profiles

A profile is a reusable starting point for a machine project: its **conventions** (naming,
abbreviations, wire labels, numbering, package structure) and its **sourcing policy**. Creating a
project copies one profile into the project's `profile/` folder; from then on the project owns its
copy. See [`docs/conventions.md`](../docs/conventions.md).

| Profile | Owner | Notes |
|---|---|---|
| [`author/`](author/) | LCAD author | RIO/FIO/CIO I/O classes, numeric wire labels, cover → exterior → interior → power → I/O per card → misc; McMaster/Radwell/Saginaw-Rittal/Icotek sourcing |

To add a profile, copy `author/` to a new folder and edit it. Profiles contain preferences, never
customer data, prices or quotes (rule P7).
