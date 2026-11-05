# Project Pulse: design and structure handoff

**Audience:** Coder implementing `app/index.html` and its data-driven states.  
**Design owner:** the dashboard stylesheet will use this markup contract. Keep these class names and `data-*` attributes stable. This is a static, contributor-facing overview, not a filtering or project-detail experience.

## Page information architecture

The initial viewport should immediately identify the page and show a useful set of project cards:

1. **Skip link** — first focusable control; jumps to `#main-content`.
2. **Dashboard header** — the product name, one concise explanation of the content, and an accessible project count that is updated after loading.
3. **Project collection** — a labelled section containing the data state and, when data is available, a responsive card list.
4. **State region** — loading, empty, or error feedback in the same location as the cards so the page never appears blank.

Use one `h1` only: **Project Pulse**. The project collection does not need an additional visible heading when the header already labels it, but it must have an accessible name.

## Required semantic HTML and deterministic hook contract

Use this structure (elements may have additional classes, but do not rename or omit the required hooks):

```html
<a class="skip-link" href="#main-content">Skip to projects</a>

<main class="dashboard" id="main-content" aria-labelledby="page-title">
  <header class="dashboard__header">
    <div class="dashboard__intro">
      <p class="eyebrow">Contributor workspace</p>
      <h1 id="page-title">Project Pulse</h1>
      <p class="dashboard__description">
        A concise view of project health, priorities, and recent contributor activity.
      </p>
    </div>
    <p class="project-count" id="project-count" aria-live="polite">
      Loading projects…
    </p>
  </header>

  <section class="project-section" aria-labelledby="projects-heading">
    <div class="project-section__heading">
      <h2 id="projects-heading">Projects</h2>
    </div>

    <div
      class="project-state"
      id="project-state"
      role="status"
      aria-live="polite"
      aria-atomic="true"
    ></div>

    <ul class="project-grid" id="project-list" aria-describedby="project-state"></ul>
  </section>
</main>
```

- `.dashboard` is the page-width and overall layout hook; `.project-grid` is the grid hook.
- Keep state messages in `#project-state`; toggle its state class rather than leaving stale messages visible. Add exactly one of `.project-state--loading`, `.project-state--empty`, or `.project-state--error` while applicable.
- Render cards as `li.project-card` children of the `ul`. This makes project count and collection semantics clear to assistive technology.
- Use `textContent` for all JSON-sourced values. Do not insert project values with `innerHTML`.
- When projects successfully render, clear the state region. Set `#project-count` to a clear text value such as “4 projects”.

### Project card markup

Render the following for every project. The specified fields map to the plan’s data model. For missing/blank data, use the stated fallback—never display `undefined`, `null`, or an empty labelled row.

```html
<li class="project-card" data-status="active" data-priority="high">
  <article class="project-card__content" aria-labelledby="project-name-1">
    <header class="project-card__header">
      <div class="project-card__identity">
        <p class="project-card__eyebrow">Project</p>
        <h3 class="project-card__title" id="project-name-1">Project name</h3>
        <p class="project-card__owner">
          <span class="sr-only">Owner: </span>Owner name
        </p>
      </div>
      <span class="status-badge status-badge--active" data-status="active">
        <span class="status-badge__icon" aria-hidden="true">●</span>
        <span class="status-badge__label">Active</span>
      </span>
    </header>

    <p class="project-card__summary">Contributor-friendly project summary.</p>

    <dl class="project-card__details">
      <div class="project-card__detail">
        <dt>Priority</dt>
        <dd>
          <span class="priority priority--high" data-priority="high">
            <span class="priority__icon" aria-hidden="true">↑</span>
            <span class="priority__label">High priority</span>
          </span>
        </dd>
      </div>
      <div class="project-card__detail project-card__detail--activity">
        <dt>Recent activity</dt>
        <dd>Activity text</dd>
      </div>
    </dl>
  </article>
</li>
```

Card anatomy, in reading order: project identity (name then owner), status, summary, priority, then recent activity. This makes urgency discoverable without hiding the explanatory content. `article` is appropriate because each card is an independent project summary; no card needs a click target in this scope.

