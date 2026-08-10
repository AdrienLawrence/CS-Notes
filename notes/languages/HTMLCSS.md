# HTML Overview

- HTML stands for HyperText Markup Language. It describes the meaning and structure of a web page

- HTML is not a programming language because it does not contain general-purpose logic. It is a markup language made from elements and attributes

- CSS stands for Cascading Style Sheets. It controls presentation, layout, spacing, color, and responsive behavior

- JavaScript controls interactive behavior. A basic division is:

    HTML = Content and meaning

    CSS = Appearance and layout

    JavaScript = Behavior

- A valid starting document is:

    ```html
    <!doctype html>
    <html lang="en">
        <head>
            <meta charset="UTF-8" />
            <meta name="viewport" content="width=device-width, initial-scale=1.0" />
            <title>Page Title</title>
            <link rel="stylesheet" href="styles.css" />
        </head>
        <body>
            <h1>Hello World</h1>
            <script src="app.js" defer></script>
        </body>
    </html>
    ```

- `<!doctype html>` selects modern standards mode

- `<html>` is the root element. Its `lang` attribute helps screen readers and search engines

- `<head>` contains page metadata and linked resources which are not normal page content

- `<body>` contains content presented to the user

- Most elements have an opening tag, content, and closing tag: `<p>Text</p>`

- Void elements do not contain children, including `img`, `input`, `meta`, `link`, `br`, and `hr`

- Attributes provide extra information: `<a href="/about" class="nav-link">About</a>`

- Use lowercase element and attribute names, quote attribute values, and indent nested content consistently

# Semantic HTML

- Semantic elements describe the purpose of content instead of only its visual appearance

- Main page landmarks:

    `<header>`: Introductory content for a page or section

    `<nav>`: Major navigation links

    `<main>`: The unique primary content of the page. Usually one per document

    `<section>`: A named thematic section

    `<article>`: Independently reusable content such as a post or product card

    `<aside>`: Related but secondary content

    `<footer>`: Closing information for a page or section

- Text elements:

    `<h1>` through `<h6>`: Hierarchical headings

    `<p>`: Paragraph

    `<strong>`: Strong importance

    `<em>`: Stress emphasis

    `<blockquote>` / `<q>`: Block or inline quotation

    `<code>` / `<pre>`: Inline code or whitespace-preserved block

    `<time datetime="2026-08-10">August 10, 2026</time>`: Machine-readable time

    `<abbr title="Application Programming Interface">API</abbr>`: Abbreviation

- Lists:

    `<ul>`: Unordered list

    `<ol>`: Ordered list

    `<dl>` with `<dt>` and `<dd>`: Term-description list

- `<div>` and `<span>` have no semantic meaning. Use them when no meaningful element fits, usually for styling or scripting hooks

- Heading levels communicate document structure. Do not choose a heading level because of its default size; use CSS for size

- Semantic HTML improves accessibility, search indexing, maintainability, browser features, and keyboard navigation

# Links, Images, and Media

- Links use `<a href="...">`. Link text should explain the destination without relying on surrounding text

    ```html
    <a href="/reports/annual.pdf" download>Download the annual report</a>
    <a href="https://example.com" target="_blank" rel="noopener noreferrer">
        External documentation
    </a>
    ```

- Relative URLs are resolved from the current site. Absolute URLs include the complete scheme and domain

- Fragment links point to an element ID: `<a href="#pricing">Pricing</a>`

- Images require useful alternative text:

    ```html
    <img
        src="team.jpg"
        alt="The engineering team standing outside the office"
        width="1200"
        height="800"
        loading="lazy"
    />
    ```

- Use `alt=""` for decorative images so assistive technology ignores them

- Width and height reserve space and reduce layout shift

- `<picture>` and `srcset` allow responsive image sources and art direction

    ```html
    <picture>
        <source media="(min-width: 800px)" srcset="hero-wide.webp" />
        <img src="hero-small.webp" alt="Mountain trail at sunrise" />
    </picture>
    ```

- `<figure>` groups media with a `<figcaption>`

- `<audio>` and `<video>` should include controls and accessible alternatives such as captions or transcripts

