# Introduction to Web Components 

This document is a quick introduction to the process of creating web components from
scratch. We are going to create three simple web components which demonstrate basic
concepts.

## Simple web component extending an HTMLElement

In this part we are going to talk about the basic process, which will help us to focus
on the essential topics like lifecycle and some basic integration. The example is
pretty straightforward.

```javascript
// example1.js
class HighlightText extends HTMLElement {
  constructor() {
    super();
  }
}

customElements.define('cc-highlight', HighlightText);
```

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Web Component Examples</title>
</head>
<body>
<p>
    Lorem ipsum dolor sit amet consectetur adipisicing elit.
    Corrupti dolores officia placeat recusandae.
    Dolores a quibusdam voluptas tempore neque recusandae in,
    <cc-highlight>quia delectus autem distinctio</cc-highlight>.
    Accusantium suscipit quam distinctio illum?
</p>

<script src="./example1.js"></script>
</body>
</html>
```

## Enhancing the Component

### Adding a Symbol to the Component

Let's enhance the component to manipulate the DOM and add a simple asterisk (`*`) symbol.

```javascript
// example1.js
class HighlightText extends HTMLElement {
  constructor() {
    super();
    this.textHighlightSymbol = null;
    this.textHighlightContainer = null;
  }

  connectedCallback() {
    this.textHighlightSymbol = document.createElement('span');
    this.textHighlightSymbol.textContent = ' *';
    this.appendChild(this.textHighlightSymbol);
  }
}

customElements.define('cc-highlight', HighlightText);
```

### Adding Event Listeners

We can add interactivity by introducing event listeners.

```javascript
class HighlightText extends HTMLElement {
  constructor() {
    super();
    this.textHighlightSymbol = null;
    this.textHighlightContainer = null;
  }

  connectedCallback() {
    this.textHighlightSymbol = document.createElement('span');
    this.textHighlightSymbol.textContent = ' *';
    this.textHighlightSymbol.addEventListener(
      'mouseenter',
      this.showHighlightedContainer.bind(this)
    );
    this.appendChild(this.textHighlightSymbol);
  }

  showHighlightedContainer() {
    this.textHighlightContainer = document.createElement('div');
    this.textHighlightContainer.textContent = this.innerText.slice(
      0,
      this.innerText.indexOf(this.textHighlightSymbol.textContent)
    );
    this.appendChild(this.textHighlightContainer);
  }
}

customElements.define('cc-highlight', HighlightText);
```

### Adding and Removing Elements Dynamically

Let's remove the element dynamically when the mouse leaves.

```javascript
class HighlightText extends HTMLElement {
  constructor() {
    super();
    this.textHighlightSymbol = null;
    this.textHighlightContainer = null;
  }

  connectedCallback() {
    this.textHighlightSymbol = document.createElement('span');
    this.textHighlightSymbol.textContent = ' *';
    this.textHighlightSymbol.addEventListener(
        'mouseenter',
        this.showHighlightedContainer.bind(this)
    );
    this.textHighlightSymbol.addEventListener(
        'mouseleave',
        this.hideHighlightedContainer.bind(this)
    );
    this.appendChild(this.textHighlightSymbol);
  }

  showHighlightedContainer() {
    this.textHighlightContainer = document.createElement('div');
    this.textHighlightContainer.textContent = this.innerText.slice(
      0,
      this.innerText.indexOf(this.textHighlightSymbol.textContent)
    );
    
    this.appendChild(this.textHighlightContainer);
  }

  hideHighlightedContainer() {
    this.removeChild(this.textHighlightContainer);
  }
}

customElements.define('cc-highlight', HighlightText);
```

### Using Attributes

You can specify attributes to customize the behavior of the component. For example, let's define a `symbol` attribute.

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Web Component Examples</title>
</head>
<body>
<p>
    Lorem ipsum dolor sit amet consectetur adipisicing elit.
    Corrupti dolores officia placeat recusandae.
    Dolores a quibusdam voluptas tempore neque recusandae in,
    <cc-highlight symbol="$">quia delectus autem distinctio</cc-highlight>.
    Accusantium suscipit quam distinctio illum?
</p>

<script src="./example1.js"></script>
</body>
</html>
```

```javascript
class HighlightText extends HTMLElement {
  constructor() {
    super();
    this.textHighlightSymbol = null;
    this.textHighlightContainer = null;
  }

  connectedCallback() {
    this.textHighlightSymbol = document.createElement('span');
    this.textHighlightSymbol.textContent = ' *';

    if (this.hasAttribute('symbol')) {
      this.textHighlightSymbol.textContent = this.getAttribute('symbol').trim();
    }

    this.textHighlightSymbol.addEventListener(
      'mouseenter',
      this.showHighlightedContainer.bind(this)
    );

    this.textHighlightSymbol.addEventListener(
      'mouseleave',
      this.hideHighlightedContainer.bind(this)
    );

    this.appendChild(this.textHighlightSymbol);
  }

  showHighlightedContainer() {
    this.textHighlightContainer = document.createElement('div');
    this.textHighlightContainer.textContent = this.innerText.slice(
      0,
      this.innerText.indexOf(this.textHighlightSymbol.textContent)
    );
    this.appendChild(this.textHighlightContainer);
  }

  hideHighlightedContainer() {
    this.removeChild(this.textHighlightContainer);
  }
}

customElements.define('cc-highlight', HighlightText);
```

