---
name: pr-summary
description: Use when writing or updating the title or description of a pull request in Azure DevOps. Produces a concise, reviewer-friendly summary that starts with the Jira issue key, following Example Team conventions.
---

# PR summary

Pull requests live in Azure DevOps, work items live in Jira. Write the PR title and description from the diff and the linked Jira issue.

## Title

- Must contain the Jira issue key in square brackets: `[PROJ-123] Add retry to invoice export`.
- Imperative mood, under 72 characters including the key ("Add retry", not "Added retries").
- Find the key in this order: the branch name (for example `feature/PROJ-123-invoice-retry`), the commit messages, then the user. If none gives a key, ask. Never invent one.
- One key in the title. If the PR touches several issues, put the others under "Linked work".
- When the agent-authorship prefix applies, the title reads `🤖 [PROJ-123] Add retry to invoice export`.

## Description

Use these sections, and skip any that do not apply:

1. **Summary**: one or two sentences on what changed and why.
2. **Changes**: short bullets grouped by area (API, UI, database, config).
3. **Risks / rollout**: migrations, feature flags, breaking changes, anything reviewers should watch.
4. **Linked work**: the Jira issue link, for example `https://your-domain.atlassian.net/browse/PROJ-123`.

## Rules

- Describe the intent, not a line-by-line recap of the diff.
- Do NOT add a Test plan or Testing section. Tests in the diff are the regression contract.
- Manual verification steps for testers go in a comment on the linked Jira issue, not in the PR body.
- The hooks in this plugin add the agent-authorship prefix (🤖) to the first line. Keep it when editing existing agent-authored content.