- `<iframe>` embeds another page. Give it a title, restrict it with `sandbox` where possible, and avoid unnecessary embeds

# Forms

- Forms collect and submit user input

    ```html
    <form action="/users" method="post">
        <div>
            <label for="email">Email</label>
            <input id="email" name="email" type="email" required autocomplete="email" />
        </div>

        <div>
            <label for="role">Role</label>
            <select id="role" name="role">
                <option value="student">Student</option>
                <option value="teacher">Teacher</option>
            </select>
        </div>

        <button type="submit">Create account</button>
    </form>
    ```

- `action` is the submission URL. `method` is normally `get` for retrieval or `post` for a mutation

- A control needs a `name` to appear in submitted form data

- Connect every input to a visible `<label>` using matching `for` and `id` values

- Useful input types include `text`, `email`, `password`, `number`, `date`, `checkbox`, `radio`, `file`, `url`, `tel`, and `search`

- `<textarea>` accepts multi-line text. `<select>` provides listed options

- Group related controls with `<fieldset>` and `<legend>`

- Common validation attributes include `required`, `min`, `max`, `minlength`, `maxlength`, `step`, and `pattern`

- Browser validation improves the interface but does not secure a server. Validate every submitted value again on the server

- Use `enctype="multipart/form-data"` when uploading files

- A `<button>` inside a form defaults to submit. Set `type="button"` for a non-submission action

- `disabled` controls are not submitted. `readonly` controls are submitted but cannot be edited

# Tables

- Tables represent two-dimensional data, not page layout

    ```html
    <table>
        <caption>Quarterly revenue</caption>
        <thead>
            <tr>
                <th scope="col">Quarter</th>
                <th scope="col">Revenue</th>
            </tr>
        </thead>
        <tbody>
            <tr>
                <th scope="row">Q1</th>
                <td>$10,000</td>
            </tr>
        </tbody>
    </table>
    ```

- `<caption>` names the table. `<th scope>` communicates row and column relationships to assistive technology

- Use `<thead>`, `<tbody>`, and `<tfoot>` to group rows

- `colspan` and `rowspan` allow a cell to cover multiple columns or rows, but complicated layouts can become difficult to understand

# CSS Fundamentals

- A CSS rule contains a selector and declarations:

    ```css
    .card {
        padding: 1rem;
        color: #1f2937;
        background-color: white;
        border: 1px solid #d1d5db;
    }
    ```

- CSS may be loaded through an external stylesheet, a `<style>` element, or an inline `style` attribute. External stylesheets are usually easiest to maintain

- Common selector types:

    `p`: Element selector

    `.card`: Class selector

    `#main`: ID selector

    `[disabled]`: Attribute selector

    `.nav a`: Descendant combinator

    `.list > li`: Direct child combinator

    `h2 + p`: Immediately following sibling

    `h2 ~ p`: Later siblings

- Pseudo-classes describe state or position: `:hover`, `:focus-visible`, `:checked`, `:disabled`, `:first-child`, `:nth-child()` and `:not()`

- Pseudo-elements style generated parts: `::before`, `::after`, `::first-line`, `::selection`, and `::placeholder`

- Prefer classes for reusable styles. IDs are usually better as unique document or JavaScript anchors

# Cascade, Specificity, and Inheritance

- The cascade decides which declaration wins through origin, importance, cascade layer, specificity, scoping proximity, and source order

- A simplified specificity order is:

    Inline style > ID > class/attribute/pseudo-class > element/pseudo-element

- If specificity is equal, the declaration appearing later wins

- `!important` reverses the normal importance order and makes future overrides harder. Use it only for a deliberate exception

- Inherited properties such as `color` and `font-family` normally pass from parent to child. Layout properties such as `margin` do not

- `inherit`, `initial`, `unset`, `revert`, and `revert-layer` explicitly control the value source

- Cascade layers organize style priority:

    ```css
    @layer reset, base, components, utilities;
    ```

- Keep selector specificity low and predictable. A class-based component system is easier to override than deeply nested selectors

# Box Model and Sizing

- Every element is a box containing content, padding, border, and margin

- The default `content-box` calculates declared width from content only. `border-box` includes padding and border

    ```css
    *,
    *::before,
    *::after {
        box-sizing: border-box;
    }
    ```