Required card hooks:

| Purpose | Hook |
| --- | --- |
| Card and data taxonomy | `.project-card`, `data-status`, `data-priority` |
| Card interior | `.project-card__content`, `.project-card__header`, `.project-card__identity` |
| Project content | `.project-card__eyebrow`, `.project-card__title`, `.project-card__owner`, `.project-card__summary` |
| Metadata | `.project-card__details`, `.project-card__detail`, `.project-card__detail--activity` |
| Status | `.status-badge`, `.status-badge--active`, `.status-badge__icon`, `.status-badge__label` |
| Priority | `.priority`, `.priority--high`, `.priority__icon`, `.priority__label` |
| Screen-reader-only copy | `.sr-only` |

Use a unique, stable heading `id` per card (for example, based on the array index only if no stable data ID exists). Normalize known machine values to lowercase kebab case for `data-status` / `data-priority` and modifier classes.

## Status and priority taxonomy

Text labels and icons are mandatory; color is supporting information only.

| Meaning | Status class / `data-status` | Visible text and icon | Treatment |
| --- | --- | --- | --- |
| Normal work | `status-badge--active` / `active` | `● Active` | Green/teal tint, dark green text |
| Needs attention | `status-badge--at-risk` / `at-risk` | `! At risk` | Amber tint, dark amber text |
| Not started | `status-badge--planned` / `planned` | `○ Planned` | Blue tint, dark blue text |
| Unknown/unmapped | `status-badge--unknown` / `unknown` | `? [original status or Unknown status]` | Neutral gray, preserve safe original value when supplied |

| Meaning | Priority class / `data-priority` | Visible text and icon | Treatment |
| --- | --- | --- | --- |
| Highest attention | `priority--high` / `high` | `↑ High priority` | Strong left border/accent plus red or dark rose text |
| Important | `priority--medium` / `medium` | `— Medium priority` | Amber accent and text |
| Routine | `priority--low` / `low` | `↓ Low priority` | Cool blue/slate accent and text |
| Unknown/unmapped | `priority--unknown` / `unknown` | `? [original priority or Unknown priority]` | Neutral gray, no urgency implied |

Known strings should be matched case-insensitively. Any unexpected, missing, or blank value must get the `--unknown` class and `unknown` data attribute; retain nonblank original text in the visible label (for example, `? Blocked`) rather than pretending it maps to a known state. Missing required text fallbacks: **Untitled project**, **Owner not specified**, **No summary provided**, and **No recent activity provided**.

## Visual system

Use CSS custom properties so the visual language remains consistent:

```css
:root {
  --color-page: #f5f7fb;
  --color-surface: #ffffff;
  --color-text: #172033;
  --color-muted: #526079;
  --color-border: #d8dfeb;
  --color-brand: #3158d4;
  --color-focus: #1649d8;
  --shadow-card: 0 8px 24px rgb(23 32 51 / 0.10);
  --radius-card: 16px;
  --radius-control: 999px;
}
```

- **Typography:** system sans stack (`Inter, ui-sans-serif, system-ui, -apple-system, "Segoe UI", sans-serif`); body 16px / 1.5; `h1` clamp(2rem, 4vw, 2.75rem) / 1.1 and 700–750; `h2` 1.25rem / 1.25; card `h3` 1.125rem / 1.25, 700. Use tabular numerals only if counts become more prominent later.
- **Spacing:** 4px base unit. Page inline padding 24px desktop / 16px narrow; dashboard max-width 1200px; header bottom gap 32px; grid gap 20px; card padding 24px desktop / 20px mobile; intra-card gaps 8px, 12px, 16px, and 24px only.
- **Surfaces:** page is cool off-white; cards are white with a 1px `--color-border` border, `var(--radius-card)`, and `var(--shadow-card)`. Give cards a 4px priority-colored top or left accent; retain a neutral border and normal reading order so the accent is not the only signal.
- **Badges:** compact pill shape (`--radius-control`), 12–14px semibold text, 6–8px inline gap between icon and label, and sufficient padding. Status color pairs must keep text contrast at least 4.5:1.
- **Long content:** allow names, owners, activity, and summaries to wrap (`overflow-wrap: anywhere`); do not truncate required information or create horizontal scrolling.
- **Motion:** a subtle card shadow/border transition may be used, but wrap it in `@media (prefers-reduced-motion: no-preference)` and do not rely on motion for feedback.

