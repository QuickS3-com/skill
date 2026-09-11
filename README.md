# QuickS3 agent plugin

Lets coding agents browse and transfer files through [QuickS3](https://quicks3.com). Agents act only with the QuickS3 roles you delegate during OAuth consent, and file bytes go directly between the agent and your storage provider.

The plugin bundles:

- the QuickS3 remote MCP server (`https://quicks3.com/mcp`, Streamable HTTP with OAuth)
- the `quicks3-operator` skill, which teaches the agent to list narrowly, download through single-use links, and upload without overwriting by default

## Install in Codex

```bash
codex plugin marketplace add https://github.com/QuickS3-com/skill.git
```

```bash
codex plugin add quicks3@quicks3
```

Codex opens a browser for QuickS3 sign-in. Choose the roles to delegate; you can revoke the grant at any time in QuickS3.

## Other MCP clients

Any client that supports remote MCP servers with OAuth can connect to `https://quicks3.com/mcp` directly. The skill in [`plugins/quicks3/skills/quicks3-operator`](plugins/quicks3/skills/quicks3-operator) is a plain `SKILL.md` and can be copied into any agent that reads skills.

## Tools

| Tool | Effect |
| --- | --- |
| `list_connections` | Storage connections the grant can see |
| `list_buckets` | Visible buckets of one connection |
| `list_objects` | One folder level, paginated |
| `create_download_link` | Single-use link, valid 5 minutes |
| `create_upload_url` | Presigned `PUT` URL, valid 15 minutes, no overwrite by default |

See [`tool-contract.md`](plugins/quicks3/skills/quicks3-operator/references/tool-contract.md) for schemas and errors.
