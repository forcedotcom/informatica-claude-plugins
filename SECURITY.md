## Security

Please report any security issue to [https://www.sfdc.co/SubmitVuln](https://www.sfdc.co/SubmitVuln)
as soon as it is discovered. This is Salesforce's standard responsible-disclosure
intake and applies across Salesforce open source projects, including this one.

This repo has no runtime dependencies to speak of — it is a Claude Code plugin
container consisting of JSON manifests, MCP server configuration, and Markdown
skill definitions (see [`CLAUDE.md`](CLAUDE.md)). The main security-relevant
surface is the `.mcp.json` MCP server declarations inside each plugin
(`informatica-in-claude/.mcp.json` at the time of writing) — review any
change to a server URL or credential/auth configuration with extra care, and
flag it through the link above if you believe it introduces a vulnerability.
