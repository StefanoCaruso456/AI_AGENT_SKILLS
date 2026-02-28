# Versioning

This repository follows semantic commit messages. Skills are versioned by git history — there are no separate version numbers per skill.

## Commit Conventions

| Prefix | Use |
|---|---|
| `feat(skill-name)` | New skill or new reference file |
| `fix(skill-name)` | Correct schema, contracts, or templates |
| `docs` | README, VERSIONING, or non-skill documentation |
| `refactor(skill-name)` | Restructure without changing behavior |

## Schema Alignment

`natural-language-sql-builder/references/schema.md` must stay in sync with `Ghostfolio/prisma/schema.prisma`. When the Prisma schema changes, update schema.md and bump the commit.
