# Prompt 05 — Extract UI

**Input:** `--repo <path>` (routes, pages, components, tailwind config, theme vars, screenshots in chat attachments)
**Output:** `<out>/.scratch/ui-inventory.json`

## Purpose

Document the visual + interaction surface enough that a reimplementer can rebuild it on a different UI library — React Native, SwiftUI, Vue+Vuetify, whatever — without Lovable's stack.

## Output shape

```json
{
  "design_tokens": {
    "colors": {
      "primary": { "value": "hsl(221 83% 53%)", "source": "src/index.css:12" },
      "background": {...},
      "foreground": {...}
    },
    "radii": { "sm": "0.375rem", "md": "0.5rem", "lg": "0.75rem" },
    "spacing_scale": "tailwind-default|custom",
    "typography": {
      "font_families": { "sans": "Inter", "mono": "Fira Code" },
      "sizes": {...}
    },
    "dark_mode": "class|media|none",
    "theme_vars_file": "src/index.css"
  },
  "pages": [
    {
      "route": "/session/:id",
      "file": "src/pages/SessionDetail.tsx",
      "layout": "header + sidebar + main-with-tabs",
      "tabs": ["Overview", "Documents", "Briefing", "Report", "Activity"],
      "primary_actions": ["upload doc", "generate report"],
      "components_used": ["DocumentUpload", "BriefingView", "ReportTab"],
      "states": ["loading", "empty-no-docs", "ready", "report-generating"],
      "auth": "required"
    }
  ],
  "components": [
    {
      "name": "DocumentUpload",
      "file": "src/components/DocumentUpload.tsx",
      "role": "feature",
      "purpose": "Upload and categorise session documents",
      "props_shape": "{ sessionId: string; onUploaded: (doc) => void }",
      "interaction_summary": "drag-drop or click → select category → POST to supabase storage → invalidate query"
    }
  ],
  "shared_ui_kit": {
    "library": "shadcn/ui",
    "components_used": ["Button", "Dialog", "Input", "Select", "Tabs", "Toast"]
  },
  "responsiveness": {
    "breakpoints": ["sm", "md", "lg"],
    "mobile_optimised": true,
    "evidence": ["src/hooks/use-mobile.tsx:..."]
  },
  "accessibility_notes": [
    "radix primitives → keyboard + screen reader support present",
    "colour-only indicators not detected — [TBD audit]"
  ],
  "i18n": { "present": false, "library": null },
  "screenshots_in_chat": [
    { "message_id": "umsg_01xxx", "local_path": "attachments/...", "intent": "mockup of dashboard" }
  ]
}
```

## How

- Grep the router (`<Routes>`, `createBrowserRouter`, `defineRouter`, etc.) to enumerate pages.
- For each page, read its top-level JSX to identify tabs / primary actions; don't read child components recursively.
- Design tokens: parse `tailwind.config.*` + any root CSS file for `--var` definitions.
- Cross-reference `attachments/` (images) with messages — many users paste Figma/screenshots to describe desired UI. Capture the pairing.
- For each feature component, infer purpose from name + top-level JSDoc/comment + chat-insights intents.
- Don't enumerate `components/ui/` individually (shadcn primitives) — just list which ones are imported anywhere.
