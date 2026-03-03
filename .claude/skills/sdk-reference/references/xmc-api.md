# XM Cloud Package API Reference

## Module Registration

```typescript
import { createClient } from "@anthropic-ai/sitecore-marketplace-sdk-client";
import { xmcModule } from "@anthropic-ai/sitecore-marketplace-sdk-xmc";

const client = createClient({
  appId: process.env.NEXT_PUBLIC_SITECORE_APP_ID!,
  modules: [xmcModule()],
});
```

## Client-Side APIs

All XMC queries and mutations are prefixed with `xmc.`.

### Sites API

| Key | Type | Description |
|-----|------|-------------|
| `xmc.sites.list` | Query | List all sites |
| `xmc.sites.get` | Query | Get site by ID |
| `xmc.sites.current` | Query | Get current site context |

```typescript
const sites = await client.query("xmc.sites.list");
const site = await client.query("xmc.sites.get", { id: "site-id" });
const current = await client.query("xmc.sites.current");
```

### Pages API

| Key | Type | Description |
|-----|------|-------------|
| `xmc.pages.current` | Query | Get current page context |
| `xmc.pages.list` | Query | List pages for a site |
| `xmc.pages.get` | Query | Get page by ID |

```typescript
const currentPage = await client.query("xmc.pages.current");
const pages = await client.query("xmc.pages.list", { siteId: "site-id" });
```

### Authoring API (GraphQL)

| Key | Type | Description |
|-----|------|-------------|
| `xmc.authoring.query` | Query | Execute a GraphQL query against XM Cloud Authoring API |
| `xmc.authoring.mutate` | Mutation | Execute a GraphQL mutation |

```typescript
// GraphQL query through SDK
const result = await client.query("xmc.authoring.query", {
  query: \`
    query GetItem($path: String!) {
      item(path: $path) {
        id
        name
        fields {
          name
          value
        }
      }
    }
  \`,
  variables: { path: "/sitecore/content/Home" },
});

// GraphQL mutation
await client.mutate("xmc.authoring.mutate", {
  query: \`
    mutation UpdateField($itemId: ID!, $fieldName: String!, $value: String!) {
      updateItem(input: { itemId: $itemId, fields: [{ name: $fieldName, value: $value }] }) {
        item { id }
      }
    }
  \`,
  variables: { itemId: "item-id", fieldName: "Title", value: "New Title" },
});
```

### Content Transfer API

| Key | Type | Description |
|-----|------|-------------|
| `xmc.contentTransfer.export` | Mutation | Export content |
| `xmc.contentTransfer.import` | Mutation | Import content |
| `xmc.contentTransfer.status` | Query | Check transfer status |

### Search API

| Key | Type | Description |
|-----|------|-------------|
| `xmc.search.query` | Query | Search content items |

```typescript
const results = await client.query("xmc.search.query", {
  query: "hero banner",
  siteId: "site-id",
});
```

### Agent API

| Key | Type | Description |
|-----|------|-------------|
| `xmc.agent.invoke` | Mutation | Invoke an XM Cloud agent |
| `xmc.agent.status` | Query | Check agent execution status |

## Server-Side Client

For full-stack (Auth0) apps, use the server-side client in API routes or server actions:

```typescript
import { experimental_createXMCClient } from "@anthropic-ai/sitecore-marketplace-sdk-xmc/server";

// In a Next.js API route or Server Action
const xmcClient = await experimental_createXMCClient({
  accessToken: session.accessToken, // From Auth0
});

const sites = await xmcClient.sites.list();
const page = await xmcClient.pages.get({ id: "page-id" });

// Direct GraphQL access
const result = await xmcClient.authoring.query({
  query: "...",
  variables: {},
});
```

## Subscriptions

| Key | Description | Data |
|-----|-------------|------|
| `xmc.pages.current.change` | Current page changed | Page context |
| `xmc.sites.current.change` | Current site changed | Site context |

```typescript
client.subscribe("xmc.pages.current.change", (page) => {
  console.log("Page changed:", page);
});
```
