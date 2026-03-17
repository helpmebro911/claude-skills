# UX Walkthrough Checklist

Evaluate each screen against these categories during a walkthrough. The goal is to **dogfood** the app — use it as a real person with a real job to do, not as a tester running through checkboxes.

## User Persona (Set Before Starting)

Before evaluating any screen, know who you are:

| Question | Why It Matters |
|----------|---------------|
| Who is the user? | A developer and a receptionist notice different things |
| How much time do they have? | Time-poor users won't explore — they need obvious paths |
| How tech-comfortable are they? | Determines tolerance for complexity |
| What device are they on? | Desktop at a desk vs phone between tasks |
| How often do they use this? | Daily power user vs monthly visitor |
| What's their emotional state? | Calm and focused vs stressed and rushing |

Default persona if not specified: "First-time user, moderate tech comfort, slightly distracted, wants to get this done quickly and move on."

## Per-Screen Evaluation

| Category | What to Check |
|----------|---------------|
| **First Impression** | Does the page orient me? Do I know what I can do here? |
| **Navigation** | Can I find what I need in 3 clicks or fewer? Is the current location clear? |
| **Labels & Icons** | Do they describe what they do? Would a first-time user understand them? |
| **Visual Hierarchy** | Is the most important information prominent? Is secondary info de-emphasised? |
| **Call to Action** | Is the primary action obvious? Are secondary actions visually distinct? |
| **Forms** | Are required fields marked? Is validation immediate and clear? Are error messages helpful? |
| **Feedback** | Does the app confirm my actions? Are there loading states? Success/error toasts? |
| **Error Recovery** | Can I undo mistakes? Is there a back button? Are destructive actions guarded? |
| **Consistency** | Same patterns used for similar features? Same terminology throughout? |
| **Data Display** | Tables sorted sensibly? Pagination? Empty states helpful? Long text truncated? |
| **Layout & Visual** | All text visible? Nothing clipped by sidebar or container? Spacing consistent? No overlapping elements? |

## Emotional Friction (Dogfooding Focus)

These go beyond "does it work?" to "how does it feel?"

| Category | What to Check |
|----------|---------------|
| **Trust** | Do I feel confident about what will happen when I click this? Or am I nervous? |
| **Certainty** | After an action, am I sure it worked? Or do I need to go check somewhere else? |
| **Momentum** | Does the app keep me moving forward? Or do I hit walls and have to backtrack? |
| **Cognitive load** | Am I thinking about my task, or thinking about the interface? |
| **Anxiety points** | Are there moments where I'm afraid I'll lose work or break something? |
| **Satisfaction** | After completing the task, do I feel accomplished or exhausted? |

### Trust Signals to Look For

- Confirmation before destructive actions (delete, overwrite, send)
- Clear indication of saved vs unsaved state
- Undo available for reversible actions
- Preview before commit (e.g. "This will email 50 people")
- No surprising side effects from simple actions

### Anxiety Red Flags

- No autosave on long forms
- Ambiguous button labels ("Process", "Submit", "Continue" — continue to WHERE?)
- Destructive actions styled the same as safe actions
- No way to see what just happened (no activity log, no confirmation)
- Modal dialogs with no escape route

## Flow Efficiency

Track the mechanical cost of completing the task:

| Metric | What to Measure |
|--------|----------------|
| **Click count** | Total clicks from start to task completion |
| **Decision points** | Moments where the user has to stop and think about what to do next |
| **Dead ends** | Times the user goes down the wrong path and has to backtrack |
| **Redundant steps** | Actions that feel unnecessary (re-entering info, confirming the obvious) |
| **Shortcuts available?** | For repeat users — keyboard shortcuts, bulk actions, defaults, recent items |

A good flow feels like sliding downhill. A bad flow feels like climbing stairs. Count the stairs.

## Resilience Testing

What happens when things go wrong or the user behaves unexpectedly?

| Scenario | What Should Happen |
|----------|-------------------|
| Navigate away mid-form | Data preserved, or clear warning before losing it |
| Submit with bad/missing data | Specific error messages on the right fields, form state preserved |
| Hit the back button | Sensible navigation, not a broken state |
| Refresh the page | State survives (especially filters, scroll position, form data) |
| Double-click a submit button | No duplicate submission |
| Slow/no network | Graceful degradation, not a white screen |
| Very long text input | Handled gracefully (truncated, scrolled, not overflowing) |

