# Frontend Selectors & Development Tools

## Quick Commands

### CSS Selectors

```css
/* Basic Selectors */
#id-selector {
}
.class-selector {
}
element {
}
* {
} /* Universal selector */

/* Combinators */
.parent > .direct-child {
}
.ancestor .descendant {
}
.element + .adjacent-sibling {
}
.element ~ .general-sibling {
}

/* Attribute Selectors */
[attribute] {
}
[attribute="value"] {
}
[attribute^="starts-with"] {
}
[attribute$="ends-with"] {
}
[attribute*="contains"] {
}
[attribute~="word"] {
}
```

### Advanced CSS Selectors

```css
/* Pseudo-classes */
:hover,
:focus,
:active {
}
:first-child,
:last-child,
:nth-child(n) {
}
:nth-of-type(odd),
:nth-of-type(even) {
}
:not(.class-name) {
}
:has(.child-selector) {
} /* CSS4 - limited support */

/* Pseudo-elements */
::before,
::after {
}
::first-line,
::first-letter {
}
::placeholder {
}
::selection {
}
```

### jQuery Selectors

```javascript
// Basic jQuery patterns
$("#id"); // ID selector
$(".class"); // Class selector
$("element"); // Element selector
$("[attribute]"); // Attribute selector

// Advanced jQuery selectors
$(":first"); // First element
$(":last"); // Last element
$(":even"); // Even-indexed elements
$(":odd"); // Odd-indexed elements
$(":eq(index)"); // Element at specific index
$(":gt(index)"); // Elements after index
$(":lt(index)"); // Elements before index

// Form selectors
$(":input"); // All input elements
$(":text"); // Text inputs
$(":checkbox"); // Checkboxes
$(":selected"); // Selected options
$(":checked"); // Checked inputs
$(":disabled"); // Disabled elements
$(":enabled"); // Enabled elements
```

## Common Patterns

### Modern CSS Techniques

```css
/* Container Queries */
@container (min-width: 300px) {
  .card {
    grid-template-columns: 1fr 1fr;
  }
}

/* CSS Grid & Flexbox */
.grid-container {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
  gap: 1rem;
}

.flex-container {
  display: flex;
  justify-content: space-between;
  align-items: center;
  gap: 1rem;
}

/* Custom Properties (CSS Variables) */
:root {
  --primary-color: #007bff;
  --font-size-base: 1rem;
  --spacing-unit: 0.5rem;
}

.component {
  color: var(--primary-color);
  font-size: calc(var(--font-size-base) * 1.25);
  padding: calc(var(--spacing-unit) * 2);
}
```

### JavaScript DOM Manipulation

```javascript
// Modern DOM selection
const element = document.querySelector("#id");
const elements = document.querySelectorAll(".class");

// Event handling
element.addEventListener("click", (event) => {
  event.preventDefault();
  // Handle click
});

// Dynamic content
element.innerHTML = "<strong>Updated content</strong>";
element.textContent = "Safe text content";
element.classList.add("new-class");
element.classList.remove("old-class");
element.classList.toggle("toggle-class");

// Data attributes
element.dataset.userId = "123";
const userId = element.dataset.userId;
```

## Personal Gotchas

### CSS Specificity

```css
/* Specificity order (low to high) */
/* Universal: 0 */
/* Element: 1 */
/* Class: 10 */
/* ID: 100 */
/* Inline: 1000 */
/* !important: 10000 */

/* Avoid !important - use specific selectors instead */
.component.component {
} /* Higher specificity */
.page .component {
} /* Contextual specificity */
```

### jQuery Performance

```javascript
// Cache selectors to avoid repeated DOM queries
const $navItems = $(".nav-item"); // Cache this
$navItems.addClass("active"); // Reuse cached selection

// Chain operations when possible
$element.addClass("highlight").fadeIn(300).delay(1000).fadeOut(300);

// Use event delegation for dynamic content
$(document).on("click", ".dynamic-button", function () {
  // Handle clicks on dynamically added buttons
});
```

### Modern JavaScript vs jQuery

```javascript
// jQuery
$(".element").hide();
$(".element").show();
$(".element").toggle();

// Vanilla JavaScript
element.style.display = "none";
element.style.display = "block";
element.style.display = element.style.display === "none" ? "block" : "none";

// jQuery
$(".element").addClass("class");

// Vanilla JavaScript
element.classList.add("class");
```

## Performance Notes

### CSS Optimization

- Use class selectors over complex descendant selectors
- Avoid universal selectors in complex combinations
- Minimize CSS specificity conflicts
- Use CSS containment for layout performance

### Selector Performance Hierarchy

1. **ID selectors**: Fastest (`#id`)
2. **Class selectors**: Fast (`.class`)
3. **Element selectors**: Moderate (`div`)
4. **Attribute selectors**: Slower (`[attr="value"]`)
5. **Pseudo-selectors**: Slowest (`:nth-child()`)

### JavaScript Performance

```javascript
// Batch DOM updates
const fragment = document.createDocumentFragment();
items.forEach((item) => {
  const div = document.createElement("div");
  div.textContent = item.text;
  fragment.appendChild(div);
});
container.appendChild(fragment); // Single DOM update

// Use requestAnimationFrame for animations
function animate() {
  element.style.left = position + "px";
  if (position < target) {
    requestAnimationFrame(animate);
  }
}
```

## Architecture Considerations

### Component-Based CSS

```css
/* BEM methodology */
.card {
} /* Block */
.card__header {
} /* Element */
.card__header--large {
} /* Modifier */

/* CSS Modules approach */
.Card_container_1a2b3c {
}
.Card_header_4d5e6f {
}

/* Utility-first approach (Tailwind-style) */
.flex {
  display: flex;
}
.justify-center {
  justify-content: center;
}
.p-4 {
  padding: 1rem;
}
```

### Modern Framework Integration

```javascript
// React-style event handling
const handleClick = (event) => {
  event.preventDefault();
  setActiveState(!activeState);
};

// Vue.js-style reactive updates
const data = reactive({
  items: [],
  loading: false,
});

// Angular-style template binding
// <div [class.active]="isActive" (click)="handleClick($event)">
```

## Context Links

- [CSS Selectors Reference](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_Selectors)
- [jQuery Selectors Documentation](https://api.jquery.com/category/selectors/)
- [Modern CSS Features](https://web.dev/learn/css/)
- [DOM Manipulation Performance](https://developers.google.com/web/fundamentals/performance/rendering/)