### Listening to Attribute Changes

To listen to attribute changes, use the `attributeChangedCallback` method and specify observed attributes.

```javascript
class HighlightText extends HTMLElement {
  constructor() {
    super();
    this.textHighlightSymbol = null;
    this.textHighlightContainer = null;
  }

  connectedCallback() {
    this.textHighlightSymbol = document.createElement('span');
    this.textHighlightSymbol.textContent = ' *';

    if (this.hasAttribute('symbol')) {
      this.textHighlightSymbol.textContent = this.getAttribute('symbol').trim();
    }

    this.textHighlightSymbol.addEventListener(
      'mouseenter',
      this.showHighlightedContainer.bind(this)
    );

    this.textHighlightSymbol.addEventListener(
      'mouseleave',
      this.hideHighlightedContainer.bind(this)
    );

    this.appendChild(this.textHighlightSymbol);
  }

  attributeChangedCallback(name, oldValue, newValue) {
    console.log(`Attribute ${name} changed from ${oldValue} to ${newValue}`);
  }

  static get observedAttributes() {
    return ['symbol'];
  }

  showHighlightedContainer() {
    this.textHighlightContainer = document.createElement('div');
    this.textHighlightContainer.textContent = this.innerText.slice(
        0,
        this.innerText.indexOf(this.textHighlightSymbol.textContent)
    );
    this.appendChild(this.textHighlightContainer);
  }

  hideHighlightedContainer() {
    this.removeChild(this.textHighlightContainer);
  }
}

customElements.define('cc-highlight', HighlightText);
```

### Listening to Specific Attributes

To make your web component listen for specific attribute changes, you need to define the attributes to observe. 
Below is an example demonstrating how to achieve this.

```javascript
class HighlightText extends HTMLElement {
  constructor() {
    super();
    this.textHighlightSymbol = null;
    this.textHighlightContainer = null;
  }

  connectedCallback() {
    this.textHighlightSymbol = document.createElement('span');
    this.textHighlightSymbol.textContent = ' *';
    
    if (this.hasAttribute('symbol')) {
      this.textHighlightSymbol.textContent = ` ${this.getAttribute('symbol').trim()}`;
    }
    
    this.textHighlightSymbol.addEventListener(
      'mouseenter',
      this.showHighlightedContainer.bind(this)
    );

    this.textHighlightSymbol.addEventListener(
      'mouseleave',
      this.hideHighlightedContainer.bind(this)
    );
    
    this.appendChild(this.textHighlightSymbol);
  }

  attributeChangedCallback(name, oldValue, newValue) {
    console.log('name: ', name);
    console.log('oldValue: ', oldValue);
    console.log('newValue: ', newValue);
  }

  // Specify the attributes to observe
  static get observedAttributes() {
    return ['symbol'];
  }

  disconnectedCallback() {
    console.log('Component removed from the DOM');
  }

  showHighlightedContainer() {
    this.textHighlightContainer = document.createElement('div');
    this.textHighlightContainer.textContent = this.innerText.slice(
      0,
      this.innerText.indexOf(this.textHighlightSymbol.textContent)
    );

    this.appendChild(this.textHighlightContainer);
  }

  hideHighlightedContainer() {
    this.removeChild(this.textHighlightContainer);
  }
}

customElements.define('cc-highlight', HighlightText);
```




Once the event listeners are in place, we can further modify the HTML element dynamically. The following example demonstrates how to achieve this.

```javascript
class HighlightText extends HTMLElement {
  constructor() {
    super();
    this.textHighlightSymbol = null;
    this.textHighlightContainer = null;
  }

  connectedCallback() {
    this.textHighlightSymbol = document.createElement('span');
    this.textHighlightSymbol.textContent = ' *';


    if (this.hasAttribute('symbol')) {
      this.textHighlightSymbol.textContent = ` ${this.getAttribute('symbol').trim()}`;
    }


    this.textHighlightSymbol.addEventListener(
      'mouseenter',
      this.showHighlightedContainer.bind(this)
    );

    this.textHighlightSymbol.addEventListener(
      'mouseleave',
      this.hideHighlightedContainer.bind(this)
    );
    
    this.appendChild(this.textHighlightSymbol);
  }

  attributeChangedCallback(name, oldValue, newValue) {
    if (oldValue === newValue) {
      return;
    }
    
    if (name === 'symbol' && this.textHighlightSymbol) {
      this.textHighlightSymbol.textContent = ` ${this.getAttribute('symbol').trim()}`;
    }
  }
  
  static get observedAttributes() {
    return ['symbol'];
  }

  disconnectedCallback() {
    console.log('Component removed from the DOM');
  }

  showHighlightedContainer() {
    this.textHighlightContainer = document.createElement('div');
    this.textHighlightContainer.textContent = this.innerText.slice(
      0,
      this.innerText.indexOf(this.textHighlightSymbol.textContent)
    );

    this.appendChild(this.textHighlightContainer);
  }

  hideHighlightedContainer() {
    this.removeChild(this.textHighlightContainer);
  }
}

customElements.define('cc-highlight', HighlightText);
```

