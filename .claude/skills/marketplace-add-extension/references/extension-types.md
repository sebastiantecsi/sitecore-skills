# Extension Point Boilerplate Templates

## Custom Field (`custom-field`)

Route: `app/custom-field/page.tsx`

A custom field appears inline in the Sitecore content editor. It receives and returns field values.

```tsx
"use client";

import { useEffect, useState } from "react";
import { useSitecoreClient } from "@/hooks/use-sitecore";
import { Input } from "@/components/ui/input";
import { Label } from "@/components/ui/label";
import { Skeleton } from "@/components/ui/skeleton";

export default function CustomFieldPage() {
  const client = useSitecoreClient();
  const [value, setValue] = useState("");
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    async function init() {
      try {
        // Get the current field value from the host
        const context = await client.query("app.context");
        // Field value would come from the extension point context
        setLoading(false);
      } catch (err) {
        console.error("Failed to initialize:", err);
        setLoading(false);
      }
    }
    init();
  }, [client]);

  const handleChange = async (newValue: string) => {
    setValue(newValue);
    // Notify the host of the value change
    // The exact mutation depends on your field type
  };

  if (loading) return <Skeleton className="h-10 w-full" />;

  return (
    <div className="p-2 space-y-2">
      <Label htmlFor="field">Field Label</Label>
      <Input
        id="field"
        value={value}
        onChange={(e) => handleChange(e.target.value)}
        placeholder="Enter value..."
      />
    </div>
  );
}
```

## Dashboard Widget (`dashboard-widget`)

Route: `app/dashboard-widget/page.tsx`

A widget displayed on the Sitecore dashboard. Good for stats, summaries, and quick actions.

```tsx
"use client";

import { useEffect, useState } from "react";
import { useSitecoreClient } from "@/hooks/use-sitecore";
import { Card, CardContent, CardHeader, CardTitle } from "@/components/ui/card";
import { Button } from "@/components/ui/button";
import { Skeleton } from "@/components/ui/skeleton";
import { Alert, AlertDescription } from "@/components/ui/alert";

export default function DashboardWidgetPage() {
  const client = useSitecoreClient();
  const [data, setData] = useState<any>(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState<string | null>(null);

  useEffect(() => {
    async function fetchData() {
      try {
        const context = await client.query("app.context");
        setData(context);
      } catch (err) {
        setError(err instanceof Error ? err.message : "Failed to load");
      } finally {
        setLoading(false);
      }
    }
    fetchData();
  }, [client]);

  if (loading) {
    return (
      <Card>
        <CardHeader><Skeleton className="h-6 w-32" /></CardHeader>
        <CardContent><Skeleton className="h-20 w-full" /></CardContent>
      </Card>
    );
  }

  if (error) {
    return <Alert variant="destructive"><AlertDescription>{error}</AlertDescription></Alert>;
  }

  return (
    <Card>
      <CardHeader>
        <CardTitle>My Widget</CardTitle>
      </CardHeader>
      <CardContent className="space-y-4">
        <p>Welcome, {data?.user?.name}</p>
        <Button onClick={() => client.mutate("app.toast", { message: "Hello!" })}>
          Say Hello
        </Button>
      </CardContent>
    </Card>
  );
}
```

## Pages Context Panel (`pages-context-panel`)

Route: `app/pages-context-panel/page.tsx`

A side panel in the Pages editor. Receives current page context and can subscribe to page changes.

