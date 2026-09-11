# QuickS3 MCP Tool Contract

Use this reference when constructing tool calls, interpreting structured results, or troubleshooting a failed QuickS3 operation.

## Server

- MCP server identifier: `QuickS3`
- Transport: Streamable HTTP
- Endpoint: `https://quicks3.com/mcp`
- Authentication: Browser-based OAuth with a QuickS3 access grant

The access grant delegates selected roles the user already holds. It does not expose provider credentials. Role removal, permission changes, membership changes, grant revocation, and grant expiry affect subsequent calls.

## `list_connections`

Read-only. Input:

```json
{}
```

Structured result:

```json
{
  "connections": [
    {
      "id": "conn_...",
      "name": "Customer files",
      "provider": "r2",
      "canRead": true,
      "canWrite": false
    }
  ]
}
```

Only connections with an effective read or write allowance somewhere are returned. Reuse the ID; later tools do not accept the display name in its place.

## `list_buckets`

Read-only. Input:

```json
{ "connectionId": "conn_..." }
```

Structured result:

```json
{ "buckets": ["documents", "media"] }
```

The current implementation returns bucket names as strings. Only visible buckets allowed by both the connection restriction and delegated permissions are returned. An empty array is a successful result.

## `list_objects`

Read-only. Input:

```json
{
  "connectionId": "conn_...",
  "bucket": "documents",
  "prefix": "approved/",
  "continuationToken": null,
  "limit": 100
}
```

`prefix`, `continuationToken`, and `limit` are optional. Omit `prefix` for the root. Limit defaults to 100 and cannot exceed 500.

Structured result:

```json
{
  "objects": [
    {
      "key": "approved/report.pdf",
      "kind": "object",
      "size": 18420,
      "lastModified": "2026-09-11T08:00:00.000Z"
    },
    {
      "key": "approved/archive/",
      "kind": "prefix",
      "size": 0
    }
  ],
  "continuationToken": null
}
```

QuickS3 always requests provider listings with delimiter `/`, so results describe only the immediate level. It merges provider common prefixes into `objects`, removes the listed folder's own marker, filters denied keys, and can append synthetic prefix entries that lead to deeper allowed scopes.

Folder keys are full prefixes, not names relative to the requested prefix. Preserve the full key in subsequent calls. Use a trailing slash to list inside a folder.

Continuation tokens are provider-issued opaque strings. Pass a returned token back with the same connection, bucket, prefix, and limit to obtain the next page.

## `create_download_link`

Read-only with respect to storage. Input:

```json
{
  "connectionId": "conn_...",
  "bucket": "documents",
  "key": "approved/report.pdf"
}
```

The content includes a `resource_link` to a QuickS3 transfer URL. Structured metadata contains:

```json
{
  "transferId": "xfer_...",
  "expiresAt": "2026-09-11T08:05:00.000Z"
}
```

The QuickS3 transfer link lasts five minutes and permits one successful redemption. Redemption rechecks authorization, creates a provider URL lasting about one minute, consumes the handle, and redirects to the provider. Follow redirects. A fresh link is required after successful redemption.

Common redemption responses:

- `403 forbidden`: the grant or current permissions no longer allow the object.
- `410 not_found`: the handle is unknown, or its connection was deleted after the link was issued.
- `410 expired_or_used`: the handle expired or was already redeemed.
- `410 signing_failed`: the provider URL could not be created; this failure does not consume the handle.
- Public `404 not_found`: malformed routing or an internal transfer lookup failure was deliberately concealed.

## `create_upload_url`

Write operation. Input:

```json
{
  "connectionId": "conn_...",
  "bucket": "documents",
  "key": "reports/2026/q3.csv",
  "size": 18420,
  "contentType": "text/csv",
  "overwrite": false
}
```

`size` is required and must be a non-negative safe integer. `contentType` and `overwrite` are optional; overwrite defaults to false.

Structured result:

```json
{
  "url": "https://storage-provider.example/presigned-secret",
  "method": "PUT",
  "headers": {},
  "expiresAt": "2026-09-11T08:15:00.000Z",
  "maxBytes": 5368709120
}
```

The URL lasts 15 minutes. Send the bytes directly to the provider with `PUT`. Include every returned header; Azure Blob uploads currently require `x-ms-blob-type: BlockBlob`. Use a file-backed, known-length body. The URL and any provider error body may contain sensitive details and should not be repeated to the user.

With `overwrite: false`, QuickS3 checks for an existing object before signing. If the provider refuses that check with 403 (for example, a write-only role), QuickS3 proceeds without it, so an existing object can still be replaced. When replacement matters and the connection has `canRead: false`, tell the user the check may be skipped.

The current MCP server has no multipart tools registered. If a file exceeds `maxBytes`, report that limitation rather than attempting nonexistent tools.

## Tool errors

Tool failures set `isError: true` and include a short message. Respond according to the message without probing beyond the requested scope.

- Access denied: selected roles do not grant the operation, or the grant was revoked or expired.
- Connection does not exist: refresh connections once if the ID came from stale context; otherwise report it.
- Missing bucket or key: correct the call using user-provided or freshly listed values.
- Invalid upload size: measure the file again; do not estimate.
- Object exists: keep `overwrite: false` until the user authorizes replacement.
- Upload too large: no multipart MCP tools are currently available.
- Download links temporarily unavailable: server public-origin configuration is unavailable; retrying the same call repeatedly is not useful.

Provider failures are intentionally summarized. Do not ask QuickS3 to reveal credentials, endpoints, signatures, stack traces, or raw provider error bodies.