### Utilizing the Cleaning Callback to Remove Listeners

To ensure proper cleanup and avoid potential memory leaks, we can remove event listeners when the component is disconnected from the DOM. Here's an example:

```javascript
class HighlightText extends HTMLElement {
  constructor() {
    super();
    this.textHighlightSymbol = null;
    this.textHighlightContainer = null;
  }

  connectedCallback() {
    this.textHighlightSymbol = document.createElement('span');
    this.textHighlightSymbol.textContent = ' *';
    
    if (this.hasAttribute('symbol')) {
      this.textHighlightSymbol.textContent = ` ${this.getAttribute('symbol').trim()}`;
    }
    
    this.textHighlightSymbol.addEventListener(
      'mouseenter',
      this.showHighlightedContainer.bind(this)
    );

    this.textHighlightSymbol.addEventListener(
      'mouseleave',
      this.hideHighlightedContainer.bind(this)
    );
    
    this.appendChild(this.textHighlightSymbol);
  }

  attributeChangedCallback(name, oldValue, newValue) {
    if (oldValue === newValue) {
        return;
    }
    
    if (name === 'symbol' && this.textHighlightSymbol) {
      this.textHighlightSymbol.textContent = ` ${this.getAttribute('symbol').trim()}`;
    }
  }
  
  static get observedAttributes() {
      return ['symbol'];
  }

  disconnectedCallback() {
    this.textHighlightSymbol.removeEventListener(
        'mouseenter',
        this.showHighlightedContainer
    );

    this.textHighlightSymbol.removeEventListener(
        'mouseleave',
        this.hideHighlightedContainer
    );
  }

  showHighlightedContainer() {
    this.textHighlightContainer = document.createElement('div');
    this.textHighlightContainer.textContent = this.innerText.slice(
      0,
      this.innerText.indexOf(this.textHighlightSymbol.textContent)
    );

    this.appendChild(this.textHighlightContainer);
  }

  hideHighlightedContainer() {
    this.removeChild(this.textHighlightContainer);
  }
}


customElements.define('cc-highlight', HighlightText);
```

### Adding Styling to Web Components

We can enhance the user experience of a web component by adding some simple styling to its elements. Here's an example:

```javascript
class HighlightText extends HTMLElement {
  constructor() {
    super();
    this.textHighlightSymbol = null;
    this.textHighlightContainer = null;
  }

  connectedCallback() {
    this.textHighlightSymbol = document.createElement('span');
    this.textHighlightSymbol.textContent = ' *';
    
    if (this.hasAttribute('symbol')) {
        this.textHighlightSymbol.textContent = ` ${this.getAttribute('symbol').trim()}`;
    }
    
    this.textHighlightSymbol.addEventListener(
        'mouseenter',
        this.showHighlightedContainer.bind(this)
    );
    
    this.textHighlightSymbol.addEventListener(
        'mouseleave',
        this.hideHighlightedContainer.bind(this)
    );
    
    this.appendChild(this.textHighlightSymbol);
    this.style.position = 'relative';
  }

  attributeChangedCallback(name, oldValue, newValue) {
    if (oldValue === newValue) {
      return;
    }
    
    if (name === 'symbol' && this.textHighlightSymbol) {
      this.textHighlightSymbol.textContent = ` ${this.getAttribute('symbol').trim()}`;
    }
  }

  static get observedAttributes() {
    return ['symbol'];
  }

  disconnectedCallback() {
    this.textHighlightSymbol.removeEventListener(
      'mouseenter',
      this.showHighlightedContainer
    );
    this.textHighlightSymbol.removeEventListener(
      'mouseleave',
      this.hideHighlightedContainer
    );
  }

  showHighlightedContainer() {
    this.textHighlightContainer = document.createElement('div');
    this.textHighlightContainer.textContent = this.innerText.slice(
      0,
      this.innerText.indexOf(this.textHighlightSymbol.textContent)
    );

    this.appendChild(this.textHighlightContainer);
    
    this.textHighlightContainer.style.backgroundColor = 'black';
    this.textHighlightContainer.style.color = 'white';
    this.textHighlightContainer.style.position = 'absolute';
  }

  hideHighlightedContainer() {
    this.removeChild(this.textHighlightContainer);
  }
}

customElements.define('cc-highlight', HighlightText);
```

### Avoiding Global Style Interference in Web Components

While building web components as shown earlier, there are potential **downsides** to consider. One major issue is the **risk of global styles overriding styles** 
within the component. Consider this example:

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>Web Component Examples</title>
  <style>
      span {
          color: red;
      }
  </style>
</head>
<body>
<p>
  Lorem, ipsum dolor sit amet consectetur adipisicing elit. Corrupti dolores
  officia placeat recusandae. Dolores a quibusdam voluptas tempore neque
  recusandae in, <cc-highlight symbol="$">quia delectus autem distinctio</cc-highlight>.
  Accusantium suscipit quam distinctio illum?
</p>

