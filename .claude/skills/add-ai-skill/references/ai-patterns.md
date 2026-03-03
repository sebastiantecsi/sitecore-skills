# AI Skills Code Patterns

## Module Registration

Ensure the AI module is registered in your client initialization (`lib/sitecore/client.ts`):

```typescript
import { createClient } from "@anthropic-ai/sitecore-marketplace-sdk-client";
import { aiModule } from "@anthropic-ai/sitecore-marketplace-sdk-ai";

export const client = createClient({
  appId: process.env.NEXT_PUBLIC_SITECORE_APP_ID!,
  modules: [aiModule()],
});
```

## Client-Side: Text Review

```tsx
"use client";
import { useState } from "react";
import { useSitecoreClient } from "@/hooks/use-sitecore";
import { Button } from "@/components/ui/button";
import { Textarea } from "@/components/ui/textarea";
import { Card, CardContent, CardHeader, CardTitle } from "@/components/ui/card";
import { Badge } from "@/components/ui/badge";
import { Alert, AlertDescription } from "@/components/ui/alert";

export function BrandReviewText() {
  const client = useSitecoreClient();
  const [content, setContent] = useState("");
  const [guidelines, setGuidelines] = useState("");
  const [review, setReview] = useState<any>(null);
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState<string | null>(null);

  const handleReview = async () => {
    setLoading(true);
    setError(null);
    try {
      const result = await client.mutate("ai.skills.generateBrandReview", {
        content: { type: "text", value: content },
        brandGuidelines: guidelines,
      });
      setReview(result);
    } catch (err) {
      setError(err instanceof Error ? err.message : "Review failed");
    } finally {
      setLoading(false);
    }
  };

  return (
    <div className="space-y-4">
      <Textarea
        placeholder="Paste content to review..."
        value={content}
        onChange={(e) => setContent(e.target.value)}
      />
      <Textarea
        placeholder="Brand guidelines..."
        value={guidelines}
        onChange={(e) => setGuidelines(e.target.value)}
      />
      <Button onClick={handleReview} disabled={loading || !content}>
        {loading ? "Reviewing..." : "Review Content"}
      </Button>

      {error && (
        <Alert variant="destructive">
          <AlertDescription>{error}</AlertDescription>
        </Alert>
      )}

      {review && (
        <Card>
          <CardHeader>
            <CardTitle className="flex items-center gap-2">
              Brand Score
              <Badge variant={review.score >= 80 ? "default" : "destructive"}>
                {review.score}/100
              </Badge>
            </CardTitle>
          </CardHeader>
          <CardContent className="space-y-2">
            <p>{review.summary}</p>
            {review.issues?.map((issue: any, i: number) => (
              <Alert key={i} variant={issue.severity === "error" ? "destructive" : "default"}>
                <AlertDescription>
                  <strong>{issue.category}:</strong> {issue.description}
                  {issue.suggestion && <p className="mt-1 text-sm">{issue.suggestion}</p>}
                </AlertDescription>
              </Alert>
            ))}
          </CardContent>
        </Card>
      )}
    </div>
  );
}
```

## Client-Side: Image Review

```typescript
// From a file input
const file = inputElement.files[0];
const reader = new FileReader();
reader.onload = async () => {
  const base64 = (reader.result as string).split(",")[1];
  const review = await client.mutate("ai.skills.generateBrandReview", {
    content: {
      type: "image",
      value: base64,
      mimeType: file.type, // "image/png", "image/jpeg", etc.
    },
    brandGuidelines: "Brand colors: #0046BE (blue), #FFFFFF (white). Logo must be visible.",
  });
};
reader.readAsDataURL(file);

// From a URL
const review = await client.mutate("ai.skills.generateBrandReview", {
  content: {
    type: "image",
    url: "https://example.com/banner.png",
  },
  brandGuidelines: "Minimum contrast ratio 4.5:1. No competitor logos.",
});
```

## Client-Side: Document Review

```typescript
// PDF review
const file = inputElement.files[0]; // .pdf file
const reader = new FileReader();
reader.onload = async () => {
  const base64 = (reader.result as string).split(",")[1];
  const review = await client.mutate("ai.skills.generateBrandReview", {
    content: {
      type: "document",
      value: base64,
      mimeType: "application/pdf",
    },
    brandGuidelines: "Follow AP style. Max reading level: grade 8. Use active voice.",
  });
};
reader.readAsDataURL(file);
```

## Server-Side: API Route (Auth0 Required)

```typescript
// app/api/brand-review/route.ts
import { experimental_createAIClient } from "@anthropic-ai/sitecore-marketplace-sdk-ai/server";
import { getAccessToken } from "@auth0/nextjs-auth0";

export async function POST(request: Request) {
  const { accessToken } = await getAccessToken();
  const aiClient = await experimental_createAIClient({
    accessToken: accessToken!,
  });

  const { content, brandGuidelines } = await request.json();

  const review = await aiClient.skills.generateBrandReview({
    content,
    brandGuidelines,
  });

  return Response.json(review);
}
```

## Server-Side: Server Action (Auth0 Required)

```typescript
// app/actions.ts
"use server";
import { experimental_createAIClient } from "@anthropic-ai/sitecore-marketplace-sdk-ai/server";
import { getAccessToken } from "@auth0/nextjs-auth0";

export async function reviewContent(text: string, guidelines: string) {
  const { accessToken } = await getAccessToken();
  const aiClient = await experimental_createAIClient({
    accessToken: accessToken!,
  });

  return aiClient.skills.generateBrandReview({
    content: { type: "text", value: text },
    brandGuidelines: guidelines,
  });
}
```

## Brand Review Response Handling

```typescript
interface BrandReviewResult {
  score: number;               // 0-100
  summary: string;
  issues: {
    severity: "error" | "warning" | "info";
    category: string;          // "tone", "visual", "terminology", etc.
    description: string;
    location?: string;
    suggestion?: string;
  }[];
  suggestions: string[];
}

// Helper to categorize score
function getScoreStatus(score: number) {
  if (score >= 80) return { label: "Excellent", variant: "default" as const };
  if (score >= 60) return { label: "Needs Work", variant: "secondary" as const };
  return { label: "Poor", variant: "destructive" as const };
}
```
