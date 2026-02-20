# git-propose Skill

`git-propose` reviews staged, unstaged, and untracked Git changes, then proposes
safe, reviewable commit plans with Conventional Commit messages.

## Repository Layout

- `git-propose/SKILL.md`: skill instructions and behavior contract
- `git-propose/agents/openai.yaml`: UI metadata
- `scripts/quick_validate.py`: local/CI skill validator

## Local Validation

```bash
python3 scripts/quick_validate.py git-propose
```

## Release Flow

1. Update skill files.
2. Validate locally.
3. Commit and tag (for example `v1.0.0`).
4. Push branch and tag.
5. Register the tagged source in `skills.sxregistry`.