## Responsive layout

The content must fit at 320px CSS width and at 200% zoom without horizontal page scroll.

| Viewport | Layout |
| --- | --- |
| `< 640px` | One card column. Header and card header stack naturally; status badge aligns to the start below/after the identity if needed. Metadata rows stack label/value when space is constrained. |
| `640px–959px` | Two equal card columns using `repeat(2, minmax(0, 1fr))`; maintain a 20px gap. |
| `≥ 960px` | Three equal columns using `repeat(3, minmax(0, 1fr))`; header can use `justify-content: space-between` with the count aligned at the end. |

Prefer CSS Grid with `minmax(0, 1fr)` so long labels do not overflow. Cards should stretch to equal row height only as a natural grid consequence; do not constrain text with fixed card heights.

## State layouts

The state component occupies the same collection area as the grid and uses a bordered, rounded surface with 24px padding, an icon, a heading, and an explanatory paragraph. It should remain left-aligned and readable rather than resembling a full-screen error.

- **Loading (`.project-state--loading`):** show “Loading projects” and “Project information is being retrieved.” Use a static progress glyph or accessible text; do not require animation. Keep `role="status"` and `aria-live="polite"`.
- **Empty (`.project-state--empty`):** show “No projects yet” and “Projects will appear here when they are available.” Hide the empty `ul` visually (or do not render it); update count to “0 projects”.
- **Error (`.project-state--error`):** show “Projects could not be loaded” and “Check that Project Pulse is opened through its local server, then try again.” Use `role="alert"` for this active error (set it only for the error state) and update count to “Projects unavailable”. A retry control is out of scope unless Coder adds real retry behavior.

When state changes after initial load, retain focus where the user is; do not programmatically move focus into state feedback. If a user-initiated retry is implemented later, focus may move to the resulting state message.

## Accessibility requirements

- Use landmarks and heading order shown above. The card `h3`s must be descendants of the page’s `h2` project section.
- Associate every `dt` with its corresponding `dd`; status is a labelled text badge, not an unlabeled decorative dot.
- Never communicate status or priority by color alone: retain the visible words and distinct characters/icons defined in the taxonomy.
- `.sr-only` must visually hide supplementary text without removing it from the accessibility tree. The skip link must become visibly styled when focused.
- All interactive elements, including the skip link and any future controls, need a highly visible `:focus-visible` outline (at least 3px solid `--color-focus`, 2px offset) that is not removed by reset styles.
- Meet WCAG 2.2 AA: normal text contrast ≥ 4.5:1, large text ≥ 3:1, and UI boundaries/focus indicators ≥ 3:1 against adjacent colors. Do not use pale badge text on saturated fills.
- Use real text, not CSS-generated content, for status, priority, and state messages. Decorative glyphs carry `aria-hidden="true"`.
- Respect user font-size settings, reduced-motion preferences, keyboard navigation, and text reflow. Avoid hover-only information and fixed heights.

## CSS implementation checklist

- [ ] Define the shared tokens, system font stack, baseline box sizing, page background, and `.sr-only` / `.skip-link` utilities.
- [ ] Style `.dashboard` as centered, padded, max-width content and establish clear header hierarchy.
- [ ] Implement `.project-grid` responsive Grid breakpoints and `.project-card` surface with `border-radius` and `box-shadow`.
- [ ] Style card identity, summary, description-list metadata, and wrapping safeguards.
- [ ] Implement known and `--unknown` variants for `.status-badge` and `.priority`, including visible text/icon support and priority accent.
- [ ] Implement `.project-state` loading, empty, and error layouts with their modifier classes.
- [ ] Add robust `:focus-visible`, skip-link, `prefers-reduced-motion`, and narrow-screen rules.
- [ ] Check at 320px, 640px, 960px, keyboard-only navigation, 200% zoom, and with long/unknown data.
