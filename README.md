# example-plugin

Example Claude Code plugin showing how a team can ship MCP servers, hooks and skills together. Pull requests live in Azure DevOps, work items live in Jira.

## Layout

```
example-plugin/
├── .claude-plugin/
│   ├── plugin.json        # plugin manifest (name, version, author)
│   └── marketplace.json   # lets people install the plugin straight from this repo
├── .mcp.json              # Azure DevOps and Atlassian (Jira) MCP servers
├── hooks/
│   └── hooks.json         # PreToolUse hooks that inject team rules
├── skills/
│   ├── pr-summary/
│   │   └── SKILL.md       # how to write PR titles and descriptions
│   └── security-review/
│       └── SKILL.md       # security review of code, findings by severity
└── README.md
```

## What it does

- **MCP servers**
  - `azure-devops`: pull requests, PR comments and wiki. It signs in with a personal access token (PAT) read from the `AZURE_DEVOPS_PAT` environment variable, and takes the organisation from `AZURE_DEVOPS_ORG`. Nothing in the plugin needs editing.
  - `atlassian`: Jira issues and comments, through the Atlassian remote MCP server. You sign in with your Atlassian account the first time it is used.
- **Hook 1 (all write tools)**: before PR, PR comment, wiki, Jira issue and Jira comment writes, reminds Claude to prefix the first line with 🤖 so reviewers can tell the content was agent-authored.
- **Hook 2 (PR tools only)**: tells Claude that the PR title must contain the Jira key (for example `[PROJ-123]`), to use the `example-plugin:pr-summary` skill, to leave out a Test plan section, and to post manual test steps as a comment on the Jira issue instead.
- **Skill `pr-summary`**: the canonical PR title and description conventions.
- **Skill `security-review`**: reviews code for vulnerabilities (injection, XSS, auth flaws, insecure data handling, hardcoded secrets) and reports each finding with file, line, severity and a recommended fix. Claude can load it on its own when you ask for a security review, or you can run it as `/example-plugin:security-review`.

## How to use it

The plugin is installed straight from this repository: <https://github.com/ManuDinicola/example-plugin>. You do not need to clone it.

### Prerequisites

- Claude Code, installed and signed in.
- Node.js, because the Azure DevOps MCP server is started with `npx`.
- An Azure DevOps organisation and a personal access token (PAT).
- A Jira site and an Atlassian account.

### 1. Set the environment variables

The Azure DevOps server reads two environment variables when Claude Code starts, so set them before you launch it:

| Variable | Value |
| :- | :- |
| `AZURE_DEVOPS_ORG` | Your organisation name, the `<org>` part of `https://dev.azure.com/<org>` |
| `AZURE_DEVOPS_PAT` | Your personal access token |

**The organisation must be configured**, but when you launch Claude Code from the team's project repository it is already done. The repository's committed `.claude/settings.json` can set it, because an organisation name is not a secret:

```json
{
  "env": {
    "AZURE_DEVOPS_ORG": "your-org"
  }
}
```

With that file in the repository, you only set your own `AZURE_DEVOPS_PAT`, and the plugin can always be added there without cloning anything or editing any file. Claude Code applies most `env` values from a project settings file only after you trust the folder. Run `/mcp` to confirm: if `azure-devops` still shows a missing-variable warning for `AZURE_DEVOPS_ORG`, export the variable in your shell as shown below. Outside that repository, set both variables yourself.

The PAT is always personal and never goes in a committed file.

Create the PAT in Azure DevOps under **User settings > Personal access tokens > New token**. Grant only the scopes the tools you use need, for example Code (Read & write) for pull requests, Wiki (Read & write) for wiki pages and Project and Team (Read) for listing projects.

On macOS or Linux, add the variables to your shell profile (`~/.zshrc` or `~/.bashrc`):

```bash
export AZURE_DEVOPS_ORG="your-org"
export AZURE_DEVOPS_PAT="your-personal-access-token"
```

On Windows, in PowerShell, then open a new terminal:

```powershell
setx AZURE_DEVOPS_ORG "your-org"
setx AZURE_DEVOPS_PAT "your-personal-access-token"
```

Keep the PAT out of git. Never put it in this repository or in a project's committed `.claude/settings.json`.

### 2. Add the marketplace and install the plugin

This repository is also a plugin marketplace (see `.claude-plugin/marketplace.json`). In a Claude Code session, run:

```text
/plugin marketplace add https://github.com/ManuDinicola/example-plugin.git
/plugin install example-plugin@example-marketplace
```

The first command registers the marketplace from the repository's git URL. You can also write the shorthand `ManuDinicola/example-plugin`, which Claude Code reads as a GitHub repository, but the full URL makes the source explicit and works the same way for any git host. The second command opens the plugin's details. Choose a scope (user scope is fine) and confirm. The same thing works from your shell:

```bash
claude plugin marketplace add https://github.com/ManuDinicola/example-plugin.git
claude plugin install example-plugin@example-marketplace
```

