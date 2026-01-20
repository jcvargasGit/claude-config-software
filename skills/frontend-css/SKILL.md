---
name: frontend-css
description: CSS expert for *.css/*.scss files. Covers Flexbox, Grid, responsive design, animations, variables, and BEM naming. Use for any CSS/styling question.
model: opus
---

# CSS Skill

Apply these CSS patterns and practices when working with stylesheets.

## Code Style

- Use meaningful class names that describe purpose
- Prefer classes over IDs for styling
- Use CSS custom properties for theming
- Mobile-first responsive design
- Maximum 2 levels of nesting (with preprocessors)

## Naming Convention (BEM)

```css
/* Block */
.card {}

/* Element */
.card__header {}
.card__body {}
.card__footer {}

/* Modifier */
.card--featured {}
.card__header--large {}
```

## Modern Layout

### Flexbox
```css
.container {
  display: flex;
  justify-content: space-between;
  align-items: center;
  gap: 1rem;
}

.item {
  flex: 1 1 auto;
}

/* Common patterns */
.center {
  display: flex;
  justify-content: center;
  align-items: center;
}

.stack {
  display: flex;
  flex-direction: column;
  gap: 1rem;
}
```

### Grid
```css
.grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
  gap: 1.5rem;
}

.sidebar-layout {
  display: grid;
  grid-template-columns: 250px 1fr;
  gap: 2rem;
}

/* Named areas */
.page {
  display: grid;
  grid-template-areas:
    "header header"
    "sidebar main"
    "footer footer";
  grid-template-columns: 250px 1fr;
  grid-template-rows: auto 1fr auto;
}
```

## Custom Properties

```css
:root {
  /* Colors */
  --color-primary: #3b82f6;
  --color-secondary: #64748b;
  --color-success: #22c55e;
  --color-error: #ef4444;

  /* Typography */
  --font-sans: system-ui, sans-serif;
  --font-mono: ui-monospace, monospace;
  --font-size-sm: 0.875rem;
  --font-size-base: 1rem;
  --font-size-lg: 1.125rem;

  /* Spacing */
  --space-xs: 0.25rem;
  --space-sm: 0.5rem;
  --space-md: 1rem;
  --space-lg: 1.5rem;
  --space-xl: 2rem;

  /* Borders */
  --radius-sm: 0.25rem;
  --radius-md: 0.5rem;
  --radius-lg: 1rem;

  /* Shadows */
  --shadow-sm: 0 1px 2px rgb(0 0 0 / 0.05);
  --shadow-md: 0 4px 6px rgb(0 0 0 / 0.1);
}

/* Dark mode */
@media (prefers-color-scheme: dark) {
  :root {
    --color-primary: #60a5fa;
  }
}
```

## Responsive Design

```css
/* Mobile-first breakpoints */
.container {
  padding: var(--space-md);
}

@media (min-width: 640px) {
  .container {
    padding: var(--space-lg);
  }
}

@media (min-width: 1024px) {
  .container {
    max-width: 1200px;
    margin: 0 auto;
  }
}

/* Container queries */
.card-container {
  container-type: inline-size;
}

@container (min-width: 400px) {
  .card {
    flex-direction: row;
  }
}
```

## Common Patterns

### Reset
```css
*,
*::before,
*::after {
  box-sizing: border-box;
  margin: 0;
  padding: 0;
}

img {
  max-width: 100%;
  display: block;
}

button {
  font: inherit;
  cursor: pointer;
}
```

### Visually Hidden (Accessibility)
```css
.visually-hidden {
  position: absolute;
  width: 1px;
  height: 1px;
  padding: 0;
  margin: -1px;
  overflow: hidden;
  clip: rect(0, 0, 0, 0);
  border: 0;
}
```

### Truncate Text
```css
.truncate {
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
}

.line-clamp {
  display: -webkit-box;
  -webkit-line-clamp: 3;
  -webkit-box-orient: vertical;
  overflow: hidden;
}
```

### Focus States
```css
:focus-visible {
  outline: 2px solid var(--color-primary);
  outline-offset: 2px;
}

button:focus:not(:focus-visible) {
  outline: none;
}
```

## Transitions & Animations

```css
/* Transitions */
.button {
  transition: background-color 150ms ease, transform 150ms ease;
}

.button:hover {
  transform: translateY(-1px);
}

/* Reduced motion */
@media (prefers-reduced-motion: reduce) {
  *,
  *::before,
  *::after {
    animation-duration: 0.01ms !important;
    transition-duration: 0.01ms !important;
  }
}
```

## Quality Checklist

When writing CSS, verify:
- [ ] Uses custom properties for colors, spacing, typography
- [ ] Mobile-first responsive approach
- [ ] No ID selectors for styling
- [ ] BEM or consistent naming convention
- [ ] Focus states for interactive elements
- [ ] Respects prefers-reduced-motion
- [ ] No magic numbers (use variables)
- [ ] Logical property usage where appropriate