## Visual & Layout Inspection

Catch layout bugs that break the visual presentation. These are easy to miss because the app still "works" — but it looks broken.

**On every page, actively look for:**

| Issue | What to look for | Common cause |
|-------|-----------------|--------------|
| **Clipped/truncated text** | Headings, labels, or descriptions cut off by a container edge. First characters missing. | Content area missing left margin/padding, or sidebar overlapping main content |
| **Overlapping elements** | Sidebar covering main content. Modals under nav bars. Floating buttons over text. | Z-index conflicts, missing margin on main content, fixed positioning errors |
| **Overflow / scrollbars** | Unexpected horizontal scroll. Content wider than viewport. Rogue scrollbar on a container. | Element with fixed width exceeding parent, uncontained images or tables |
| **Broken grid/alignment** | Cards or columns not aligned. Uneven spacing. Items jumping when data loads. | CSS grid/flex issues, missing min-width constraints, layout shift from async data |
| **Text contrast** | Text hard to read against background. Light grey on white. Dark text on dark background. | Missing dark mode styles, low-contrast colour choices, text over images without overlay |
| **Misaligned sidebar/content** | Main content starts behind or underneath the sidebar. Content pushed too far right with gap. | Sidebar width not accounted for in main content margin/padding |
| **Broken responsive transitions** | Layout looks fine at desktop and mobile but breaks at tablet widths. Nav items wrapping oddly. | Missing breakpoint styles for mid-range widths |
| **Image issues** | Broken image icons. Images stretched or squished. Oversized images causing slow load. | Missing src, no aspect-ratio/object-fit, unoptimised originals |
| **Invisible elements** | Buttons or links that exist but can't be seen (same colour as background). | Dark mode missing styles, transparent text, hidden by z-index |
| **Spacing inconsistency** | Some sections have generous padding, others are cramped. Cards with different internal spacing. | Inconsistent use of spacing utilities, missing design tokens |

**How to check**: On each page, before evaluating usability, do a quick visual scan:
1. Does all text render fully? No clipped first/last characters?
2. Does the sidebar and main content sit side by side cleanly?
3. Are all elements inside their containers?
4. Is spacing consistent between similar elements?
5. In dark mode: can you read everything? Any elements disappear?

**Severity guide for visual issues**:
- **Critical**: Content is unreadable or unreachable (text fully hidden behind sidebar)
- **High**: Content is partially clipped or overlapping (first letters cut off, as in the screenshot issue)
- **Medium**: Visual inconsistency that doesn't block usage (uneven spacing, slight misalignment)
- **Low**: Minor polish (1-2px alignment, subtle colour inconsistency)

## Cross-Cutting Checks

| Category | What to Check |
|----------|---------------|
| **Mobile (375px)** | Touch targets at least 44px. No horizontal scroll. Text readable. Forms usable. |
| **Dark Mode** | All text readable. No invisible elements. Borders/separators visible. Images appropriate. |
| **Keyboard** | Tab order logical. Focus indicators visible. Modals trap focus. Escape closes dialogs. |
| **Loading States** | Skeleton screens or spinners. No layout shift when data loads. Buttons disabled during submit. |
| **Empty States** | Helpful message when no data. Clear call to action to add first item. |

## Friction Scoring

When you find an issue, classify it:

| Severity | Definition | Example |
|----------|-----------|---------|
| **Critical** | User cannot complete their task | Submit button does nothing, form data lost |
| **High** | User is confused or takes wrong path | Unclear labels cause wrong selection, no undo on delete |
| **Medium** | User succeeds but with unnecessary effort | Required field not marked, have to scroll to find action |
| **Low** | Minor polish issue | Inconsistent capitalisation, alignment off by a few pixels |

## The Big Questions

Ask these after completing (or failing) the task. These are the dogfooding questions — they go beyond "does it work?" to "is it good?"

1. **Would I come back?** Or would I look for an alternative tool next time?
2. **Could I teach someone else?** In under 2 minutes, with no documentation?
3. **What's the one thing?** If I could change one thing to make this twice as easy, what would it be?
4. **Where did I hesitate?** Every hesitation is a design failure — the interface made me think instead of act.
5. **Did I trust it?** At any point, was I unsure whether my data was saved, my action was processed, or my input was correct?
6. **What would a busy person skip?** Which steps feel optional, tedious, or "I'll do that later"?
7. **Would I recommend this?** Not just "it works" but "you should use this, it's good."
