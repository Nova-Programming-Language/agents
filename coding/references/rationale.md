# Why the Coding Rules Work This Way

Reasons behind rules in `../SKILL.md`. The rules themselves live there.

- **Names are the primary documentation.** A reader learns what code does
  from its names before its comments.
- **A refactor should reduce net lines.** A refactor that adds more code than it
  removes is adding complexity, not reducing it.
- **No formatting churn.** Reformatting lines you did not otherwise modify
  creates diff churn that obscures real changes and pollutes git blame.
- **Search for placeholders, then read every branch.** Renamed or indirect
  placeholders evade text search, so grep alone cannot prove completeness.
