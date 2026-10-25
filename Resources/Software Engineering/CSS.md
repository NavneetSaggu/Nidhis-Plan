# CSS

Here’s a very simple page that uses CSS, and illustrates it’s components

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>JS Interaction Demo</title>
    <!-- Here "body", ".status-box" are all called "selectors" -->
    <!--  The attributes listed here are called "properties"   -->
    <!-- One single property: value pair is called a "declaration" -->
    <!-- The complete set of declarations against a selector is called "rule set" -->
    <style>
		    :root {
				    --main-blue: #3b82f6
		    }
		    
        body {
            font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif;
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: center;
            height: 100vh;
            background-color: #f8fafc;
            gap: 20px;
        }

        /* The base class we discussed */
        .status-box {
            background-color: #e2e8f0;
            color: #475569;
            padding: 20px;
            border-radius: 12px;
            border: 2px solid #cbd5e1;
            font-weight: 600;
            transition: all 0.3s ease;
            width: 250px;
            text-align: center;
            box-shadow: 0 4px 6px -1px rgb(0 0 0 / 0.1);
        }

        /* An extra class we can "toggle" with JS */
        .is-active {
            background-color: #dcfce7;
            color: #166534;
            border-color: #86efac;
            transform: scale(1.05);
        }

        .controls {
            display: flex;
            gap: 10px;
        }

        button {
            padding: 8px 16px;
            border-radius: 6px;
            border: 1px solid #cbd5e1;
            background: white;
            cursor: pointer;
            font-weight: 500;
        }

        button:hover {
            background-color: #f1f5f9;
        }
    </style>
</head>
<body>

    <!-- The HTML element JavaScript will target -->
    <div id="myStatus" class="status-box">
        System Standby
    </div>

    <div class="controls">
        <button id="textBtn">Update Text</button>
        <button id="stateBtn">Toggle State</button>
    </div>

    <script>
        // 1. SELECTING the element
        // We can find it by its ID or by its Class name
        const statusElement = document.querySelector('.status-box');
        const textBtn = document.getElementById('textBtn');
        const stateBtn = document.getElementById('stateBtn');

        // 2. INTERACTING: Changing Content
        textBtn.addEventListener('click', () => {
            statusElement.textContent = "Status Updated!";
        });

        // 3. INTERACTING: Toggling Classes
        // This is the most professional way to change styles
        stateBtn.addEventListener('click', () => {
            // This adds 'is-active' if it's missing, or removes it if it exists
            statusElement.classList.toggle('is-active');
            
            // Logic to update text based on the state
            if (statusElement.classList.contains('is-active')) {
                statusElement.textContent = "System Active";
            } else {
                statusElement.textContent = "System Standby";
            }
        });

        // 4. INTERACTING: Inline Style (Direct override)
        statusElement.addEventListener('mouseenter', () => {
            statusElement.style.cursor = 'pointer';
        });
    </script>
</body>
</html>
```

# Selectors

Here’s all the kinds of selectors (please cross verify these)

## 1. Basic Selectors

These are the core building blocks of CSS styling.

| Name | Syntax | Description | Example |
| --- | --- | --- | --- |
| **Universal** | `*` | Selects all elements. | `* { margin: 0; }` |
| **Type** | `element` | Selects by tag name. | `p { color: blue; }` |
| **Class** | `.class` | Selects by class attribute. | `.btn { padding: 10px; }` |
| **ID** | `#id` | Selects a unique ID. | `#main { width: 100%; }` |

**`.logo.react:hover`**: This is a "chained" selector. It only targets an element that has **both** the `logo` class **and** the `react` class simultaneously. If an element only has one of those classes, this rule will ignore it.
****

## 2. Attribute Selectors (Sub-types)

Attribute selectors match elements based on the exact value or partial matches of their attributes.

| Sub-type | Syntax | Description | Example |
| --- | --- | --- | --- |
| **Presence** | `[attr]` | Has the attribute. | `[title] { border-bottom: 1px dotted; }` |
| **Exact Match** | `[attr="val"]` | Exactly matches "val". | `input[type="password"] { color: red; }` |
| **Word Match** | `[attr~="val"]` | List containing "val". | `[class~="special"] { glow: auto; }` |
| **Prefix Match** | `[attr | ="val"]` | Is "val" or starts "val-". |
| **Starts With** | `[attr^="val"]` | Begins with "val". | `a[href^="https"] { color: green; }` |
| **Ends With** | `[attr$="val"]` | Ends with "val". | `a[href$=".pdf"] { font-weight: bold; }` |
| **Contains** | `[attr*="val"]` | Contains "val" anywhere. | `[id*="user-"] { background: #eee; }` |

