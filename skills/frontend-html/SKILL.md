---
name: frontend-html
description: HTML expert for *.html files. Covers semantic elements, accessibility (a11y), forms, SEO meta tags, and ARIA attributes. Use for any HTML question.
model: opus
---

# HTML Skill

Apply these HTML patterns and practices when writing markup.

## Code Style

- Use semantic elements over generic divs
- Use meaningful class names
- Keep nesting levels shallow
- Indent with 2 spaces
- Use lowercase for tags and attributes
- Always include `alt` for images

## Semantic Structure

### Page Layout
```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Page Title</title>
</head>
<body>
  <header>
    <nav aria-label="Main navigation">
      <!-- navigation -->
    </nav>
  </header>

  <main>
    <article>
      <!-- main content -->
    </article>
  </main>

  <aside>
    <!-- sidebar content -->
  </aside>

  <footer>
    <!-- footer content -->
  </footer>
</body>
</html>
```

### Content Sections
```html
<article>
  <header>
    <h1>Article Title</h1>
    <time datetime="2024-01-15">January 15, 2024</time>
  </header>

  <section>
    <h2>Section Heading</h2>
    <p>Section content</p>
  </section>

  <footer>
    <p>Author: Jane Doe</p>
  </footer>
</article>
```

## Semantic Elements

### Text Content
```html
<p>Paragraph text</p>
<strong>Important text</strong>
<em>Emphasized text</em>
<mark>Highlighted text</mark>
<code>Inline code</code>
<abbr title="HyperText Markup Language">HTML</abbr>
<time datetime="2024-01-15T09:00">9:00 AM</time>
<address>Contact information</address>
```

### Lists
```html
<!-- Unordered -->
<ul>
  <li>Item one</li>
  <li>Item two</li>
</ul>

<!-- Ordered -->
<ol>
  <li>First step</li>
  <li>Second step</li>
</ol>

<!-- Description -->
<dl>
  <dt>Term</dt>
  <dd>Definition</dd>
</dl>
```

### Figures and Media
```html
<figure>
  <img src="chart.png" alt="Sales data showing 20% growth in Q4">
  <figcaption>Quarterly sales performance</figcaption>
</figure>

<picture>
  <source srcset="image.webp" type="image/webp">
  <source srcset="image.jpg" type="image/jpeg">
  <img src="image.jpg" alt="Description">
</picture>
```

## Forms

### Form Structure
```html
<form action="/submit" method="post">
  <fieldset>
    <legend>Personal Information</legend>

    <div class="field">
      <label for="name">Full Name</label>
      <input
        type="text"
        id="name"
        name="name"
        required
        autocomplete="name"
      >
    </div>

    <div class="field">
      <label for="email">Email Address</label>
      <input
        type="email"
        id="email"
        name="email"
        required
        autocomplete="email"
      >
    </div>
  </fieldset>

  <button type="submit">Submit</button>
</form>
```

### Input Types
```html
<input type="text" name="username">
<input type="email" name="email">
<input type="password" name="password">
<input type="tel" name="phone">
<input type="url" name="website">
<input type="number" name="quantity" min="1" max="100">
<input type="date" name="date">
<input type="search" name="query">
```

### Select and Options
```html
<label for="country">Country</label>
<select id="country" name="country">
  <option value="">Select a country</option>
  <optgroup label="North America">
    <option value="us">United States</option>
    <option value="ca">Canada</option>
  </optgroup>
  <optgroup label="Europe">
    <option value="uk">United Kingdom</option>
    <option value="de">Germany</option>
  </optgroup>
</select>
```

### Checkboxes and Radios
```html
<!-- Checkboxes -->
<fieldset>
  <legend>Select features</legend>
  <label>
    <input type="checkbox" name="features" value="dark-mode">
    Dark mode
  </label>
  <label>
    <input type="checkbox" name="features" value="notifications">
    Notifications
  </label>
</fieldset>

<!-- Radio buttons -->
<fieldset>
  <legend>Select plan</legend>
  <label>
    <input type="radio" name="plan" value="free" checked>
    Free
  </label>
  <label>
    <input type="radio" name="plan" value="pro">
    Pro
  </label>
</fieldset>
```

## Accessibility

### ARIA Landmarks
```html
<nav aria-label="Main navigation">...</nav>
<nav aria-label="Breadcrumb">...</nav>
<main>...</main>
<aside aria-label="Related articles">...</aside>
```

### ARIA States
```html
<button aria-expanded="false" aria-controls="menu">Menu</button>
<div id="menu" hidden>...</div>

<button aria-pressed="false">Toggle</button>

<div role="alert" aria-live="polite">Message sent</div>
```

### Skip Links
```html
<body>
  <a href="#main-content" class="skip-link">Skip to main content</a>
  <header>...</header>
  <main id="main-content" tabindex="-1">...</main>
</body>
```

### Accessible Buttons
```html
<!-- Icon button with label -->
<button type="button" aria-label="Close dialog">
  <svg aria-hidden="true">...</svg>
</button>

<!-- Loading state -->
<button type="submit" aria-busy="true" disabled>
  <span aria-hidden="true">Loading...</span>
  <span class="visually-hidden">Submitting form</span>
</button>
```

## Tables

```html
<table>
  <caption>Monthly Sales Report</caption>
  <thead>
    <tr>
      <th scope="col">Product</th>
      <th scope="col">Q1</th>
      <th scope="col">Q2</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th scope="row">Widget A</th>
      <td>100</td>
      <td>150</td>
    </tr>
  </tbody>
  <tfoot>
    <tr>
      <th scope="row">Total</th>
      <td>100</td>
      <td>150</td>
    </tr>
  </tfoot>
</table>
```

## Meta Tags

```html
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <meta name="description" content="Page description for SEO">
  <meta name="theme-color" content="#3b82f6">

  <!-- Open Graph -->
  <meta property="og:title" content="Page Title">
  <meta property="og:description" content="Description">
  <meta property="og:image" content="https://example.com/image.jpg">

  <!-- Favicon -->
  <link rel="icon" href="/favicon.ico" sizes="32x32">
  <link rel="icon" href="/icon.svg" type="image/svg+xml">
  <link rel="apple-touch-icon" href="/apple-touch-icon.png">
</head>
```

## Quality Checklist

When writing HTML, verify:
- [ ] Valid document structure (DOCTYPE, html, head, body)
- [ ] Language attribute on html element
- [ ] Semantic elements used appropriately
- [ ] All images have alt attributes
- [ ] Form inputs have associated labels
- [ ] Heading hierarchy is logical (h1-h6)
- [ ] Interactive elements are keyboard accessible
- [ ] ARIA used only when necessary
- [ ] Tables have proper headers and captions
