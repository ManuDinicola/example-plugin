---
name: security-review
description: Use when reviewing code for security vulnerabilities such as SQL injection, cross-site scripting (XSS), authentication or authorization flaws, insecure data handling, or hardcoded secrets. Reports findings by severity with file, line and a recommended fix.
allowed-tools: Read, Grep
model: sonnet
---

# Security review

Review the code the user points at for security issues. This is a review only: report findings and do not edit files.

If the user names no files or folders, start from the entry points (routes, controllers, handlers, auth code, data access) and search outward with Grep.

## Common vulnerabilities

- SQL injection risks
- Cross-site scripting (XSS)
- Authentication and authorization issues
- Insecure data handling
- Hardcoded secrets or credentials

## Security best practices

- Input validation is present
- Output encoding is correct
- Secure defaults are used
- Error messages are safe
- Logging is appropriate (no sensitive data)

## Report

For each issue found, give:

- File and line number
- Severity (Critical, High, Medium or Low)
- Description of the vulnerability
- Recommended fix

Order the issues by severity, most severe first. If nothing is found, say which areas you checked.

## Sharing the findings

Findings are written in the conversation by default. If the user asks to post them as an Azure DevOps pull request comment or a Jira comment, keep the 🤖 prefix on the first line, as the hooks in this plugin remind you to.
