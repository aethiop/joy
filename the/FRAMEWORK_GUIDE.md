# THE Framework - Complete Developer Guide

## Table of Contents
1. [Introduction](#introduction)
2. [Architecture Overview](#architecture-overview)
3. [Getting Started](#getting-started)
4. [Core Concepts](#core-concepts)
5. [CSS System](#css-system)
6. [API Reference](#api-reference)
7. [Examples](#examples)
8. [Best Practices](#best-practices)

---

## Introduction

**THE** (The Rendering Engine) is a security-focused, sandboxed rendering framework for the web. It provides a reactive rendering system with built-in security isolation, persistent state management, and a comprehensive CSS utility system.

### Key Features

- **Sandboxed Execution**: Code runs in isolated Web Workers and iframes for security
- **Reactive State**: Automatic view updates when state changes
- **Persistent Storage**: Built-in localStorage integration via `the.player`
- **WebGL Support**: Optional 3D rendering capabilities
- **Zero Dependencies**: No external runtime dependencies
- **Event System**: Mouse, keyboard, and touch input handling
- **CSS Utilities**: Comprehensive layout and styling classes

### Architecture Layers

```
┌─────────────────────────────────┐
│   Application Layer             │
├─────────────────────────────────┤
│   Enclave (Security Wrapper)    │  ← enclave.js
├─────────────────────────────────┤
│   Sandbox (Isolated Context)    │  ← sandbox.js
├─────────────────────────────────┤
│   Web Workers (Code Execution)  │
└─────────────────────────────────┘
```

---

## Architecture Overview

### Core Files

| File | Purpose | Size | Key Functionality |
|------|---------|------|-------------------|
| **enclave.js** | Security layer | 82 lines | Creates iframe sandbox, handles postMessage communication |
| **sandbox.js** | Main engine | 1,087 lines | Rendering engine, state management, worker spawning |
| **content.js** | Browser extension | 65 lines | Injects SecureRender into pages |
| **layout.css** | Styling system | 185 lines | Utility classes for layout and design |
| **service.js** | Service Worker | 147 lines | Caching, integrity checks, offline support |

### Security Model

THE uses a three-tier security model:

1. **Application Level**: Your HTML page
2. **Enclave Level**: An iframe that wraps the sandbox
3. **Sandbox Level**: A sandboxed iframe with 'null' origin where code executes
4. **Worker Level**: Web Workers for untrusted code execution

This isolation ensures that application code cannot directly access or tamper with user data.

---

## Getting Started

### Installation

#### As a Library

```bash
# Clone the repository
git clone [repository-url]
cd the

# Install dev dependencies (optional)
npm install .

# Start dev server
npm run dev
```

#### As a Browser Extension

1. Navigate to `chrome://extensions/`
2. Enable "Developer mode"
3. Click "Load unpacked"
4. Select the `the` folder

### Your First Application

Create an HTML file with the following structure:

```html
<!DOCTYPE html>
<html>
<head>
    <title>My First THE App</title>
</head>
<body>
    <script class="SecureRender" src="../content.js">
        // Your application code here
        the.view = "Hello World!"
    </script>
</body>
</html>
```

**Key Points:**
- The script tag MUST have `class="SecureRender"`
- The `src="../content.js"` loads the framework
- Code inside the script runs in a sandboxed environment
- `the.view` controls what is rendered

---

## Core Concepts

### The `the` Object

The global `the` object is your main API interface:

```javascript
the.view     // What to render on screen
the.player   // Persistent storage (localStorage)
the.aim      // Mouse/touch position and target
the.key      // Keyboard state
the.on       // Event handlers
the.file     // File picker interface
```

### Reactive Rendering

THE automatically updates the view when you change `the.view`:

```javascript
// Set a simple string
the.view = "Hello World!"

// Set HTML content
the.view = {
    name: 'myDiv',
    fill: 'Welcome to THE'
}

// Use a function for dynamic content
the.view = function(state) {
    return "Counter: " + (the.player.counter || 0)
}
```

### State Management with `the.player`

`the.player` is a proxy to localStorage that persists across page reloads:

```javascript
// Write data
the.player.username = "Alice"
the.player.score = 100

// Read data
console.log(the.player.username) // "Alice"

// Data persists across page reloads!
```

### The View System

The view system uses a hierarchical structure similar to the DOM:

```javascript
// Create a view element
var container = the.view({
    name: 'container',
    size: [[20, '~'], [10, '~']],  // 20em x 10em
    fill: [1, 1, 1, 1]              // White background
})

// Add to the main view
container.into(the.view)

// Create a child element
var text = the.view({
    name: 'text',
    fill: 'Hello!'
})

// Add text inside container
text.into(container)
```

### View Placement Methods

```javascript
element.into(parent)     // Add as last child
element.begin(parent)    // Add as first child
element.after(sibling)   // Add after sibling
element.before(sibling)  // Add before sibling
```

### View Properties

| Property | Type | Description | Example |
|----------|------|-------------|---------|
| `name` | String | Unique identifier | `'myElement'` |
| `fill` | String/Array | Text content or RGBA color | `'Text'` or `[1,0,0,1]` |
| `size` | Array | Width, height, depth | `[[10,'~'],[5,'~']]` |
| `grab` | Array | Position offset (x,y,z) | `[5, 10, 0]` |
| `turn` | Array | Rotation in turns (x,y,z) | `[0, 0.25, 0]` |
| `zoom` | Array | Scale (x,y,z) | `[1.5, 1.5, 1]` |
| `flow` | Array | Text direction | `['>','v']` |
| `drip` | Number | Alignment (-1 to 1) | `0` (center) |

### Units System

THE supports three unit types:

- `'~'` - em units (relative to font size)
- `'.'` - pixels
- `'%'` - percentage

```javascript
size: [
    [10, '~', 50, '~'],  // min 10em, max 50em
    [5, '~', 25, '~']    // min 5em, max 25em
]
```

---

## CSS System

THE includes a comprehensive CSS utility system in `layout.css` (185 lines). These classes provide layout, spacing, and visual utilities that work seamlessly with the rendering engine.

### Layout Classes

#### Container Classes

```html
<!-- Full viewport coverage -->
<div class="full">Content</div>

<!-- Responsive padding with auto-centering -->
<div class="pad">Content</div>

<!-- Max width constraint -->
<div class="max">Content</div>

<!-- Min width constraint -->
<div class="min">Content</div>
```

**Class Definitions:**
- `.full` - 100% width, 100vh min-height
- `.pad` - 95% width, 5% margin, centered, responsive sizing
- `.max` - 48em max-width
- `.min` - 12em min-width

#### Grid Layout

```html
<!-- Row container -->
<div class="row">
    <div class="col">Column 1</div>
    <div class="col">Column 2</div>
</div>
```

**Class Definitions:**
- `.row` - 100% width with clearfix
- `.col` - 12-24em width columns

### Alignment Classes

#### Horizontal Alignment

```html
<div class="center">Centered</div>
<div class="left">Left aligned</div>
<div class="right">Right aligned</div>
<div class="mid">Auto margins (centered)</div>
```

#### Vertical Alignment

```html
<div class="top">Top aligned</div>
<div class="low">Bottom aligned</div>
```

### Spacing Classes

```html
<!-- Margin utilities -->
<div class="rim">1% margin on all sides</div>
<div class="crack">1% bottom margin</div>
<div class="sit">0 bottom margin</div>

<!-- Padding utility -->
<div class="gap">3% padding (clamped 0.5-1.5em)</div>

<!-- Line spacing -->
<div class="stack">0 line-height (compact)</div>
```

### Visual Utilities

#### Visibility

```html
<div class="hide">Fades out with transition</div>
<div class="none">Display none</div>
```

#### Background Tints

```html
<div class="shade">10% black overlay</div>
<div class="tint">10% white overlay</div>
```

#### Animation

```html
<button class="pulse">Pulsing button</button>
```

The `.pulse` class creates a breathing effect:
- 2s infinite animation
- Fades between 100% and 50% opacity

### Interactive Elements

```html
<button class="act">Interactive Button</button>
```

The `.act` class adds:
- Pointer cursor
- 0.3s transitions
- Optimized for interactive elements

### Overflow Control

```html
<div class="leak">Overflow visible</div>
<div class="hold">Overflow hidden</div>
```

### Focus Management

```html
<div class="focus">
    Cleared floats, centered, no float
</div>
```

### Form Elements

THE automatically styles form elements with:
- Inherited backgrounds, borders, and colors
- Full-width inputs (except buttons)
- Smooth focus animations
- 30% opacity placeholders

```css
input:not([type=button]):not([type=submit]), textarea {
    width: 100%;
}

input:focus, button:focus {
    animation: pulse 2s infinite;
}
```

### CSS Architecture

The CSS follows this structure:

1. **Reset & Base Styles** (lines 1-35)
   - Viewport setup
   - Universal element resets
   - Box-sizing and transitions

2. **Element Defaults** (lines 37-79)
   - Form elements
   - Lists
   - Text elements
   - Placeholders

3. **Layout Utilities** (lines 81-161)
   - Container sizes
   - Grid system
   - Alignment
   - Spacing

4. **Visual Effects** (lines 162-185)
   - Interactive states
   - Backgrounds
   - Animations

---

## API Reference

### Core API

#### `the.view`

Controls what is rendered on screen.

```javascript
// String - displays text
the.view = "Hello World"

// Function - dynamic rendering
the.view = function(state) {
    return "Count: " + state.count
}

// Object - create view element
the.view = {
    name: 'myElement',
    fill: 'Content',
    size: [[10,'~'], [5,'~']]
}
```

#### `the.player`

Persistent storage using localStorage.

```javascript
// Set values
the.player.name = "Alice"
the.player.settings = { theme: "dark" }

// Get values
console.log(the.player.name) // "Alice"

// Values persist across page reloads
```

#### `the.aim`

Mouse/touch cursor information.

```javascript
// Properties
the.aim.x  // X position (-1 to 1, normalized viewport)
the.aim.y  // Y position (-1 to 1, normalized viewport)
the.aim.z  // Z position (for VR/3D)
the.aim.at // Name of element under cursor
```

#### `the.key`

Keyboard state tracking.

```javascript
// Check if key is pressed
if (the.key.Space) {
    console.log("Space is pressed")
}

// Check timing
if (the.key.KeyW && Date.now() - the.key.KeyW < 100) {
    console.log("W was just pressed")
}

// Double tap detection
if (the.key.KeyW && lastPress && Date.now() - lastPress < 300) {
    console.log("Double tap W")
}
```

**Key Codes:**
- Letter keys: `KeyA`, `KeyB`, ... `KeyZ`
- Numbers: `Digit0`, `Digit1`, ... `Digit9`
- Arrows: `Up`, `Down`, `Left`, `Right`
- Mouse: `M0` (left), `M1` (middle), `M2` (right)

#### `the.on`

Event handlers for user interactions.

```javascript
// Event types: see, zip, aim, tap, hop, arc, use, act

// Global event handler
the.on.tap = function(event) {
    console.log("Tap event:", event)
}

// Element-specific handler
myElement.on('tap', function(event) {
    console.log("Element tapped")
})

// Tag-based handler
the.on.tag.button = {
    tap: function(event) {
        console.log("Button tapped")
    }
}
```

#### `the.file`

File picker interface.

```javascript
// Trigger file picker
the.file.pick.into()

// Listen for file selection
the.on.tap = async function(file) {
    if (the.aim.at === 'file') {
        var data = await file.data
        console.log("File loaded:", data)
    }
}
```

### View Element Methods

#### Placement Methods

```javascript
var element = the.view({ name: 'myElement' })

// Add to parent
element.into(parent)      // As last child
element.begin(parent)     // As first child

// Relative to sibling
element.after(sibling)    // After sibling
element.before(sibling)   // Before sibling
```

#### Event Binding

```javascript
element.on('tap', function(event) {
    console.log("Element tapped!")
})

// Event types:
// tap - click/touch
// aim - hover
// see - visible
// zip - drag
```

### Math Utilities

THE includes helper math functions:

```javascript
// Linear interpolation
Math.mix(a, b, t)      // Returns a + (b - a) * t

// Inverse interpolation (get t from value)
Math.remix(a, b, v)    // Returns (v - a) / (b - a)
```

---

## Examples

### Example 1: Hello World

```javascript
the.view = "Hello World!"
```

### Example 2: Interactive Counter

```javascript
// Initialize counter
the.player.count = the.player.count || 0

// Display counter
the.view = "Count: " + the.player.count

// Listen for clicks
the.on.tap = function() {
    the.player.count++
    the.view = "Count: " + the.player.count
}
```

### Example 3: Complex Layout

```javascript
// Create container
var container = the.view({
    name: 'container',
    size: [[30, '~'], [20, '~']],
    fill: [0.9, 0.9, 0.9, 1]  // Light gray
})
container.into(the.view)

// Create title
var title = the.view({
    name: 'title',
    fill: 'My App',
    size: [2, 2, 1]  // 2x font size
})
title.into(container)

// Create button
var button = the.view({
    name: 'button',
    fill: 'Click Me',
    size: [[10, '~'], [3, '~']],
    fill: [0.2, 0.6, 1, 1]  // Blue
})
button.into(container)

// Button handler
button.on('tap', function() {
    console.log("Button clicked!")
})
```

### Example 4: Mouse Tracking

```javascript
var tracker = the.view({ name: 'tracker' })
tracker.into(the.view)

// Update on every frame
setInterval(function() {
    tracker.fill = "X: " + the.aim.x.toFixed(2) +
                   " Y: " + the.aim.y.toFixed(2)
}, 16)
```

### Example 5: Keyboard Input

```javascript
var position = {x: 0, y: 0}
var player = the.view({
    name: 'player',
    size: [[2, '~'], [2, '~']],
    fill: [1, 0, 0, 1]
})
player.into(the.view)

// Game loop
setInterval(function() {
    var speed = 0.1

    // WASD movement
    if (the.key.KeyW) position.y += speed
    if (the.key.KeyS) position.y -= speed
    if (the.key.KeyA) position.x -= speed
    if (the.key.KeyD) position.x += speed

    // Update position
    player.grab = [position.x, position.y, 0]
}, 16)
```

### Example 6: Animation

```javascript
var box = the.view({
    name: 'box',
    size: [[5, '~'], [5, '~']],
    fill: [1, 0.5, 0, 1]
})
box.into(the.view)

var time = 0
setInterval(function() {
    time += 0.01

    // Rotate
    box.turn = [time, 0, 0]

    // Scale with sin wave
    var scale = 1 + Math.sin(time * 2) * 0.3
    box.zoom = [scale, scale, 1]
}, 16)
```

---

## Best Practices

### Performance

1. **Minimize View Updates**
   - Only update views that change
   - Use requestAnimationFrame for smooth animations
   - Batch multiple changes together

2. **Efficient State Management**
   - Don't overuse `the.player` for temporary state
   - Keep localStorage data small and simple
   - Use local variables for frame-by-frame data

3. **Event Handlers**
   - Avoid creating handlers in loops
   - Reuse handlers when possible
   - Clean up unused handlers

### Security

1. **Trust the Sandbox**
   - User code runs isolated in Web Workers
   - Data cannot leak outside the sandbox
   - Parent windows cannot access worker data

2. **Data Validation**
   - Validate all user input
   - Sanitize localStorage data
   - Be careful with file uploads

### Code Organization

1. **Structure Your Code**
```javascript
// Configuration
var config = {
    speed: 0.1,
    maxScore: 100
}

// State
var state = {
    score: 0,
    level: 1
}

// Initialize
function init() {
    setupUI()
    loadData()
    startGame()
}

// Game loop
function update() {
    // Update logic
}

// Start
init()
setInterval(update, 16)
```

2. **Modular Components**
```javascript
// Button component
function createButton(label, onClick) {
    var btn = the.view({
        name: 'btn_' + Math.random(),
        fill: label,
        size: [[8, '~'], [2, '~']]
    })
    btn.on('tap', onClick)
    return btn
}

// Usage
var saveBtn = createButton('Save', function() {
    saveGame()
})
saveBtn.into(the.view)
```

### Debugging

1. **Console Logging**
```javascript
console.log("State:", state)
console.log("Aim:", the.aim.x, the.aim.y)
console.log("Keys:", Object.keys(the.key).filter(k => the.key[k]))
```

2. **Visual Debugging**
```javascript
// Show hitboxes
var debug = the.view({ name: 'debug' })
debug.into(the.view)
debug.fill = "Aim: " + the.aim.at + "\nKeys: " +
             Object.keys(the.key).filter(k => the.key[k]).join(', ')
```

### CSS Integration

1. **Use Framework Classes**
   - Prefer CSS classes over inline styles
   - Combine utility classes for complex layouts
   - Extend with custom CSS when needed

2. **Responsive Design**
```html
<div class="pad center">
    <div class="row">
        <div class="col center gap">Column 1</div>
        <div class="col center gap">Column 2</div>
    </div>
</div>
```

3. **Interactive Elements**
```html
<button class="act center gap pulse">
    Click Me
</button>
```

---

## Advanced Topics

### Custom Rendering

THE supports both CSS3 transforms and WebGL rendering:

```javascript
// CSS rendering (default)
// Automatic, works in all browsers

// WebGL rendering (experimental)
// Uncomment line 576 in sandbox.js to enable
// Provides better performance for 3D graphics
```

### Service Worker Integration

THE includes a Service Worker for:
- Offline functionality
- Asset caching
- Integrity verification
- Version upgrades

See `service.js` for implementation details.

### Browser Extension Mode

THE can run as a browser extension:

1. Load extension via chrome://extensions
2. Scripts are injected via `content.js`
3. Background script handles RPC messages
4. Full sandbox security maintained

### Physics and WebGL

THE includes WebGL plugin support for advanced graphics:

```javascript
// Load WebGL plugin
// See plug/webgl.js for implementation

// Access in examples
// See examples/physics/ for demos
```

---

## Troubleshooting

### CSP Issues

If you encounter Content Security Policy errors:

1. Comment out line 19 in `enclave.js`:
```javascript
// (sr.csp = document.querySelector('meta')).content = ...
```

2. Refresh the page several times

### Worker Errors

If Web Workers fail to spawn:

1. Check browser console for errors
2. Ensure you're running from localhost or HTTPS
3. Verify the script has `class="SecureRender"`

### View Not Updating

If `the.view` doesn't update:

1. Check for JavaScript errors
2. Verify view structure is correct
3. Ensure unique element names
4. Try refreshing the page

### Storage Issues

If `the.player` doesn't persist:

1. Check localStorage is enabled
2. Verify you're on the same origin
3. Check browser privacy settings
4. Clear localStorage and retry

---

## Resources

- **Examples**: See `/examples` folder for working demos
- **Source Code**: Core implementation in `sandbox.js`
- **CSS System**: Full styling reference in `layout.css`
- **Tests**: See `/test` folder for advanced usage

## Contributing

When adding features to THE:

1. Maintain security isolation
2. Follow existing code style
3. Add examples for new features
4. Update documentation
5. Test in multiple browsers

---

## License

See repository LICENSE file for details.

---

**Built with THE Framework** - The anti-monopoly game engine for the web.
