# Google Stitch SDK Reference

`@google/stitch-sdk` — generate UI screens from text prompts and extract HTML + screenshots.

## Install

```bash
npm install @google/stitch-sdk
```

Set `STITCH_API_KEY` in your environment or `.dev.vars`.

## Quick Start

```typescript
import { stitch } from "@google/stitch-sdk";

// Create a project
const result = await stitch.callTool("create_project", { title: "My Site" });

// Reference an existing project
const project = stitch.project("4044680601076201931");

// Generate a screen
const screen = await project.generate("A modern landing page with hero section", "DESKTOP");

// Get assets
const htmlUrl = await screen.getHtml();    // Download URL for HTML
const imageUrl = await screen.getImage();  // Download URL for screenshot

// Edit an existing screen
const edited = await screen.edit("Make the background dark and enlarge the CTA button");

// Generate variants
const variants = await screen.variants("Try different colour schemes", {
  variantCount: 3,
  creativeRange: "EXPLORE",     // "REFINE" | "EXPLORE" | "REIMAGINE"
  aspects: ["COLOR_SCHEME"],    // "LAYOUT" | "COLOR_SCHEME" | "IMAGES" | "TEXT_FONT" | "TEXT_CONTENT"
});
```

## Device Types

`"MOBILE"` | `"DESKTOP"` | `"TABLET"` | `"AGNOSTIC"`

## Model Selection

```typescript
// Default model
const screen = await project.generate(prompt, "DESKTOP");

// Specific model
const screen = await project.generate(prompt, "DESKTOP", "GEMINI_3_PRO");
// Options: "GEMINI_3_PRO" | "GEMINI_3_FLASH"
```

## Project Management

```typescript
// List all projects
const projects = await stitch.projects();

// Get screens in a project
const screens = await project.screens();

// Get a specific screen
const screen = await project.getScreen("screenId");
```

## Tool Client (Low-Level)

For direct MCP tool access:

```typescript
const tools = await stitch.listTools();
const result = await stitch.callTool("generate_screen_from_text", {
  projectId: "123",
  prompt: "A dashboard with charts",
  deviceType: "DESKTOP",
});
```

## Downloading Assets

The `getHtml()` and `getImage()` methods return download URLs. Fetch with curl or node:

```bash
# Download HTML
curl -L -o .design/designs/index.html "$(htmlUrl)"

# Download screenshot (append =w{width} for full resolution)
curl -L -o .design/screenshots/index.png "${imageUrl}=w1280"
```

## Authentication

| Variable | Required | Description |
|----------|----------|-------------|
| `STITCH_API_KEY` | Yes (or OAuth) | API key |
| `STITCH_ACCESS_TOKEN` | No | OAuth access token (alternative) |
| `GOOGLE_CLOUD_PROJECT` | With OAuth | GCP project ID |

## Error Handling

```typescript
import { StitchError } from "@google/stitch-sdk";

try {
  const screen = await project.generate(prompt);
} catch (error) {
  if (error instanceof StitchError) {
    console.error(error.code);    // AUTH_FAILED, NOT_FOUND, RATE_LIMITED, etc.
    console.error(error.message);
  }
}
```

## Vercel AI SDK Integration

```typescript
import { generateText, stepCountIs } from "ai";
import { google } from "@ai-sdk/google";
import { stitchTools } from "@google/stitch-sdk/ai";

const { text, steps } = await generateText({
  model: google("gemini-2.5-flash"),
  tools: stitchTools(),
  prompt: "Create a project and generate a modern dashboard",
  stopWhen: stepCountIs(5),
});
```

## Tips for Design Loop Integration

1. **Persist project ID** in `.design/metadata.json` — don't create a new project each iteration
2. **Use `screen.edit()`** for refinements rather than full regeneration
3. **Post-process Stitch HTML** — replace headers/footers with your shared elements
4. **Screenshot URLs need width suffix** — append `=w1280` for full resolution (Google CDN serves thumbnails by default)
5. **Include DESIGN.md context in prompts** — Stitch generates better results with explicit design system instructions
