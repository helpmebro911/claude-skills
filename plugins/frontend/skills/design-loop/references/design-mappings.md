# Design Mappings & Descriptors

Use these mappings to transform vague user requests into precise design instructions.

## UI/UX Keyword Refinement

| Vague Term | Professional Terminology |
|:---|:---|
| "menu at the top" | "sticky navigation bar with logo and menu items" |
| "big photo" | "full-width hero section with focal-point imagery" |
| "list of things" | "responsive card grid with hover states and subtle elevation" |
| "button" | "primary call-to-action button with hover transition" |
| "form" | "form with labelled input fields, validation states, and submit button" |
| "picture area" | "hero section with background image or video" |
| "sidebar" | "collapsible side navigation with icon-label pairings" |
| "popup" | "modal dialog with overlay and smooth entry animation" |
| "footer stuff" | "footer with sitemap links, contact info, and legal notices" |
| "cards" | "content cards with consistent padding, rounded corners, and shadow" |
| "tabs" | "tabbed interface with active indicator and smooth content transition" |
| "search" | "search input with icon, placeholder text, and results dropdown" |
| "pricing" | "pricing comparison cards with highlighted recommended tier" |
| "testimonials" | "testimonial carousel or grid with avatar, quote, and attribution" |

## Atmosphere & Vibe Descriptors

| Basic Vibe | Enhanced Description |
|:---|:---|
| "Modern" | "Clean, minimal, generous whitespace, high-contrast typography" |
| "Professional" | "Sophisticated, trustworthy, subtle shadows, restricted premium palette" |
| "Fun / Playful" | "Vibrant, rounded corners, bold accent colours, bouncy animations" |
| "Dark Mode" | "High-contrast accents on deep slate or near-black backgrounds" |
| "Luxury" | "Elegant, spacious, fine lines, serif headers, high-fidelity photography" |
| "Tech / Cyber" | "Futuristic, neon accents, glassmorphism, monospaced typography" |
| "Warm / Friendly" | "Soft colours, rounded shapes, handwritten accents, inviting imagery" |
| "Bold / Industrial" | "Strong typography, high contrast, geometric shapes, dark backgrounds" |
| "Organic / Natural" | "Earth tones, soft textures, organic shapes, nature photography" |
| "Editorial" | "Magazine-like layouts, strong typographic hierarchy, generous leading" |

## Geometry & Shape Language

| Description | Tailwind Class | Visual Effect |
|:---|:---|:---|
| Pill-shaped | `rounded-full` | Buttons, tags, badges |
| Softly rounded | `rounded-xl` (12px) | Cards, containers, modals |
| Gently rounded | `rounded-lg` (8px) | Inputs, smaller elements |
| Sharp / precise | `rounded-none` or `rounded-sm` | Technical, brutalist aesthetic |
| Glassmorphism | `backdrop-blur-md bg-white/10 border border-white/20` | Overlays, nav bars |
| Frosted | `backdrop-blur-sm bg-white/80` | Subtle glass effect |

## Depth & Elevation

| Level | Description | Tailwind |
|:---|:---|:---|
| Flat | No shadows, focus on colour blocking and borders | `shadow-none` |
| Whisper-soft | Diffused, barely visible lift | `shadow-sm` |
| Subtle | Gentle shadow for card elevation | `shadow-md` |
| Floating | High-offset, soft shadow — element appears lifted | `shadow-lg` or `shadow-xl` |
| Dramatic | Strong shadow for hero elements or modals | `shadow-2xl` |
| Inset | Inner shadow for pressed or nested elements | `shadow-inner` |

## Section Spacing Scale

| Density | Description | Tailwind |
|:---|:---|:---|
| Tight | Compact, information-dense | `py-8 md:py-12` |
| Balanced | Standard section spacing | `py-12 md:py-16` |
| Generous | Breathing room, premium feel | `py-16 md:py-24` |
| Dramatic | Statement spacing, luxury/editorial | `py-24 md:py-32` |