<script src="./example1.js"></script>
</body>
</html>
```

#### Issue with Global Styles

In this example, the `span` tag has a global style (`color: red`), which might unintentionally affect the `span` 
element inside the `<cc-highlight>` web component.

#### Solution: Shadow DOM

To address this, we can isolate the content of the web component using Shadow DOM. Shadow DOM creates an encapsulated styling context, 
ensuring that styles defined outside the component do not interfere with the component's internal elements.

#### Key Shadow DOM Modes

- `mode: 'open'`: Allows external access and modification of the Shadow DOM through the shadowRoot property. 
- `mode: 'closed'`: Restricts external access to the Shadow DOM.

> **Best Practice**: Use `mode: 'open'` in most cases, as it provides flexibility while maintaining encapsulation.

```javascript
class HighlightText extends HTMLElement {
  constructor() {
    super();
    this.textHighlightSymbol = null;
    this.textHighlightContainer = null;

    // Attach Shadow DOM with mode 'open'
    this.attachShadow({ mode: 'open' });
  }

  connectedCallback() {
    this.textHighlightSymbol = document.createElement('span');
    this.textHighlightSymbol.textContent = ' *';

    if (this.hasAttribute('symbol')) {
      this.textHighlightSymbol.textContent = ` ${this.getAttribute('symbol').trim()}`;
    }

    this.textHighlightSymbol.addEventListener(
      'mouseenter',
      this.showHighlightedContainer.bind(this)
    );

    this.textHighlightSymbol.addEventListener(
      'mouseleave',
      this.hideHighlightedContainer.bind(this)
    );
    
    this.shadowRoot.appendChild(this.textHighlightSymbol);
    this.style.position = 'relative';
  }

  attributeChangedCallback(name, oldValue, newValue) {
    if (oldValue === newValue) {
        return;
    }

    if (name === 'symbol' && this.textHighlightSymbol) {
      this.textHighlightSymbol.textContent = ` ${this.getAttribute('symbol').trim()}`;
    }
  }

  static get observedAttributes() {
    return ['symbol'];
  }

  disconnectedCallback() {
    this.textHighlightSymbol.removeEventListener(
      'mouseenter',
      this.showHighlightedContainer
    );

    this.textHighlightSymbol.removeEventListener(
      'mouseleave',
      this.hideHighlightedContainer
    );
  }

  showHighlightedContainer() {
    this.textHighlightContainer = document.createElement('div');
    this.textHighlightContainer.textContent = this.innerText.slice(
      0,
      this.innerText.indexOf(this.textHighlightSymbol.textContent)
    );

    this.shadowRoot.appendChild(this.textHighlightContainer);
    
    this.textHighlightContainer.style.backgroundColor = 'black';
    this.textHighlightContainer.style.color = 'white';
    this.textHighlightContainer.style.position = 'absolute';
  }

  hideHighlightedContainer() {
    this.shadowRoot.removeChild(this.textHighlightContainer);
  }
}

customElements.define('cc-highlight', HighlightText);
```

#### Advantages of Using Shadow DOM:

1. **Encapsulation**: Ensures that styles and scripts within the Shadow DOM do not leak out and are not affected by global styles.
2. **Reusability**: Components are self-contained, making them easy to reuse across projects.
3. **Flexibility**: With mode: 'open', developers can interact with the Shadow DOM if needed.

### Resolving Issues with Shadow DOM Initialization

When you initialize the **Shadow DOM**, you might notice two issues:
1. Your inner text disappears.
2. The highlight symbol (`*`) also vanishes.

This happens because appending elements to the Shadow DOM isolates them from the light DOM. To fix this, we need to update the component to ensure that elements are properly rendered and the functionality works as intended.

```javascript
class HighlightText extends HTMLElement {
  constructor() {
    super();
    this.textHighlightSymbol = null;
    this.textHighlightContainer = null;
    
    this.attachShadow({ mode: 'open' });
  }

  connectedCallback() {
    this.textHighlightSymbol = document.createElement('span');
    this.textHighlightSymbol.textContent = ' *';

    if (this.hasAttribute('symbol')) {
        this.textHighlightSymbol.textContent = ` ${this.getAttribute('symbol').trim()}`;
    }
    
    this.textHighlightSymbol.addEventListener(
      'mouseenter',
      this.showHighlightedContainer.bind(this)
    );
    this.textHighlightSymbol.addEventListener(
      'mouseleave',
      this.hideHighlightedContainer.bind(this)
    );
    
    this.shadowRoot.appendChild(this.textHighlightSymbol);
    this.style.position = 'relative';
  }

  attributeChangedCallback(name, oldValue, newValue) {
    if (oldValue === newValue) {
      return;
    }

    if (name === 'symbol' && this.textHighlightSymbol) {
      this.textHighlightSymbol.textContent = ` ${this.getAttribute('symbol').trim()}`;
    }
  }

  static get observedAttributes() {
    return ['symbol'];
  }

  disconnectedCallback() {
    this.textHighlightSymbol.removeEventListener(
      'mouseenter',
      this.showHighlightedContainer
    );
    this.textHighlightSymbol.removeEventListener(
      'mouseleave',
      this.hideHighlightedContainer
    );
  }

  showHighlightedContainer() {
    this.textHighlightContainer = document.createElement('div');
    this.textHighlightContainer.textContent = this.innerText.slice(
      0,
      this.innerText.indexOf(this.textHighlightSymbol.textContent)
    );

    this.shadowRoot.appendChild(this.textHighlightContainer);

    this.textHighlightContainer.style.backgroundColor = 'black';
    this.textHighlightContainer.style.color = 'white';
    this.textHighlightContainer.style.position = 'absolute';
  }

