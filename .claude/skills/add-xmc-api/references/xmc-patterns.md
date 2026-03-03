# XM Cloud API Code Patterns

## Module Registration

Ensure the XMC module is registered in your client initialization (`lib/sitecore/client.ts`):

```typescript
import { createClient } from "@anthropic-ai/sitecore-marketplace-sdk-client";
import { xmcModule } from "@anthropic-ai/sitecore-marketplace-sdk-xmc";

export const client = createClient({
  appId: process.env.NEXT_PUBLIC_SITECORE_APP_ID!,
  modules: [xmcModule()],
});
```

## Sites API Patterns

### List All Sites (Client-Side)
```tsx
"use client";
import { useEffect, useState } from "react";
import { useSitecoreClient } from "@/hooks/use-sitecore";

export function SitesList() {
  const client = useSitecoreClient();
  const [sites, setSites] = useState<any[]>([]);

  useEffect(() => {
    client.query("xmc.sites.list").then(setSites);
  }, [client]);

  return (
    <ul>
      {sites.map((site) => (
        <li key={site.id}>{site.name}</li>
      ))}
    </ul>
  );
}
```

### Get Current Site Context
```typescript
const currentSite = await client.query("xmc.sites.current");
// Subscribe to changes
client.subscribe("xmc.sites.current.change", (site) => {
  console.log("Site changed:", site);
});
```

## Pages API Patterns

### Get Current Page with Subscription
```tsx
"use client";
import { useEffect, useState } from "react";
import { useSitecoreClient } from "@/hooks/use-sitecore";

export function PageInfo() {
  const client = useSitecoreClient();
  const [page, setPage] = useState<any>(null);

  useEffect(() => {
    // Get initial page
    client.query("xmc.pages.current").then(setPage);

    // Subscribe to page changes (important for context panels)
    const unsub = client.subscribe("xmc.pages.current.change", setPage);
    return () => unsub();
  }, [client]);

  if (!page) return null;
  return <div>Current page: {page.name}</div>;
}
```

## Authoring API Patterns (GraphQL)

### Query Items
```typescript
const result = await client.query("xmc.authoring.query", {
  query: `
    query GetItem($path: String!) {
      item(path: $path) {
        id
        name
        fields {
          name
          value
        }
        children {
          nodes {
            id
            name
          }
        }
      }
    }
  `,
  variables: { path: "/sitecore/content/Home" },
});
```

### Mutate Items
```typescript
const result = await client.mutate("xmc.authoring.mutate", {
  query: `
    mutation CreateItem($parentId: ID!, $name: String!, $templateId: ID!) {
      createItem(input: {
        parentId: $parentId
        name: $name
        templateId: $templateId
      }) {
        item {
          id
          name
        }
      }
    }
  `,
  variables: {
    parentId: "parent-item-id",
    name: "New Item",
    templateId: "template-id",
  },
});
```

## Search API Patterns

### Search Content
```typescript
const results = await client.query("xmc.search.query", {
  query: "hero banner",
  siteId: "site-id",
  // Optional filters
  // templateId: "template-id",
  // language: "en",
});
```

## Content Transfer API Patterns

### Export Content
```typescript
const exportResult = await client.mutate("xmc.contentTransfer.export", {
  itemIds: ["item-id-1", "item-id-2"],
  includeChildren: true,
});

// Check status
const status = await client.query("xmc.contentTransfer.status", {
  transferId: exportResult.transferId,
});
```

## Agent API Patterns

### Invoke Agent
```typescript
const result = await client.mutate("xmc.agent.invoke", {
  agentId: "agent-id",
  input: { /* agent-specific input */ },
});

// Check status
const status = await client.query("xmc.agent.status", {
  executionId: result.executionId,
});
```

## Server-Side Patterns (Auth0 Required)

### API Route
```typescript
// app/api/sites/route.ts
import { experimental_createXMCClient } from "@anthropic-ai/sitecore-marketplace-sdk-xmc/server";
import { getAccessToken } from "@auth0/nextjs-auth0";

export async function GET() {
  const { accessToken } = await getAccessToken();
  const xmcClient = await experimental_createXMCClient({
    accessToken: accessToken!,
  });

  const sites = await xmcClient.sites.list();
  return Response.json(sites);
}
```

### Server Action
```typescript
// app/actions.ts
"use server";
import { experimental_createXMCClient } from "@anthropic-ai/sitecore-marketplace-sdk-xmc/server";
import { getAccessToken } from "@auth0/nextjs-auth0";

export async function getSites() {
  const { accessToken } = await getAccessToken();
  const xmcClient = await experimental_createXMCClient({
    accessToken: accessToken!,
  });
  return xmcClient.sites.list();
}

export async function queryAuthoring(query: string, variables: Record<string, any>) {
  const { accessToken } = await getAccessToken();
  const xmcClient = await experimental_createXMCClient({
    accessToken: accessToken!,
  });
  return xmcClient.authoring.query({ query, variables });
}
```