```tsx
"use client";

import { useEffect, useState } from "react";
import { useSitecoreClient } from "@/hooks/use-sitecore";
import { Card, CardContent, CardHeader, CardTitle } from "@/components/ui/card";
import { Separator } from "@/components/ui/separator";
import { Badge } from "@/components/ui/badge";
import { Skeleton } from "@/components/ui/skeleton";

export default function PagesContextPanelPage() {
  const client = useSitecoreClient();
  const [page, setPage] = useState<any>(null);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    async function init() {
      try {
        const currentPage = await client.query("xmc.pages.current");
        setPage(currentPage);
      } catch (err) {
        console.error("Failed to load page context:", err);
      } finally {
        setLoading(false);
      }
    }
    init();

    // Subscribe to page changes
    const unsubscribe = client.subscribe("xmc.pages.current.change", (newPage) => {
      setPage(newPage);
    });

    return () => unsubscribe();
  }, [client]);

  if (loading) {
    return (
      <div className="p-4 space-y-3">
        <Skeleton className="h-6 w-full" />
        <Skeleton className="h-4 w-3/4" />
        <Skeleton className="h-4 w-1/2" />
      </div>
    );
  }

  return (
    <div className="p-4 space-y-4">
      <div>
        <h2 className="text-lg font-semibold">Page Info</h2>
        <p className="text-sm text-muted-foreground">{page?.name}</p>
      </div>
      <Separator />
      <Card>
        <CardHeader>
          <CardTitle className="text-sm">Status</CardTitle>
        </CardHeader>
        <CardContent>
          <Badge variant="secondary">Draft</Badge>
        </CardContent>
      </Card>
    </div>
  );
}
```

## Fullscreen (`fullscreen`)

Route: `app/fullscreen/page.tsx`

A full-page experience within the Sitecore shell. Use for complex CRUD interfaces, settings, or dashboards.

```tsx
"use client";

import { useEffect, useState } from "react";
import { useSitecoreClient } from "@/hooks/use-sitecore";
import { Button } from "@/components/ui/button";
import { Card, CardContent, CardHeader, CardTitle } from "@/components/ui/card";
import {
  Table,
  TableBody,
  TableCell,
  TableHead,
  TableHeader,
  TableRow,
} from "@/components/ui/table";
import { Skeleton } from "@/components/ui/skeleton";

export default function FullscreenPage() {
  const client = useSitecoreClient();
  const [context, setContext] = useState<any>(null);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    async function init() {
      try {
        const ctx = await client.query("app.context");
        setContext(ctx);
      } catch (err) {
        console.error("Failed to initialize:", err);
      } finally {
        setLoading(false);
      }
    }
    init();
  }, [client]);

  if (loading) {
    return (
      <div className="p-8 space-y-4">
        <Skeleton className="h-8 w-64" />
        <Skeleton className="h-64 w-full" />
      </div>
    );
  }

  return (
    <div className="p-8 space-y-6">
      <div className="flex items-center justify-between">
        <h1 className="text-2xl font-bold">My App</h1>
        <Button>New Item</Button>
      </div>

      <Card>
        <CardHeader>
          <CardTitle>Items</CardTitle>
        </CardHeader>
        <CardContent>
          <Table>
            <TableHeader>
              <TableRow>
                <TableHead>Name</TableHead>
                <TableHead>Status</TableHead>
                <TableHead>Actions</TableHead>
              </TableRow>
            </TableHeader>
            <TableBody>
              <TableRow>
                <TableCell>Example Item</TableCell>
                <TableCell>Active</TableCell>
                <TableCell>
                  <Button variant="ghost" size="sm">Edit</Button>
                </TableCell>
              </TableRow>
            </TableBody>
          </Table>
        </CardContent>
      </Card>
    </div>
  );
}
```

## Standalone (`standalone`)

Route: `app/standalone/page.tsx`

An independent page not embedded in Sitecore shell. Good for OAuth callbacks, public pages, or standalone tools.

```tsx
"use client";

import { Button } from "@/components/ui/button";
import { Card, CardContent, CardHeader, CardTitle, CardDescription } from "@/components/ui/card";

export default function StandalonePage() {
  return (
    <div className="min-h-screen flex items-center justify-center bg-background">
      <Card className="w-full max-w-md">
        <CardHeader>
          <CardTitle>Standalone Page</CardTitle>
          <CardDescription>
            This page runs independently of the Sitecore shell.
          </CardDescription>
        </CardHeader>
        <CardContent className="space-y-4">
          <p className="text-sm text-muted-foreground">
            Use this for OAuth callbacks, public-facing pages, or standalone tools.
          </p>
          <Button className="w-full">Get Started</Button>
        </CardContent>
      </Card>
    </div>
  );
}
```