## 3. Combinators

Combinators define the relationship between multiple selectors.

- **Descendant (`A B`):** `article p { line-height: 1.5; }` (All `<p>` inside `<article>`)
- **Child (`A > B`):** `nav > ul { display: flex; }` (Only `<ul>` that is a direct child of `<nav>`)
- **Adjacent Sibling (`A + B`):** `h2 + p { margin-top: 0; }` (The first `<p>` immediately following an `<h2>`)
- **General Sibling (`A ~ B`):** `img ~ p { font-style: italic; }` (All `<p>` elements following an `<img>` under the same parent)

## 4. Pseudo-classes (Sub-types)

### A. User Action Pseudo-classes

- `:hover` — `button:hover { opacity: 0.8; }`
- `:active` — `a:active { color: red; }` (While the link is being clicked)
- `:focus` — `input:focus { outline: 2px solid blue; }`

### B. Structural Pseudo-classes

- **`:first-child`**: Targets an element only if it is the very first child of its parent. (e.g., `li:first-child` bolds only the first item in a list).
- **`:last-child`**: Targets an element only if it is the final child of its parent. (e.g., `li:last-child` is often used to remove the bottom border from the last list item).
- **`:nth-child(n)`**: Targets elements based on a specific number, formula, or keyword like `even` or `odd`. (e.g., `tr:nth-child(even)` is used for zebra-striping table rows).
- **`:only-child`**: Targets an element only if it has no siblings at all within its parent. (e.g., `span:only-child` turns purple only if it's the lone occupant of its container).
- **`:root`**: Targets the highest-level element in the document, which is the `<html>` tag. It is the standard place to define global CSS variables.

### C. UI State Pseudo-classes

- **`:checked`**: Targets radio buttons, checkboxes, or option elements that are currently toggled "on."
    - **Example:** `input:checked + label { text-decoration: line-through; }` (Crosses out the text of a label when its checkbox is checked).
- **`:disabled`**: Targets any form element (like a button or input) that has the `disabled` attribute and cannot be interacted with.
    - **Example:** `button:disabled { cursor: not-allowed; }` (Changes the mouse cursor to a "block" icon when hovering over a disabled button).
- **`:valid / :invalid`**: Targets form elements based on whether their current value meets the requirements (like `required`, `type="email"`, or `pattern`).
    - **Example:** `input:invalid { border: 1px solid red; }` (Highlights an input field in red if the user has entered data that doesn't match the required format).

### D. Logical Pseudo-classes (Functional)

- `:not()` — `div:not(.header) { padding: 10px; }` (All divs except those with the class `.header`)
- `:is()` — `:is(h1, h2, h3) { color: navy; }` (Shorthand for selecting multiple headings)
- `:where()` — `:where(article, section) p { margin: 1em; }` (Same as `:is`, but does not add to the specificity score)

## 5. Pseudo-elements (`::`)

- **`::before / ::after`**: Injects decorative or functional content before or after an element's actual content. It requires the `content` property to work.
    - **Example:** `blockquote::before { content: "“"; }` (Automatically adds a large opening quotation mark before every blockquote).
- **`::first-letter`**: Targets only the very first character of a block-level element (like a paragraph).
    - **Example:** `p::first-letter { font-size: 200%; }` (Creates a "drop cap" effect by making the first letter of a paragraph twice as large).
- **`::selection`**: Styles the portion of the text that a user has highlighted or dragged their cursor across.
    - **Example:** `::selection { background: yellow; }` (Changes the default blue highlight color to yellow when a user selects text).
- **`::placeholder`**: Targets the temporary "hint" text inside an input field or textarea before a user starts typing.
    - **Example:** `input::placeholder { color: #999; }` (Makes the placeholder text a light grey so it's clearly distinguished from user-entered text).

## 6. Grouping Selectors

- **Syntax:** `A, B, C`
- **Example:** `h1, h2, .title { font-family: 'Helvetica', sans-serif; }`

## 7. Selector Specificity

1. **Inline Styles** (1000)
2. **IDs** (100)
3. **Classes, Attributes, Pseudo-classes** (10)
4. **Elements, Pseudo-elements** (1)

*Note: The universal selector (`*`) and combinators (`+`, `>`, `~`) have no specificity value.*

# Custom Properties (Variables)

## 1. Overview

CSS Variables, officially known as **Custom Properties**, allow developers to store specific values (colors, fonts, sizes) in one place and reuse them throughout a stylesheet. Unlike preprocessor variables (Sass/Less), CSS variables are native to the browser, reactive, and accessible via JavaScript.

## 2. Where they are defined (The :root scope)

While variables can be defined in any selector, they are most commonly placed in the `:root` pseudo-class.

- **Why `:root`?** Because `:root` matches the `<html>` element, variables defined here are globally scoped and inherited by every other element on the page.

```
:root {
  --primary-color: #3498db;
  --base-padding: 1rem;
}

```

## 3. How they are used

To use a variable, you wrap the custom property name in the `var()` function.

- **Syntax:** `property: var(--variable-name, fallback-value);`
- **Example:**
    
    ```
    .card {
      background-color: var(--primary-color);
      padding: var(--base-padding);
    }
    
    ```
    

## 4. When to use them

CSS Variables are best utilized in the following scenarios:

### A. Theming and Dark Mode

Variables make switching themes trivial. By simply updating the value of a few variables, the entire UI changes.

```
/* Update variables via a class or media query */
[data-theme='dark'] {
  --primary-color: #2c3e50;
  --text-color: #ffffff;
}

```

### B. Avoiding "Magic Numbers"

If you use a specific shade of blue or a specific border-radius dozens of times, using a variable ensures consistency. If the brand color changes, you only update it in one line of code.

### C. Dynamic Updates with JavaScript

Since these are part of the DOM, JavaScript can update them in real-time.

```
// Change the primary color based on user input
document.documentElement.style.setProperty('--primary-color', 'red');

```

## 5. Key Advantages

1. **Maintainability:** Centralizes design tokens (colors, spacing).
2. **Readability:** `-main-header-spacing` is more descriptive than `24px`.
3. **Inheritance:** Variables follow the CSS cascade, allowing for "contextual" overrides.
4. **No Compilation:** Unlike Sass, they work directly in the browser without extra tools.

# Media Queries

## 1. Overview

Media Queries are a core feature of CSS3 that allow content to adapt to different conditions, such as screen resolution, viewport width, or device orientation. They are the fundamental building block of **Responsive Web Design (RWD)**.

## 2. Basic Syntax

A media query consists of a media type and at least one expression that limits the style sheets' scope using media features.

- **Syntax:**
    
    ```
    @media media-type and (media-feature) {
      /* CSS rules go here */
    }
    
    ```
    
- **Example:**
    
    ```
    @media screen and (max-width: 768px) {
      body {
        background-color: lightblue;
      }
    }
    
    ```
    

## 3. Common Media Features

Media queries can test for many different features of the user agent and device:

- **Width/Height:** `min-width`, `max-width`, `min-height`, `max-height`.
- **Orientation:** `portrait` or `landscape`.
- **Resolution:** Matches the pixel density of the output device (e.g., `min-resolution: 300dpi`).
- **Preferences:** `prefers-color-scheme` (dark/light mode) or `prefers-reduced-motion`.

## 4. Logical Operators

You can combine multiple conditions using logical operators:

- **and:** Combines multiple features.
- **not:** Negates a media query.
- **only:** Prevents older browsers from applying the styles (rarely needed today).
- **comma (or):** Separates multiple queries; if any are true, the styles apply.

## 5. Mobile-First Strategy

The industry standard is to write styles for the smallest screens first (outside of any media query) and then use `min-width` queries to add complexity for larger screens.

```
/* Base styles for mobile */
.container { width: 100%; }

/* Tablet and Desktop additions */
@media (min-width: 768px) {
  .container { width: 750px; }
}

@media (min-width: 1200px) {
  .container { width: 1170px; }
}

```

## 6. Key Advantages

1. **Flexibility:** Allows a single HTML codebase to look great on phones, tablets, and desktops.
2. **Performance:** By using the `media` attribute on `<link>` tags, browsers can prioritize downloading only the CSS needed for the current device.
3. **Accessibility:** Can be used to adjust font sizes and layouts for users with different visual needs.