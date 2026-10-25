# HTML

# Dev Tools Help

## Debugging Ephemeral Components

Loss of focus by navigating away from the page can cause tooltips / toasts to disappear. To disappear such ephemeral components you can try these methods.

### 1. Pause Using Debugging

Press `Cmd + \` on Mac to pause the browser.

### 2. Emulate a Focused Page

Many pop-ups disappear because they detect that you clicked "away" into the DevTools window (a `blur` event). You can tell the browser to pretend the page is always focused.

1. In DevTools, press `Ctrl` + `Shift` + `P` (Windows) or `Cmd` + `Shift` + `P` (Mac) to open the **Command Menu**.
2. Type **"Emulate a focused page"** and select it.
3. Now, even when you click inside DevTools to expand the HTML, the website thinks it still has focus and won't trigger the "close" logic for the pop-up.

# Basic Dynamic Webpage

```jsx
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>JS Interaction Demo</title>
    <style>
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

# Webpage With Explicit Shadow DOM Injection

```jsx
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Understanding Shadow DOM</title>
    <style>
        body {
            font-family: system-ui, -apple-system, sans-serif;
            padding: 2rem;
            line-height: 1.5;
            max-width: 800px;
            margin: 0 auto;
            background-color: #f0f2f5;
        }

        .container {
            background: white;
            padding: 2rem;
            border-radius: 12px;
            box-shadow: 0 4px 6px rgba(0,0,0,0.05);
        }

        /* GLOBAL STYLE: 
           This normally affects ALL <h3> tags on the page. 
        */
        h3 {
            color: #ef4444; /* Bright Red */
            text-transform: uppercase;
            border-bottom: 2px solid #ef4444;
        }

        .comparison {
            display: grid;
            grid-template-columns: 1fr 1fr;
            gap: 20px;
            margin-top: 20px;
        }

        .box {
            padding: 15px;
            border: 1px dashed #ccc;
            border-radius: 8px;
        }

        code {
            background: #eee;
            padding: 2px 4px;
            border-radius: 4px;
        }
    </style>
</head>
<body>

    <div class="container">
        <h1>Shadow DOM vs. Light DOM</h1>
        <p>
            Notice the red style applied to <code>h3</code> tags in the CSS above. 
            The red style "leaks" into the Light DOM, but the Shadow DOM is protected.
        </p>

        <div class="comparison">
            <!-- 1. LIGHT DOM (Standard) -->
            <div class="box">
                <p><strong>Standard (Light) DOM</strong></p>
                <div id="light-root">
                    <h3>Header One</h3>
                    <p>I am red because I am in the main document.</p>
                </div>
            </div>

            <!-- 2. SHADOW DOM HOST -->
            <div class="box">
                <p><strong>Shadow DOM</strong></p>
                <div id="shadow-host">
                    <!-- JavaScript will attach shadow content here -->
                </div>
            </div>
        </div>
    </div>

    <script>
        // --- SHADOW DOM IMPLEMENTATION ---

        // 1. Select the "Host" element (the bridge between light and shadow)
        const host = document.querySelector('#shadow-host');

        // 2. Attach a Shadow Root
        // 'mode: open' means we can still access it via JavaScript
        const shadowRoot = host.attachShadow({ mode: 'open' });

        // 3. Create content for the Shadow DOM
        // Even though we use an <h3>, the global CSS won't find it!
        shadowRoot.innerHTML = `
            <style>
                /* Internal styles ONLY stay inside this shadow root */
                h3 {
                    color: #0891b2; /* Blue */
                    border-bottom: 2px solid #0891b2;
                    text-transform: none;
                }
                p {
                    color: #334155;
                    font-style: italic;
                }
            </style>
            
            <h3>Header Two</h3>
            <p>I am blue because I have my own private styles.</p>
        `;

        // Verification in Console
        console.log("Global h3 count:", document.querySelectorAll('h3').length); 
        // Result: 1 (The shadow h3 is hidden from the global document!)
    </script>
</body>
</html>
```