  hideHighlightedContainer() {
    this.shadowRoot.removeChild(this.textHighlightContainer);
  }
}

customElements.define('cc-highlight', HighlightText);
```

#### What's Fixed?

- **Highlight Symbol**: The `*` symbol is appended to the Shadow DOM, ensuring it is displayed.
- **Highlight Container**: The `showHighlightedContainer` and `hideHighlightedContainer` functions now target the Shadow DOM for appending and removing the highlight container.

This will at least returns the highlighted attribute, but as you can see, the functionality doesn't work properly. 

### Adding a Template for the Component

To improve the maintainability and readability of the component, we can use an HTML `<template>` to define reusable structures. 
This helps separate the layout logic from JavaScript and ensures consistent rendering.

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Web Component Examples</title>
    <style>
        span {
            color: red;
        }
    </style>
</head>
<body>

<template id="highlight-template">
    <span id="symbol-placeholder"></span>
</template>

<p>
  Lorem, ipsum dolor sit amet consectetur adipisicing elit. Corrupti dolores
  officia placeat recusandae. Dolores a quibusdam voluptas tempore neque
  recusandae in,
  <cc-highlight symbol="$">quia delectus autem distinctio</cc-highlight>.
  Accusantium suscipit quam distinctio illum?
</p>

<script src="./example1.js"></script>
</body>
</html>
```

```javascript
class HighlightText extends HTMLElement {
  constructor() {
    super();
    
    this.attachShadow({ mode: 'open' });
    
    const template = document.querySelector('#highlight-template');
    this.shadowRoot.appendChild(template.content.cloneNode(true));
    
    this.textHighlightSymbol = null;
    this.textHighlightContainer = null;
  }

  connectedCallback() {
    this.textHighlightSymbol = this.shadowRoot.querySelector('#symbol-placeholder');
    this.textHighlightSymbol.textContent = ' *';
    
    if (this.hasAttribute('symbol')) {
        this.textHighlightSymbol.textContent = ` ${this.getAttribute('symbol').trim()}`;
    }
    
    this.textHighlightSymbol.addEventListener(
        'mouseenter',
        this.showHighlightedContainer.bind(this)
    );
    this.textHighlightSymbol.addEventListener(
        'mouseleave',
        this.hideHighlightedContainer.bind(this)
    );
    
    this.style.position = 'relative';
  }

  attributeChangedCallback(name, oldValue, newValue) {
    if (oldValue === newValue) {
      return;
    }

    if (name === 'symbol' && this.textHighlightSymbol) {
      this.textHighlightSymbol.textContent = ` ${this.getAttribute('symbol').trim()}`;
    }
  }

  static get observedAttributes() {
    return ['symbol'];
  }

  disconnectedCallback() {
    this.textHighlightSymbol.removeEventListener(
      'mouseenter',
      this.showHighlightedContainer
    );
    this.textHighlightSymbol.removeEventListener(
      'mouseleave',
      this.hideHighlightedContainer
    );
  }

  showHighlightedContainer() {
    this.textHighlightContainer = document.createElement('div');
    this.textHighlightContainer.textContent = this.innerText.slice(
      0,
      this.innerText.indexOf(this.textHighlightSymbol.textContent)
    );

    this.shadowRoot.appendChild(this.textHighlightContainer);
    this.textHighlightContainer.style.backgroundColor = 'black';
    this.textHighlightContainer.style.color = 'white';
    this.textHighlightContainer.style.position = 'absolute';
  }

  hideHighlightedContainer() {
    this.shadowRoot.removeChild(this.textHighlightContainer);
  }
}

customElements.define('cc-highlight', HighlightText);
```

#### Simplifying the Query for the `span` Element

Since the template contains only a single `<span>` element, we can query it directly by its tag name. This simplifies our logic while retaining functionality.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>Web Component Examples</title>
  <style>
      span {
          color: red;
      }
  </style>
</head>
<body>

<template id="highlight-template">
    <span></span>
</template>

<p>
  Lorem, ipsum dolor sit amet consectetur adipisicing elit. Corrupti dolores
  officia placeat recusandae. Dolores a quibusdam voluptas tempore neque
  recusandae in,
  <cc-highlight symbol="$">quia delectus autem distinctio</cc-highlight>.
  Accusantium suscipit quam distinctio illum?
</p>

<script src="./example1.js"></script>
</body>
</html>
```

```javascript
class HighlightText extends HTMLElement {
  constructor() {
    super();
    
    this.attachShadow({ mode: 'open' });
    
    const template = document.querySelector('#highlight-template');
    this.shadowRoot.appendChild(template.content.cloneNode(true));
    
    this.textHighlightSymbol = null;
    this.textHighlightContainer = null;
  }

  connectedCallback() {
    this.textHighlightSymbol = this.shadowRoot.querySelector('span');
    this.textHighlightSymbol.textContent = ' *';
    
    if (this.hasAttribute('symbol')) {
        this.textHighlightSymbol.textContent = ` ${this.getAttribute('symbol').trim()}`;
    }
    
    this.textHighlightSymbol.addEventListener(
        'mouseenter',
        this.showHighlightedContainer.bind(this)
    );
    this.textHighlightSymbol.addEventListener(
        'mouseleave',
        this.hideHighlightedContainer.bind(this)
    );
    
    this.style.position = 'relative';
  }