To enable it for everyone working in one project repository, install it with `--scope project` and commit the `.claude/settings.json` change it makes. Each teammate still runs the install once on their own machine.

The first time the plugin's MCP servers start, Claude Code asks you to approve them. If the summary says `Run /reload-plugins to activate`, run that command in the session.

### 3. Check that it loaded

- Run `claude plugin list` in your shell, or open `/plugin` and the **Installed** tab. `example-plugin@example-marketplace` should be enabled. The **Errors** tab shows anything that failed to load.
- Type `/` and look for `/example-plugin:pr-summary` and `/example-plugin:security-review`.
- Run `/mcp`. The `azure-devops` and `atlassian` servers from the plugin should be listed. A missing-variable warning on `azure-devops` means `AZURE_DEVOPS_ORG` or `AZURE_DEVOPS_PAT` was not set when Claude Code started. Sign in to the Atlassian server when it asks.

### 4. Try it

1. Check out a branch whose name contains a Jira key, for example `feature/PROJ-123-invoice-retry`, and make a change.
2. Ask Claude: `Create a pull request for this branch`.
3. Before the pull request is created, the hooks remind Claude of the team rules. You should see:
   - a title in the form `🤖 [PROJ-123] Add retry to invoice export`
   - a description that follows the `pr-summary` sections, with no Test plan section
   - manual test steps posted as a comment on the Jira issue instead of in the PR body

The hooks only add reminders, so if Claude misses a rule, tell it and it will fix the PR.

### 5. Run a security review

Run the skill directly, or just ask for a security review and Claude loads it:

```text
/example-plugin:security-review src/
```

Claude reports each finding with file and line, severity, a description and a recommended fix, most severe first. The skill only reports and does not change files. Its `allowed-tools: Read, Grep` line pre-approves those two tools for the turn, it does not block other tools, so the "report only" rule comes from the skill's instructions.

### Update the plugin

Marketplaces added from GitHub do not update automatically by default. In a session, open `/plugin`, go to the **Marketplaces** tab, select `example-marketplace` and choose **Update marketplace**, or turn on auto-update there. From your shell:

```bash
claude plugin update example-plugin@example-marketplace
```

Then run `/reload-plugins` in a running session, or restart Claude Code.

### Troubleshooting

- **Plugin not listed**: check the **Errors** tab in `/plugin`. If the marketplace add fails, make sure `.claude-plugin/marketplace.json` is pushed to the default branch of the repository.
- **Azure DevOps server not connected**: run `/mcp` to see its status. Check that Node.js is installed and that both environment variables are set in the shell you started Claude Code from.
- **Azure DevOps rejects the PAT**: `.mcp.json` starts the server with `-a envvar`, which the Azure DevOps MCP documentation describes as reading a bearer token from `ADO_MCP_AUTH_TOKEN`. If your PAT is refused, use the server's PAT mode instead: change the arguments to `"-a", "pat"`, replace the `env` entry with `"PERSONAL_ACCESS_TOKEN": "${AZURE_DEVOPS_PAT}"`, and set `AZURE_DEVOPS_PAT` to the base64 encoding of `<email>:<pat>`:

  ```bash
  export AZURE_DEVOPS_PAT="$(printf '%s' '<email>:<pat>' | base64)"
  ```

- **Skill missing**: `skills/` must sit at the plugin root, next to `.claude-plugin/`, not inside it.
- **Rules not applied**: the hook matchers contain the plugin and server names. Make sure the MCP tools are named `mcp__plugin_example_plugin_<server>__<tool>` (see Naming below), and update `hooks/hooks.json` if you renamed anything.

### Working on the plugin (clone)

You only need a clone to change the plugin. Load your local copy for one session with:

```bash
git clone https://github.com/ManuDinicola/example-plugin.git
claude plugin validate ./example-plugin
claude --plugin-dir ./example-plugin
```

Run `/reload-plugins` in the session after each edit.

## Naming

MCP tools from a plugin are exposed as `mcp__plugin_<plugin-name>_<server-name>__<tool>`. The Claude Code documentation says non-alphanumeric characters in the plugin and server names are replaced by `_`, so the tools in this plugin are named, for example:

- `mcp__plugin_example_plugin_azure_devops__repo_create_pull_request`
- `mcp__plugin_example_plugin_atlassian__addCommentToJiraIssue`

The hook matchers in `hooks/hooks.json` accept either a hyphen or an underscore in those names (`example[-_]plugin`, `azure[-_]devops`), so they match whichever form your Claude Code version uses. If you rename the plugin or a server, update the `matcher` regexes to match.

## Notes

- The hooks only add context. They do not block or rewrite tool calls, so compliance is up to Claude.
- To enforce a rule instead (for example a title that must match `^\[[A-Z]+-\d+\]`), point the hook `command` at a script that reads the tool input from stdin and returns a deny decision. An Azure DevOps branch policy can also check PR titles on the server side.
