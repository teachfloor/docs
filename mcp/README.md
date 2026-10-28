# Teachfloor MCP Server

Connect Teachfloor to AI assistants.

## What Is It?

The Teachfloor MCP server exposes your organization's courses, members, teams and activity data to an AI assistant as a set of tools. Once connected, you can ask the assistant to do things like *"list the learners who haven't started the onboarding course"* or *"add a module with three lessons to the compliance course"*, and it calls Teachfloor on your behalf.

It speaks the [Model Context Protocol](https://modelcontextprotocol.io), an open standard, so it is not tied to one assistant.

**Endpoint:** `https://mcp.teachfloor.com`

## How It Works

A client connects to the endpoint, discovers the tools it offers, and calls them as the conversation needs them. Each tool is a single operation — read a course, create a module, enroll a member — and the assistant picks the ones it needs.

Access is per organization and per person: a token reaches only what that member can reach in Teachfloor.

### Common Use Cases

- **Ask questions about your data** - Progress, enrollments, completions, without building a report
- **Build course content** - Draft modules and elements from an outline
- **Automate admin work** - Enroll a team, update member fields, revoke access

## Supported Clients

| Client | |
|---|---|
| [Claude](./mcp/clients/claude) | Available as a custom connector |
| [Custom clients](./mcp/clients/custom) | Any MCP client, or your own script |

## Quick Links

- [Getting Started](./mcp/getting-started) - Authentication, scopes and protocol, common to every client
- [Tools Reference](./mcp/tools-reference) - Every tool the server offers

## Requirements

A plan that includes MCP. Requests from an organization without it are answered with `403 Forbidden`.

## Need Help?

If you have questions or need assistance, contact support through your Teachfloor dashboard.