  attributeChangedCallback(name, oldValue, newValue) {
    if (oldValue === newValue) {
        return;
    }
    
    if (name === 'symbol' && this.textHighlightSymbol) {
        this.textHighlightSymbol.textContent = ` ${this.getAttribute('symbol').trim()}`;
    }
  }

  static get observedAttributes() {
    return ['symbol'];
  }

  disconnectedCallback() {
    this.textHighlightSymbol.removeEventListener(
        'mouseenter',
        this.showHighlightedContainer
    );
    this.textHighlightSymbol.removeEventListener(
        'mouseleave',
        this.hideHighlightedContainer
    );
  }

  showHighlightedContainer() {
    this.textHighlightContainer = document.createElement('div');
    this.textHighlightContainer.textContent = this.innerText.slice(
      0,
      this.innerText.indexOf(this.textHighlightSymbol.textContent)
    );
    
    this.shadowRoot.appendChild(this.textHighlightContainer);
    this.textHighlightContainer.style.backgroundColor = 'black';
    this.textHighlightContainer.style.color = 'white';
    this.textHighlightContainer.style.position = 'absolute';
  }

  hideHighlightedContainer() {
    this.shadowRoot.removeChild(this.textHighlightContainer);
  }
}

customElements.define('cc-highlight', HighlightText);
```

#### Introducing `slots` 

We are still missing the content which is put between opening and closing tag of the
custom component. To render this, we need to use concept called `slot`. There can be a
default value.

```html
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1.0" />
<title>Web Component examples</title>
<style>
span {
color: red;
}
</style>
</head>
<body>
  <template id="highlight-template">
  <slot>Default content</slot><span></span>
  </template>

  <p>
    Lorem, ipsum dolor sit amet consectetur adipisicing elit. Corrupti dolores
    officia placeat recusandae. Dolores a quibusdam voluptas tempore neque
    recusandae in,
    <cc-highlight symbol="$">quia delectus autem distinctio</cc-highlight>.
    Accusantium suscipit quam distinctio illum?
  </p>
  
  <cc-highlight></cc-highlight>
  
  <script src="./example1.js"></script>
</body>
</html>
```

```javascript
class HighlightText extends HTMLElement {
  constructor() {
    super();
    
    this.textHighlightSymbol;
    this.textHighlightContainer;
    this.attachShadow({ mode: 'open' });
    const template = document.querySelector('#highlight-template');
    
    this.shadowRoot.appendChild(template.content.cloneNode(true));
  }

  connectedCallback() {
    this.textHighlightSymbol = this.shadowRoot.querySelector('span');
    this.textHighlightSymbol.textContent = ' *';
    
    if (this.hasAttribute('symbol')) {
      this.textHighlightSymbol.textContent = ` ${this.getAttribute('symbol').trim()}`;
    }
    
    this.textHighlightSymbol.addEventListener(
      'mouseenter',
      this.showHighlightedContainer.bind(this)
    );
  
    this.textHighlightSymbol.addEventListener(
      'mouseleave',
      this.hideHighlightedContainer.bind(this)
    );
    
    this.shadowRoot.appendChild(this.textHighlightSymbol);
    this.style.position = 'relative';
  }

  attributeChangedCallback(name, oldValue, newValue) {
    if (oldValue === newValue) {
      return;
    }

    if (name === 'symbol' && this .textHighlightSymbol) {
      this.textHighlightSymbol.textContent = ` ${this.getAttribute('symbol').trim()}`;
    }
  }

  static get observedAttributes() {
    return ['symbol'];
  }

  disconnectedCallback() {
    this.textHighlightSymbol.removeEventListener(
      'mouseenter',
      this.showHighlightedContainer
    );

    this.textHighlightSymbol.removeEventListener(
      'mouseleave',
      this.hideHighlightedContainer
    );
  }

  showHighlightedContainer() {
    this.textHighlightContainer = document.createElement('div');
    const slot = this.shadowRoot.querySelector('slot');
    const text = slot.assignedNodes()[0]
      ? slot.assignedNodes()[0].textContent
      : slot.textContent;
  
    this.textHighlightContainer.textContent = text;
    this.shadowRoot.appendChild( this .textHighlightContainer);
    this.textHighlightContainer.style.backgroundColor = 'black';
    this.textHighlightContainer.style.color = 'white';
    this.textHighlightContainer.style.position = 'absolute';
  }

  hideHighlightedContainer() {
    this.shadowRoot.removeChild( this .textHighlightContainer);
  }
}

customElements.define('cc-highlight', HighlightText);
```

#### Removing the `<template> from the light DOM`

We re-implemented the initial functionality, but there are obviously a few issues
caused by the fact the template is inserted as a part of light DOM. We can improve
this.

```html
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1.0" />
<title>Web Component examples</title>
<style>
  span {
    color: red;
  }
</style>
</head>
<body>
  <p>
    Lorem, ipsum dolor sit amet consectetur adipisicing elit. Corrupti dolores
    officia placeat recusandae. Dolores a quibusdam voluptas tempore neque
    recusandae in, <cc-highlight symbol="$">quia delectus autem distinctio</cc-highlight>.
    Accusantium suscipit quam distinctio illum?
  </p>
  
  <cc-highlight></cc-highlight>
  
  <script src="./example1.js"></script>