- Margin creates outer space. Padding creates inner space. Borders lie between them

- Vertical margins of normal block elements may collapse. Flex and Grid items do not use margin collapsing in the same way

- `overflow` controls content outside a box: `visible`, `hidden`, `clip`, `auto`, or `scroll`

- Common units:

    `px`: CSS pixel

    `%`: Percentage of a related containing value

    `rem`: Root font size

    `em`: Current element font size, or parent font size when used on `font-size`

    `vw`, `vh`, `dvw`, `dvh`: Viewport dimensions

    `ch`: Approximate width of the `0` character

    `fr`: Fraction of remaining Grid space

- `min()`, `max()`, and `clamp()` create fluid constraints

    ```css
    h1 {
        font-size: clamp(2rem, 5vw, 4rem);
    }
    ```

# Normal Flow and Positioning

- Normal flow lays block content vertically and inline content within lines

- `display` controls the outer and inner layout behavior

- Common display values include `block`, `inline`, `inline-block`, `flex`, `grid`, and `none`

- Position values:

    `static`: Normal positioning

    `relative`: Remains in flow and becomes a containing block for positioned descendants

    `absolute`: Removed from normal flow and positioned against a containing block

    `fixed`: Positioned against the viewport

    `sticky`: Behaves normally until reaching a specified scroll offset

- `z-index` only compares elements within compatible stacking contexts. Opacity, transforms, positioning, isolation, and other properties may create new stacking contexts

- Use positioning for overlays and special alignment, not as the primary page layout system

# Flexbox

- Flexbox is a one-dimensional layout system for a row or column

    ```css
    .toolbar {
        display: flex;
        align-items: center;
        justify-content: space-between;
        gap: 1rem;
        flex-wrap: wrap;
    }
    ```

- Main container properties:

    `flex-direction`: Main axis direction

    `justify-content`: Alignment along the main axis

    `align-items`: Alignment along the cross axis

    `align-content`: Distribution of multiple wrapped lines

    `gap`: Space between items

    `flex-wrap`: Whether items wrap

- Main item properties:

    `flex-grow`: Share of available positive space

    `flex-shrink`: Ability to shrink

    `flex-basis`: Starting main-axis size

    `flex`: Shorthand for grow, shrink, and basis

    `align-self`: Per-item cross-axis alignment

    `order`: Visual order

- A common equal-width row uses `.item { flex: 1 1 0; }`

- Visual reordering does not change keyboard or screen-reader order. Keep DOM order meaningful

# Grid

- Grid is a two-dimensional layout system for rows and columns

    ```css
    .gallery {
        display: grid;
        grid-template-columns: repeat(auto-fit, minmax(16rem, 1fr));
        gap: 1.5rem;
    }
    ```

- `grid-template-columns` and `grid-template-rows` define explicit tracks

- `repeat()` repeats track definitions. `minmax()` gives a track a minimum and maximum

- `auto-fit` collapses empty repeated tracks. `auto-fill` preserves possible empty tracks

- Items can be placed with line numbers, named lines, or named template areas

    ```css
    .page {
        display: grid;
        grid-template-areas:
            "header header"
            "sidebar main"
            "footer footer";
        grid-template-columns: 16rem 1fr;
    }
    ```

- `subgrid` lets a nested grid use track sizing from its parent

- Grid controls visual layout without changing DOM order. Preserve a logical source order for accessibility

# Responsive Design

- Responsive design adapts content to different screen sizes, input types, preferences, and containers

- Start with a usable small-screen layout, then add space when available

    ```css
    .layout {
        display: block;
    }

    @media (min-width: 48rem) {
        .layout {
            display: grid;
            grid-template-columns: 16rem 1fr;
        }
    }
    ```

- Media queries inspect the viewport or device preference

- Container queries inspect the space available to a component

    ```css
    .card-list {
        container-type: inline-size;
    }

    @container (min-width: 40rem) {
        .card {
            grid-template-columns: 12rem 1fr;
        }
    }
    ```

- Use `max-width`, fluid units, wrapping, and intrinsic sizing before adding many breakpoints

- Respect preferences such as `prefers-reduced-motion`, `prefers-color-scheme`, and `prefers-contrast`

