# Client Package API Reference

## createClient(options)

Creates and initializes the SDK client.

```typescript
import { createClient } from "@anthropic-ai/sitecore-marketplace-sdk-client";

const client = createClient({
  appId: string;          // Required: App ID from Developer Portal
  modules?: Module[];     // Optional: Additional modules (XMC, AI)
});
```

## QueryMap — Available Queries

Queries are read-only operations that fetch data from the host application.

| Query Key | Description | Returns |
|-----------|-------------|---------|
| `app.context` | Get current app context (user, org, env) | `AppContext` |
| `app.theme` | Get current theme (light/dark) | `Theme` |
| `app.locale` | Get current locale | `Locale` |
| `app.viewport` | Get current viewport dimensions | `Viewport` |

### Usage
```typescript
const context = await client.query("app.context");
// { user: { id, email, name }, organization: { id, name }, environment: { id, name } }

const theme = await client.query("app.theme");
// { mode: "light" | "dark" }

const locale = await client.query("app.locale");
// { language: "en", country: "US" }
```

## MutationMap — Available Mutations

Mutations perform actions in the host application.

| Mutation Key | Description | Params |
|-------------|-------------|--------|
| `app.toast` | Show a toast notification | `{ message: string, type?: "success" \| "error" \| "info" }` |
| `app.navigate` | Navigate the host app | `{ url: string }` |
| `app.resize` | Resize the app iframe | `{ height: number }` |
| `app.modal.open` | Open a modal dialog | `{ url: string, title?: string, size?: "sm" \| "md" \| "lg" }` |
| `app.modal.close` | Close current modal | `{}` |

### Usage
```typescript
// Show a toast
await client.mutate("app.toast", { message: "Saved!", type: "success" });

// Resize iframe
await client.mutate("app.resize", { height: 600 });

// Open modal
await client.mutate("app.modal.open", {
  url: "/modal-content",
  title: "Edit Item",
  size: "lg",
});
```

## SubscribeMap — Available Subscriptions

Subscriptions listen for real-time events from the host application.

| Subscription Key | Description | Data |
|-----------------|-------------|------|
| `app.theme.change` | Theme changed | `Theme` |
| `app.locale.change` | Locale changed | `Locale` |
| `app.viewport.change` | Viewport resized | `Viewport` |
| `app.context.change` | App context changed | `AppContext` |

### Usage
```typescript
// Subscribe to theme changes
const unsubscribe = client.subscribe("app.theme.change", (theme) => {
  console.log("Theme changed:", theme.mode);
});

// Cleanup
unsubscribe();
```

## React Hooks

The scaffold generates React hooks for convenient SDK access:

```typescript
import { useSitecoreClient } from "@/hooks/use-sitecore";

function MyComponent() {
  const client = useSitecoreClient();

  // Use client.query(), client.mutate(), client.subscribe()
}
```

## TypeScript Types

```typescript
interface AppContext {
  user: {
    id: string;
    email: string;
    name: string;
  };
  organization: {
    id: string;
    name: string;
  };
  environment: {
    id: string;
    name: string;
  };
}

interface Theme {
  mode: "light" | "dark";
}

interface Locale {
  language: string;
  country: string;
}

interface Viewport {
  width: number;
  height: number;
}
```
