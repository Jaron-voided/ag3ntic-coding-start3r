# Design Brief — EdgeDesk

_Source of truth for all UI/UX decisions. Read this before writing any frontend code or making visual choices._

---

## 1. Visual identity

**Mood:** Futuristic, cyberpunk, dark, sharp, immersive. Feels like a real OS command center — not a web form. Neon energy with Linear's discipline: glowing accents that don't hurt to read.

**Reference apps:**
- **Cyberpunk 2077** — dark interface, neon blue/pink accents, glowing UI elements, high-tech command center feeling
- **Linear** — clean layout, dark theme, simple typography, organized panels, professional software feel
- Goal: Cyberpunk 2077's *aesthetic* + Linear's *restraint*

**Anti-references:**
- **Craigslist** — cluttered, no visual hierarchy, looks broken
- **X / Twitter (post-2022)** — chaotic minimalism, stripped of warmth and intentional design
- EdgeDesk should feel *deliberately designed* — every element has a reason

**Brand constraints:**
- Brand: **SouthEast EdgeRunners** (shortened to **EdgeRunners** in day-to-day use)
- Logo: "ER" monogram with integrated lightning bolt — angular, cyberpunk-ready
- Split-color logo treatment to explore: **E in `#00E5FF`**, **R in `#FF2BD6`**
- Existing hex codes (locked — do not deviate):
  - `#1E1E24` — dark grey (primary background)
  - `#0B0D10` — super dark grey (deep background, window chrome, sidebar)
  - `#00E5FF` — cyber neon blue (primary accent)
  - `#FF2BD6` — punk neon pink (secondary accent)

---

## 2. Information architecture

**Primary screens (v1):**
- `/login` — single client login screen
- `/desktop` — the app launcher home (this is the hero screen)
- Each app opens as a **slide-in panel** over the desktop — no separate routes per app

**The 6 desktop app icons:**
1. **CRM** — captured leads and customer list (pulled from Airtable)
2. **EdgeDrive** — photo/file upload that triggers Facebook auto-post via n8n
3. **Resources** — links and assets Jaron has delivered (website, forms, docs)
4. **Calendar** — scheduled jobs or appointments
5. **Todo** — task list
6. **Catch-Up** — global inbox: all notifications from all apps for the day, markable as read/dismissed

**Navigation model:**
- Desktop is the nav. No persistent top nav or side nav.
- Clicking an icon slides open a full-screen panel from the right.
- Close/back button returns to the desktop.
- Icons show a numbered notification badge when there are unread/new items.

**The hero screen:** `/desktop` — the app launcher. In 3 seconds it communicates: "here's what happened while you were on the job site." Notification badges on icons show unread counts at a glance.

---

## 3. Component approach

- **Framework:** React
- **Component library:** [Headless UI](https://headlessui.com/) — accessible unstyled primitives
- **Styling:** Tailwind CSS
- **Icons:** Heroicons

**Headless UI primitives in use:**
- `Dialog` — slide-in panels for each app
- `Menu` — dropdowns (e.g. lead actions, notification options)
- `Switch` — toggles (e.g. mark notification read)

**Custom components (build from scratch):**
- App icon grid with notification badges
- Desktop wallpaper/chrome layer

**Third-party libraries:**
- `react-dropzone` — file upload UX for EdgeDrive
- `@fullcalendar/react` — calendar view

---

## 4. Visual tokens

**Color palette:**

| Token | Hex | Usage |
|---|---|---|
| `bg-base` | `#0B0D10` | Deepest background, window chrome |
| `bg-surface` | `#1E1E24` | Card surfaces, panel backgrounds |
| `accent-blue` | `#00E5FF` | Primary interactive elements, focus rings, highlights |
| `accent-pink` | `#FF2BD6` | Secondary accents, notifications, badges |
| `text-primary` | `#F0F0F0` | Body text |
| `text-muted` | `#6B7280` | Secondary text, labels |
| `success` | `#22C55E` | Tailwind green-500 |
| `warning` | `#F59E0B` | Tailwind amber-500 |
| `danger` | `#EF4444` | Tailwind red-500 |

**Typography:**
- **Display / headings:** [Orbitron](https://fonts.google.com/specimen/Orbitron) — futuristic, techy, cyberpunk energy
- **Body / UI text:** [Inter](https://fonts.google.com/specimen/Inter) — clean, readable at small sizes (Linear's font)
- Scale: `text-xs` (labels), `text-sm` (body), `text-base` (default), `text-lg` (subheadings), `text-2xl`+ (headings/display)

**Spacing scale:** Tailwind defaults.

**Border radius:** `rounded-md` — applied consistently across all components (cards, buttons, panels, inputs). Adjust globally via Tailwind config if the vibe needs to shift sharper.

**Shadow / glow:**
- Neon glow on interactive elements: `box-shadow: 0 0 8px #00E5FF` for blue-accent elements, `0 0 8px #FF2BD6` for pink-accent elements
- Use sparingly — only on focused, hovered, or active states. Glow everywhere = glow nowhere.
- Panel elevation: `shadow-lg` with `bg-surface` background

---

## 5. Accessibility floor

All five committed for v1:

- Keyboard-navigable end-to-end (tab through desktop icons, open/close panels)
- WCAG AA contrast on all text (neon on dark grey exceeds AA by default — verify with a contrast checker)
- Visible labels on all form inputs — no placeholder-as-label
- Visible focus states — neon blue (`#00E5FF`) focus rings on all interactive elements. Do not `outline: none` without a visible replacement.
- Color is never the only way to convey status — notification badges show numbers, not just color; status indicators use icons + color

---

## 6. Responsive strategy

- **v1 target:** Desktop browser only. Built and tested at 1280px+ wide.
- **Mobile:** Out of scope for v1. A future phase will adapt the app-launcher model to a phone UI (full-screen panels + swipe-back navigation).
- **Breakpoints:** Tailwind defaults defined but only `lg`+ is actively designed for in v1.
- **Graceful degradation:** App is desktop-only — no requirement to look good on mobile in v1.

---

## 7. Risks & unknowns

- **Neon glow performance:** CSS `box-shadow` glow on many elements can cause paint performance issues, especially on large lists. Apply glow only on interactive state (hover/focus), never as a static style on list items.
- **Orbitron readability at small sizes:** Orbitron is designed for display use. At `text-xs` or `text-sm` it can become hard to read. Use Inter for all body/UI text; Orbitron only for headings and app icon labels.
- **Desktop OS metaphor scope creep:** Draggable windows, resizable panels, taskbars, and a clock are all tempting. They are not v1. The slide-in panel model is the ceiling for v1.
- **Split-color logo:** Needs to be recreated in a vector tool (Figma, Illustrator) using `#00E5FF` + `#FF2BD6`. The current PNG is single-color.

---

## 8. Out of scope (for v1)

- Dark mode toggle (it's always dark)
- Draggable / resizable windows (slide-in panels only)
- Animated wallpaper or particle effects on the desktop
- Custom illustrations or icon artwork beyond Heroicons
- Mobile / responsive layout
- Taskbar, system clock, or OS-style menu bar
- Internationalization
- User-customizable themes or accent color picker