# Typography, Color, and Visual Effects

- Define a readable font stack, line height, and maximum text width

    ```css
    body {
        font-family: system-ui, sans-serif;
        line-height: 1.5;
        color: #111827;
    }

    p {
        max-width: 70ch;
    }
    ```

- Use `font-weight`, `font-style`, `letter-spacing`, `text-align`, `text-decoration`, and `text-transform` intentionally

- CSS colors may use named colors, hex, `rgb()`, `hsl()`, and newer color spaces such as `oklch()`

- Custom properties create reusable values and participate in the cascade

    ```css
    :root {
        --color-brand: #1d4ed8;
        --space-md: 1rem;
    }

    .button {
        background: var(--color-brand);
        padding: var(--space-md);
    }
    ```

- Gradients are generated images. Shadows, filters, blend modes, masks, and clipping can create visual effects but should not hide important content

- Transitions animate a change between states. Keyframes define multi-step animations

    ```css
    .button {
        transition: transform 150ms ease, background-color 150ms ease;
    }

    .button:hover {
        transform: translateY(-2px);
    }

    @media (prefers-reduced-motion: reduce) {
        *, *::before, *::after {
            scroll-behavior: auto !important;
            animation-duration: 0.01ms !important;
            transition-duration: 0.01ms !important;
        }
    }
    ```

- Animate `transform` and `opacity` when possible because they commonly avoid repeated layout calculations

# Accessibility

- Accessibility begins with semantic HTML and a logical document order

- Every interactive element must be usable with a keyboard. Prefer native buttons, links, inputs, and dialogs over recreating them with generic elements

- Visible focus indicators are required. Use `:focus-visible` to style keyboard focus without hiding it

- Text and important interface controls require sufficient color contrast

- Do not communicate meaning through color alone

- Every form control needs an accessible name. Errors should identify the field and explain how to fix it

- Use ARIA only when native HTML cannot express the necessary role or state. ARIA changes accessibility semantics but does not add behavior

- `aria-live` announces changing status messages. Use it carefully to avoid excessive interruptions

- Modal dialogs must manage focus, label themselves, close by keyboard, and prevent interaction with content behind them. The native `<dialog>` element provides useful behavior

- Test with keyboard-only navigation, zoom, reduced motion, high contrast, and at least one screen reader

# CSS Organization and Debugging

- Common organization approaches include utility classes, component classes, CSS Modules, BEM naming, design tokens, and cascade layers

- Choose one predictable approach. Mixing many unrelated specificity systems causes fragile styles

- A practical layer order is reset, base, components, and utilities

- Browser DevTools can inspect computed styles, overridden rules, Grid/Flex overlays, box dimensions, accessibility trees, and responsive breakpoints

- When a rule does not apply, check selector matching, syntax, inheritance, specificity, source order, media queries, and whether the property works for that display mode

- When layout overflows, inspect fixed widths, unbreakable text, intrinsic image sizes, `min-width: auto` on flex/grid items, and viewport units

- Use a CSS reset or normalization deliberately. Do not remove useful native focus and form behavior without replacing it accessibly

# Quick Reference

- Structure and meaning -> HTML

- Reusable visual styling -> Class selector

- One-dimensional alignment -> Flexbox

- Two-dimensional page or card layout -> Grid

- Component responds to its own width -> Container query

- Page responds to viewport -> Media query

- Overlay attached to a container -> Absolute positioning inside a relative parent

- Persistent viewport control -> Fixed positioning

- Scroll-aware header -> Sticky positioning

- Reusable theme value -> CSS custom property

- Interactive action -> `<button>`

- Navigation -> `<a>`

- Grouped page content -> Semantic landmark or section

- Generic wrapper with no semantic match -> `<div>` or `<span>`

# Tailwind CSS

- Tailwind CSS is a utility-first CSS framework. Small classes directly express layout and visual rules in markup

    ```html
    <article class="mx-auto max-w-xl rounded-2xl border border-slate-200 bg-white p-6 shadow-sm">
        <h2 class="text-xl font-semibold text-slate-900">Notebook</h2>
        <p class="mt-2 leading-7 text-slate-600">Study notes and generated quizzes.</p>
        <button class="mt-4 rounded-lg bg-blue-600 px-4 py-2 font-medium text-white hover:bg-blue-700 focus-visible:outline-2 focus-visible:outline-offset-2 focus-visible:outline-blue-600">
            Open
        </button>
    </article>
    ```