</body>
</html>
```

```javascript
class HighlightText extends HTMLElement {
  constructor () {
    super();
    
    this.textHighlightSymbol;
    this.textHighlightContainer;
    this.attachShadow({ mode: 'open' });
    
    this.shadowRoot.innerHTML = `
      <style></style>
      <slot>Default Content</slot>
      <span></span>
    `;
  }

  connectedCallback() {
    this.textHighlightSymbol = this.shadowRoot.querySelector('span');
    this.textHighlightSymbol.textContent = ' *';
    
    
    if (this .hasAttribute('symbol')) {
      this.textHighlightSymbol.textContent = ` ${ this.getAttribute('symbol').trim()}`;
    }
    
    this.textHighlightSymbol.addEventListener(
      'mouseenter',
      this.showHighlightedContainer.bind(this)
    );
    
    this.textHighlightSymbol.addEventListener(
      'mouseleave',
      this.hideHighlightedContainer.bind(this)
    );
    
    this.shadowRoot.appendChild(this.textHighlightSymbol);
    this.style.position = 'relative';
  }

  attributeChangedCallback(name, oldValue, newValue) {
    if (oldValue === newValue) {
      return;
    }
  
    if (name === 'symbol' && this.textHighlightSymbol) {
      this.textHighlightSymbol.textContent = ` ${ this .getAttribute('symbol').trim()}`;
    }
  }

  static get observedAttributes() {
    return ['symbol'];
  }

  disconnectedCallback() {
    this.textHighlightSymbol.removeEventListener(
      'mouseenter',
      this .showHighlightedContainer
    );
    
    this.textHighlightSymbol.removeEventListener(
      'mouseleave',
      this.hideHighlightedContainer
    );
  }

  showHighlightedContainer() {
    this.textHighlightContainer = document.createElement('div');
    const slot = this.shadowRoot.querySelector('slot');
    const text = slot.assignedNodes()[ 0 ]
      ? slot.assignedNodes()[ 0 ].textContent
      : slot.textContent;
    
    this.textHighlightContainer.textContent = text;
    this.shadowRoot.appendChild( this .textHighlightContainer);
    this.textHighlightContainer.style.backgroundColor = 'black';
    this.textHighlightContainer.style.color = 'white';
    this.textHighlightContainer.style.position = 'absolute';
  }

  hideHighlightedContainer() {
    this.shadowRoot.removeChild( this .textHighlightContainer);
  }
}

customElements.define('cc-highlight', HighlightText);
```
#### Updating the styling

We can also update styling. There are a few specific selectors which are associated with web components:

- `::slotted | ::slotted(.className)` - styling of slotted selector. Can contain one child, but deeper hierarchy is not allowed.
- `:host | :host(.className)` - selector for the whole web component.
- `:host-context(tag|class)` - selector for a web component if it's present within a specific hierarchy

You can also define a variable in the style and refer to them in the Shadow DOM styles. `--color-primary: #a4e6fc`; 
and then refer to them in the style with a fallback `var(--color-primary, #ccc)`. The best place where to store the
variable is either `:root` or `html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1.0" />
<title>Web Component examples</title>
<style>
  :root {
  --color-primary: gray;
  }
</style>
</head>
<body>
  <p>
    Lorem, ipsum dolor sit amet consectetur adipisicing elit. Corrupti dolores
    officia placeat recusandae. Dolores a quibusdam voluptas tempore neque
    recusandae in, <cc-highlight symbol="$"><span class="test">quia delectus autem distinctio</span></cc-highlight>. 
    Accusantium suscipit quam distinctio illum?
  </p>
  
  <cc-highlight></cc-highlight>
  
  <script src="./example1.js"></script>
</body>
</html>
```

```javascript
class HighlightText extends HTMLElement {
  constructor () {
    super();
    
    this.textHighlightSymbol;
    this.textHighlightContainer;
    this.attachShadow({ mode: 'open' });
    this.shadowRoot.innerHTML = `
      <style>
      ::slotted(.test) {
      color: var(--color-primary, orange);
      }
      
      :host {
      position: relative;
      }
      
      :host-context(p) {
      text-decoration: underline;
      }
      </style>
      <slot>Default Content</slot>
      <span></span>
    `;
  }

  connectedCallback() {
    this.textHighlightSymbol = this.shadowRoot.querySelector('span');
    this.textHighlightSymbol.textContent = ' *';
    
    if (this.hasAttribute('symbol')) {
      this.textHighlightSymbol.textContent = ` ${ this.getAttribute('symbol').trim()}`;
    }
    
    this.textHighlightSymbol.addEventListener(
      'mouseenter',
      this.showHighlightedContainer.bind(this)
    );
    
    this.textHighlightSymbol.addEventListener(
      'mouseleave',
      this.hideHighlightedContainer.bind(this)
    );
    
    this.shadowRoot.appendChild( this.textHighlightSymbol);
  }

  attributeChangedCallback(name, oldValue, newValue) {
    if (oldValue === newValue) {
      return ;
    }

    if (name === 'symbol' && this .textHighlightSymbol) {
      this.textHighlightSymbol.textContent = ` ${this.getAttribute('symbol').trim()}`;
    }
  }

  static get observedAttributes() {
    return ['symbol'];
  }

  disconnectedCallback() {
    this.textHighlightSymbol.removeEventListener(
      'mouseenter',
      this.showHighlightedContainer
    );

    this.textHighlightSymbol.removeEventListener(
      'mouseleave',
      this.hideHighlightedContainer
    );
  }

  showHighlightedContainer() {
    this.textHighlightContainer = document.createElement('div');
    const slot = this .shadowRoot.querySelector('slot');
    const text = slot.assignedNodes()[0]
      ? slot.assignedNodes()[0].textContent
      : slot.textContent;
    
    this.textHighlightContainer.textContent = text;
    this.shadowRoot.appendChild(this.textHighlightContainer);
    this.textHighlightContainer.style.backgroundColor = 'black';
    this.textHighlightContainer.style.color = 'white';
    this.textHighlightContainer.style.position = 'absolute';
  }

  hideHighlightedContainer() {
    this.shadowRoot.removeChild( this .textHighlightContainer);
  }
}

customElements.define('cc-highlight', HighlightText);
```

