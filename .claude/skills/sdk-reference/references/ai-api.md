# AI Skills Package API Reference

## Module Registration

```typescript
import { createClient } from "@anthropic-ai/sitecore-marketplace-sdk-client";
import { aiModule } from "@anthropic-ai/sitecore-marketplace-sdk-ai";

const client = createClient({
  appId: process.env.NEXT_PUBLIC_SITECORE_APP_ID!,
  modules: [aiModule()],
});
```

## Brand Review API

The Brand Review API provides AI-powered content analysis against brand guidelines.

### Client-Side

| Key | Type | Description |
|-----|------|-------------|
| `ai.skills.generateBrandReview` | Mutation | Generate a brand review for content |

```typescript
// Text review
const review = await client.mutate("ai.skills.generateBrandReview", {
  content: {
    type: "text",
    value: "Your marketing copy here...",
  },
  brandGuidelines: "Keep tone professional and concise. Avoid jargon.",
});

// Image review (base64)
const review = await client.mutate("ai.skills.generateBrandReview", {
  content: {
    type: "image",
    value: base64EncodedImage,
    mimeType: "image/png",
  },
  brandGuidelines: "Brand colors: blue (#0046BE), white. No red.",
});

// Image review (URL)
const review = await client.mutate("ai.skills.generateBrandReview", {
  content: {
    type: "image",
    url: "https://example.com/banner.png",
  },
  brandGuidelines: "Logo must be visible. Min contrast ratio 4.5:1.",
});

// Document review (PDF or text file)
const review = await client.mutate("ai.skills.generateBrandReview", {
  content: {
    type: "document",
    value: base64EncodedPDF,
    mimeType: "application/pdf",
  },
  brandGuidelines: "Follow AP style. Max reading level: grade 8.",
});
```

### Brand Review Response Type

```typescript
interface BrandReviewResult {
  score: number;               // 0-100 brand compliance score
  summary: string;             // Overall assessment
  issues: BrandReviewIssue[];  // List of specific issues found
  suggestions: string[];       // Improvement suggestions
}

interface BrandReviewIssue {
  severity: "error" | "warning" | "info";
  category: string;            // e.g., "tone", "visual", "terminology"
  description: string;         // What the issue is
  location?: string;           // Where in the content (if applicable)
  suggestion?: string;         // How to fix it
}
```

### Content Input Types

```typescript
type BrandReviewContent =
  | { type: "text"; value: string }
  | { type: "image"; value: string; mimeType: string }     // base64
  | { type: "image"; url: string }                          // URL
  | { type: "document"; value: string; mimeType: string };  // base64 PDF/text

// Supported mimeTypes for images: "image/png", "image/jpeg", "image/webp", "image/gif"
// Supported mimeTypes for documents: "application/pdf", "text/plain"
```

## Server-Side Client

For full-stack (Auth0) apps:

```typescript
import { experimental_createAIClient } from "@anthropic-ai/sitecore-marketplace-sdk-ai/server";

// In a Next.js API route or Server Action
const aiClient = await experimental_createAIClient({
  accessToken: session.accessToken, // From Auth0
});

const review = await aiClient.skills.generateBrandReview({
  content: { type: "text", value: "Content to review..." },
  brandGuidelines: "Brand guidelines here...",
});
```