- Utility groups:

    Layout: `block`, `flex`, `grid`, `hidden`, `relative`, `absolute`

    Spacing: `p-4`, `px-6`, `mt-2`, `gap-4`, `space-y-3`

    Sizing: `w-full`, `max-w-xl`, `min-h-screen`, `size-10`

    Typography: `text-lg`, `font-semibold`, `leading-7`, `tracking-tight`

    Color: `text-slate-900`, `bg-blue-600`, `border-slate-200`

    Effects: `shadow`, `opacity-50`, `blur`, `ring-2`

    Borders: `border`, `rounded-lg`, `divide-y`

    Transforms/transitions: `translate-x-2`, `scale-95`, `transition`

- Variant prefixes apply a utility under a condition

    `hover:bg-blue-700`: Pointer hover

    `focus-visible:ring-2`: Keyboard-like visible focus

    `disabled:opacity-50`: Disabled state

    `first:mt-0`: First-child state

    `group-hover:block`: State from a parent with `group`

    `peer-checked:...`: State from a sibling with `peer`

    `dark:bg-slate-900`: Dark-mode variant

    `md:grid-cols-2`: Medium breakpoint and above

- Tailwind breakpoints are mobile-first. An unprefixed class applies at every size; `md:` overrides it at the medium minimum width

    ```html
    <div class="grid grid-cols-1 gap-4 md:grid-cols-2 xl:grid-cols-4"></div>
    ```

- Do not use `sm:` to mean "small screens only". It means the rule begins at the small minimum width and continues upward

- Arbitrary values handle one-off design values: `grid-cols-[16rem_1fr]`, `top-[117px]`, and `bg-[#1e40af]`

- Arbitrary values should remain exceptions. Repeated values belong in theme tokens or custom utilities

- Tailwind v4 uses a CSS-first setup

    ```css
    @import "tailwindcss";

    @theme {
        --color-brand-500: oklch(0.62 0.2 255);
        --font-display: "Inter", sans-serif;
        --breakpoint-3xl: 120rem;
    }
    ```

- Theme variables create matching utilities and variants, such as `bg-brand-500` or `font-display`

- `@source` explicitly registers source paths not found through automatic detection

- `@utility` defines a reusable custom utility which works with variants

    ```css
    @utility content-auto {
        content-visibility: auto;
    }
    ```

- `@custom-variant` defines a project-specific condition such as a data-attribute theme

- `@apply` inserts utility declarations into custom CSS. Use it sparingly because normal reusable UI components usually preserve the utility system more clearly

- Dynamic class fragments are not reliably detected:

    ```tsx
    // Avoid constructing incomplete class names
    const color = `bg-${name}-600`

    // Map complete known strings instead
    const colors = {
        blue: 'bg-blue-600 hover:bg-blue-700',
        red: 'bg-red-600 hover:bg-red-700',
    }
    ```

- Class-merging tools may help compose conditional component classes, but they should not hide an unclear variant design

- A reusable component can map explicit variants:

    ```tsx
    const variants = {
        primary: 'bg-blue-600 text-white hover:bg-blue-700',
        secondary: 'border border-slate-300 bg-white text-slate-900 hover:bg-slate-50',
    }

    export function Button({ variant = 'primary', ...props }) {
        return (
            <button
                className={`rounded-lg px-4 py-2 font-medium ${variants[variant]}`}
                {...props}
            />
        )
    }
    ```

- Tailwind does not replace CSS knowledge. The classes map to real CSS behavior including the cascade, box model, Flexbox, Grid, positioning, and accessibility

- Keep focus states, semantic elements, contrast, responsive behavior, reduced motion, and readable DOM order even when styling is fast

- Prefer design tokens and a small component variant system over unique arbitrary combinations on every screen

- When a style does not apply, inspect generated CSS, competing classes, responsive/state prefixes, class detection, and parent layout constraints
