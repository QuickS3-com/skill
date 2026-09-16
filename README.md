# quickS3 agent plugin

Lets coding agents browse, transfer, and share files through [quickS3](https://quicks3.com). Agents act only with the quickS3 roles you delegate during OAuth consent, and file bytes go directly between the agent and your storage provider.

This repository contains the client plugin and skill. The MCP server itself is hosted by quickS3 at `https://quicks3.com/mcp`; its source is not in this repository.

## Requirements

- A quickS3 organization with at least one storage connection. [Sign up at quicks3.com](https://quicks3.com).
- A quickS3 role that gives you access to the buckets or folders the agent should use.

Supported providers: AWS S3, Cloudflare R2, Backblaze B2, DigitalOcean Spaces, Wasabi, MinIO, Azure Blob Storage, and other S3-compatible services.

The plugin bundles:

- the quickS3 hosted MCP server (`https://quicks3.com/mcp`, Streamable HTTP with OAuth)
- the `quicks3-operator` skill, which teaches the agent to list narrowly, download through single-use links, share files only when asked, and upload without overwriting by default

## Install in Claude Code

In Claude Code, run:

```
/plugin marketplace add QuickS3-com/skill
/plugin install quicks3@quicks3
```

Or from a shell:

```bash
claude plugin marketplace add QuickS3-com/skill
```

```bash
claude plugin install quicks3@quicks3
```

Run `/mcp` in Claude Code and choose `QuickS3` to sign in. Choose the roles to delegate; you can revoke the grant at any time in quickS3.

## Claude.ai and Claude Desktop

1. In Claude, open **Settings → Connectors** and choose **Add custom connector**.
2. Enter `https://quicks3.com/mcp` as the URL and complete the quickS3 sign-in.
3. Optional: to add the skill, zip the [`quicks3-operator`](plugins/quicks3/skills/quicks3-operator) folder and upload it in **Settings → Capabilities → Skills**.

## ChatGPT

1. In ChatGPT's settings, add a custom connector.
2. Enter `https://quicks3.com/mcp` as the URL and choose OAuth as the authentication.
3. ChatGPT opens quickS3 so you can approve the connector and choose its roles.

## Install in Codex

```bash
codex plugin marketplace add https://github.com/QuickS3-com/skill.git
```

```bash
codex plugin add quicks3@quicks3
```

Codex opens a browser for quickS3 sign-in. Choose the roles to delegate; you can revoke the grant at any time in quickS3.

## Other MCP clients

Any client that supports remote MCP servers with OAuth can connect to `https://quicks3.com/mcp` directly. The skill in [`plugins/quicks3/skills/quicks3-operator`](plugins/quicks3/skills/quicks3-operator) is a plain `SKILL.md` and can be copied into any agent that reads skills.

## Tools

| Tool | Effect |
| --- | --- |
| `list_connections` | Storage connections the grant can see |
| `list_buckets` | Visible buckets of one connection |
| `list_objects` | One folder level, paginated |
| `create_download_link` | Single-use link, valid 5 minutes |
| `create_share_link` | Public share link for someone else, valid 5 minutes to 30 days (default 24 hours) |
| `create_upload_url` | Presigned `PUT` URL, valid 15 minutes, no overwrite by default |

See [`tool-contract.md`](plugins/quicks3/skills/quicks3-operator/references/tool-contract.md) for schemas and errors.

## Learn more

- [Setup guide](https://quicks3.com/s3-mcp-server/)
- [Security model](https://quicks3.com/docs/security/model/)
- [Privacy policy](https://quicks3.com/privacy/)