#### Improving the styling

We can also remove some styles from the functions and moved it to the template.

```javascript
class HighlightText extends HTMLElement {
    
  constructor () {
    super();
    
    this.textHighlightSymbol;
    this.textHighlightContainer;
    this.attachShadow({ mode: 'open' });
    this.shadowRoot.innerHTML = `
      <style>
      ::slotted(.test) {
      color: var(--color-primary, orange);
      }
      
      :host {
      position: relative;
      }
      
      :host-context(p) {
      text-decoration: underline;
      }
      
      #text-highlight-container {
      display: none;
      }
      
      #text-highlight-container.visible {
      display: block;
      background-color: black;
      color: white;
      position: absolute;
      }
      </style>
      <slot>Default Content</slot>
      <span></span>
      <div id="text-highlight-container"></div>
    `;
  }

  connectedCallback() {
    this.textHighlightSymbol = this.shadowRoot.querySelector('span');
    this.textHighlightSymbol.textContent = ' *';
    if (this.hasAttribute('symbol')) {
      this .textHighlightSymbol.textContent = ` ${ this .getAttribute(
      'symbol'
      ).trim()}`;
    }
  
    this.textHighlightSymbol.addEventListener(
      'mouseenter',
      this.showHighlightedContainer.bind(this)
    );
    
    this.textHighlightSymbol.addEventListener(
      'mouseleave',
      this.hideHighlightedContainer.bind(this)
    );
    
    this.shadowRoot.appendChild(this.textHighlightSymbol);
  }

  attributeChangedCallback(name, oldValue, newValue) {
    if (oldValue === newValue) {
        return;
    }
    
    if (name === 'symbol' && this .textHighlightSymbol) {
      this.textHighlightSymbol.textContent = ` ${this.getAttribute('symbol').trim()}`;
    }
  }

  static get observedAttributes() {
    return ['symbol'];
  }
  
  disconnectedCallback() {
    this.textHighlightSymbol.removeEventListener(
      'mouseenter',
      this.showHighlightedContainer
    );
    
    this.textHighlightSymbol.removeEventListener(
      'mouseleave',
      this.hideHighlightedContainer
    );
  }

  showHighlightedContainer() {
    this.textHighlightContainer = this.shadowRoot.querySelector('#text-highlight-container');

    const slot = this.shadowRoot.querySelector('slot');
    const text = slot.assignedNodes()[0]
      ? slot.assignedNodes()[0].textContent
      : slot.textContent;

    this.textHighlightContainer.textContent = text;
    this.textHighlightContainer.classList.add('visible');
  }

  hideHighlightedContainer() {
    this.textHighlightContainer.classList.remove('visible');
  }
}

customElements.define('cc-highlight', HighlightText);
```

## Simple web component extending a specific element (`HTMLAnchorElement`)

```html
<!DOCTYPE html>
<html lang="en">
<head>

<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1.0" />
<title>Web Component examples</title>
<style>
  :root {
  --color-primary: gray;
  }
</style>
</head>
<body>
  <p>
    Lorem, ipsum dolor sit amet consectetur adipisicing elit. Corrupti dolores
    officia placeat recusandae. Dolores a quibusdam voluptas tempore neque
    recusandae in, <cc-highlight symbol="$">
      <span class="test">quia delectus autem distinctio</span>
      </cc-highlight>. Accusantium suscipit quam distinctio illum?
    </p>
    <cc-highlight></cc-highlight>
    <p><a is="cc-confirm-link" href="http://www.google.com">Google</a></p>
    <script src="./example1.js"></script>
    <script src="./example2.js"></script>
</body>
</html>
```

```javascript
class ConfirmLink extends HTMLAnchorElement {
  connectedCallback() {
    this.addEventListener('click', (event) => {
      if (!confirm('Do you really want to leave?')) {
        event.preventDefault();
      }
    });
  }
}

customElements.define('cc-confirm-link', ConfirmLink, { extends: 'a' });
```

## Conclusion

Thanks for following my notes. Any feedback appreciated (radek.tomasek@gmail.com).