# Frontend CSS/Styling Guidelines

## Table of Contents
1. [Semantic Markup and Page Structure](#semantics-and-page-structure)
2. [Naming Conventions](#naming-conventions)
3. [Hover and Transitions](#hover-and-transitions)
4. [Layout](#layout)
5. [Magic Numbers](#magic-numbers)
6. [Media](#media)
7. [Responsive Approach](#responsive-approach)
8. [Handling SVG/icons](#handling-svg)
9. [CSS Appoaches](#css-approaches)


<h2 id="#semantics-and-page-structure">Semantic Markup and Page Structure</h2>

Proper semantic markup is the foundation of accessible and maintainable pages. Follow these rules when you work on the HTML markup of the page:

- Use appropriate semantic tags (e.g., *ul*, *ol*, *li*, *button*, etc. ).
- Divide the page into regions (W3C landmarks: *main*, *header*, *footer*, *nav*, *section*).
- Markup must be valid. Run your HTML through the official [W3C validator](validator.w3.org).
- Make sure that the page has a **title**.
- Make sure that the page has only one **h1** heading (mandatory).
- Make sure that the page has a logical heading structure (*h1* -> *h2* -> *h3*, etc.).
- Explicitly connect labels to inputs using **for** and **id**, group related form elements with **fieldset** and **legend**.
- Make sure *html* element has a **lang** attribute (e.g., *lang="en"*).
- Make sure that interactive elements (buttons, links, inputs) are not nested inside other interactive elements. 

<h2 id="naming-conventions">Naming</h2>

### _CSS_
Use [BEM](https://ru.bem.info/methodology/quick-start/) methodology if there are no *CSS modules* or *CSS-in-JS* on the project.

```
.block {
    $this: &;

    position: relative;
    width: 100vw;

    &__element-bg {
        position: absolute;
        top: 0;
        left: 0
        width: 100px;
        height: 100px;
        background-color: $color-green;
        
        &--right {
            left: auto;
            right: 0;
        }
        
        #{$this}--big & {
            width: 300px;
            height: 300px;
        }
    }
}
```

Don't overload BEM elements.
If an element starts to contain two or more semantic attachments, it means that you most likely need to move such an element into a separate block.
Do not forget that each block must live in its own separate file.

❌ BAD:
```navigation.scss
.navigation {
    &__list {
        ...
    }
    
    &__item {
        ....
    }
    
    &__item-list {
        ...
    }
    
    &__item-list-icon {
        ...
    }
}
```

👍 GOOD:
```
navigation.scss
.navigation {
    &__list {
        ...
    }
    
    &__item {
        ....
    }
    
    &__link {
        ...
    }
}

navigation-list.scss
.navigation-list {
    ...
    
    &__link {
        ...
    }
    
    &__icon {
        ...
    }
}
```



### _CSS-selectors_
Don't use heavy selectors unless you really need to.
Don't use pure tags as part of a selector unless you really need to.
If you need to use a heavy selector to solve a problem, you might need to simplify something.

❌ BAD:
```
.block {
    $this: &;
    
    &__element {
        color: $color-white;
        
        .section .section--dark & {
            color: $color-black;
        }
    }
}
```

👍 GOOD:
```
.block {
    $this: &;
    
    &__element {
        color: $color-white;
    }
    
    #{$this}--dark & {
        color: $color-black;
    }
}
```

Don't use **!important** directive unless you absolutely need to.

<h2 id="hover-and-transitions">Hover and Transitions</h2>

### Active state

Make sure to add styles for the active state to interactive elements, such as buttons and links. If the designer didn't provide the necessary details, use your sense of beauty to indicate the active state by for example changing the color of a button or scaling an icon. And of course, ping the designer, let them take a look at your perspective and suggest their own.

All in all, any clickable element must have an active state so that the user is aware that it can be interacted with. </br>
*cursor property: pointer* is also required for all clickable elements.

### Transitions
Don't forget to add at least a simple transition for the active state. It will make things look much better than just an instant jerky switch.

```
.link {
    color: $color-black;
    transition: color 0.3s linear;
    
    @include hover-focus {
        color: $color-red;
    }
}
```


❌ BAD:
```
.element {
    transition: all 0.5s ease;
}
```
👍 GOOD:

```
.element {
    transition: background-color 0.3s linear, color 0.3s linear, 
}
```
 

## Layout

### Box-sizing
Use *box-sizing: border-box* to simplify the calculation of elements dimensions 
```
*,
*::before,
*::after {
    box-sizing: border-box;
}
```

### Modal and PageScroll
To manage page scroll behavior when modals or overlays are active, here's the following blocking strategy:

- **is-blocked** is used to prevent scrolling on desktop by hiding overflow and compensate for scrollbar width to prevent layout shift.

- **is-blocked-touch**: is used for fixed positioning for touch devices to maintain scroll position and prevent page movement behind modals.

```
.html {
    position: relative;
    overflow-x: hidden;
    overflow-y: scroll;
    height: 100%;
    font-size: 100%;
    -webkit-overflow-scrolling: touch;

    &.is-blocked {
        overflow: hidden;
        
        .compensate-scroll {
            padding-right: var(--scrollbar);
        }
    }

    &.is-blocked-touch {
        position: fixed;
        overflow-y: scroll;
        width: 100%;
        height: auto;
    }
}
```

### Container
To horizontally center the content
```
.container {
    max-width: $container-max-width;
    margin-right: auto;
    margin-left: auto;
    
    &--small {
        max-width: $container-max-width-small;
    }
    
    &--middle {
        max-width: $container-max-width-middle;
    }    
}
```

### Margins and paddings
You should differentiate between the logic of margins and the logic of paddings. 
For example, we have a list of cards in a row - it's worth wrapping each card into the element called  **"block__item"** that will get the outer spacings (margins), whereas the card itself will get the inner spacings (paddings).
 
```
.block {
    &__list {
        display: flex;
    }
  
    &__item {    
        & + & {
            margin-left: 10px;
        }
    } 
  
    &__card {
        padding: 10px;
        background-color: $color-blue;
    }
  
    &__link {
        font-size: 12px;
        color: $color-yellow;        
    }
}
```

### Component spacings

In case you are using **components approach** on the project (and **especially, if the components are managed in CMS**), make sure you take an effort to discuss the logic of vertical spacings between components together with the designer and come up with consistent rules that you will follow during the development. 
Otherwise, you are guaranteed to get bugs that the spacings do not match the design because content managers are free to come up with any combination of components order.

Possible style rules may look like this:

```
.components {
    $this: &;

    &__item {
        #{$this} > & ~ & {
            margin-top: $default-mobile-vertical-gap-between-components;

            @include tablet {
                margin-top: $default-desktop-vertical-gap-between-components;
            }

            &:last-child {
                margin-bottom: $default-mobile-vertical-gap-between-components;

                @include tablet {
                    margin-top: $default-desktop-vertical-gap-between-components;
                }

                .components & {
                    margin-bottom: 0;
                }
            }

            &--big:last-child {
                margin-bottom: 0;
            }
        }
    }
}
```

<h2 id="magic-numbers">Magic Numbers</h2>

Your code should not contain magic numbers, i.e. figures plucked from the air.

### CSS Magic

All values that are reused in different parts of the project, especially systematically, should be stored in *variables.scss*.

### Magic z-index

If you need to put an element on top of the entire application, create a scss variable for it and put it in the *variables.scss* file. No need to set extremely large values, use values no more than a thousand in increments of tens or hundreds. 

```
$layer-alert-bar-container: 80;
$layer-prompt: 90;
$layer-header: 100;
$layer-element-spinner: 100;
$layer-cookie-bar: 200;
$layer-overlay: 300;
$layer-menu: 500;
$layer-modal: 700;
$layer-page-spinner: 1000;
```

To use z-index locally, set the value of the parent (block) to zero and you safely redefine the values of the elements. However, stay within the values from -10 to 10. 
```
.block {
    position: relative;
    z-index: 0;
    
    &__bg {
        position: absolute;
        z-index: -1;
    }
    
    &__play {
        position: absolute;
        z-index: 1;
    }
}
```

If you cannot do without magic numbers, add a comment describing why you did it that way.

<h2 id="media">Media</h2>

### Image Compression
Before you add an image to the project's assets or directly to the page through CMS, make sure to compress it without quality losses using any online tool, e.g.[tinypng](https://tinypng.com/), [tinyjpg](https://tinyjpg.com/). Proper image compression reduces page load time and improves Core Web Vitals scores.

### Lazy Loading 
Loading all images upfront is a bad practice, because it wastes bandwidth and slows down initial page load.
```
<img 
  src="image.jpg"
  loading="lazy"
  alt="alt text"
>
```
However, don't lazy load the following:
1. Hero/first-viewport images
2. Logos (small file size)
3. Critical UI elements
4. Images likely to be printed

### Image alt text 
Alt text for informative images should convey meaning and give the context, it shouldn't be too generic or redundant. 

❌ BAD (generic):
```
<img src="chart.jpg" alt="Chart">
```

❌ BAD (redundant):
```
<img src="button-submit.jpg" alt="Submit button">
```

👍 GOOD (gives context):
```
<img src="icon-search.svg"  alt="Search products">
```

👍 GOOD (complex image with description):
```
<img src="chart.jpg" alt="Company organizational structure">
```

Decorative images don't need an alt text, because they are purely visual, e.g. visual dividers, decorative icons with text labels, background patterns.

❌ BAD (skipping alt entirely is an accessibility violation):
```
<img src="divider.png">
```

❌ BAD (meaningless text):
```
<img src="divider.png" alt="divider">
```

👍 GOOD (empty alt with role):
```
<img src="divider.png" alt="">
```

### No layout shift on load
Layout shift occurs when visible elements move unexpectedly, causing a frustrating user experience.

Specify *width* and *height*, or *aspect-ratio* in HTML:
```
<img 
  src="product.jpg"
  alt="Product image"
  width="400"
  height="300"
  style="aspect-ratio: 400 / 300;"
>
```

Provide *sizes* attribute for responsive images:
```
<img 
  srcset="small.jpg 400w, large.jpg 800w"
  sizes="(max-width: 600px) 100vw, 50vw"
  src="small.jpg"
  alt="..."
  width="800"
  height="600"
>
```

CSS to prevent layout shift:
1. Reserve space with *aspect-ratio*
```
.image-container {
  position: relative;
  width: 100%;
  padding-bottom: 75%; // 4:3 aspect ratio (3/4 = 0.75)
  
  img {
    position: absolute;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    object-fit: cover;
  }
}
```

2. Use modern *aspect-ratio* property
```
.image-wrapper {
  aspect-ratio: 16 / 9; // width / height
  
  img {
    width: 100%;
    height: 100%;
    object-fit: cover;
  }
}
```
You can test CLS in DevTools with network throttling and monitor Core Web Vitals in Lighthouse.

<h2 id="responsive-approach">Responsive Approach</h2>

### Viewports

We support all viewports from 375 to 4k screens.
But since there is rarely designs for 4k, you need to make sure that everything looks good on them yourself. 

### Mobile-first approach
Create a mobile version first, then go on to the desktop.

```
.nav {
    &__list {
        display: flex;
        align-items: center;
        
        @include desktop {
            justify-content: space-between;
        }
    }
    
    &__item {
        padding: 12px;
        font-size: 16px;
        
        @include desktop {
            padding: 20px;
        }
    }
}
```

### Tablet designs
If there is no separate designs for tablets, use your sense of beauty to get a decent variation of the existing mobile and desktop designs.

### Breakpoints
We have a list of all possible breakpoints used on the project:

```
$mobile-width: 375px;
$tablet-width: 768px;
$tablet-xl-width: 1024px;
$desktop-width: 1200px;
$desktop-xl-width: 1920px;

@mixin mobile {
  @media (min-width: #{$mobile-width}) {
    @content;
  }
}

@mixin tablet {
  @media (min-width: #{$tablet-width}) {
    @content;
  }
}

@mixin tablet-xl {
  @media (min-width: #{$tablet-xl-width}) {
    @content;
  }
}

@mixin desktop {
  @media (min-width: #{$desktop-width}) {
    @content;
  }
}

@mixin desktop-xl {
  @media (min-width: #{$desktop-xl-width}) {
    @content;
  }
}
```

<h2 id="handling-svg">Handling SVG/icons</h2>

As a rule, we have external SVG files which are imported into the component. 
However, it is possible to inline critical SVG icons right in the HTML (e.g. company logo).

### Color Control Methods
If you need to change the icon color in the CSS, e.g. on hover, avoid hardcoding it in the SVG itself:

❌ BAD: Colors baked into SVG:
```
<svg width="24" height="24" viewBox="0 0 24 24">
  <path fill="#007AFF" .../> 
</svg
```
👍 GOOD (using currentColor):
```
<svg class="icon" width="24" height="24" viewBox="0 0 24 24">
  <path fill="currentColor" .../>
</svg>
``` 

Changing the SVG color in CSS:
```
.icon {
  &:hover {
    fill: $color-primary;
  }
}
```

If you are using monochrome icons and simple logos in UI, make sure (or ask the designer to fix it) to prepare SVG with only one path, avoiding multiple elements:

❌ BAD:
```
<svg width="24" height="24" viewBox="0 0 24 24">
  <circle cx="12" cy="12" r="8" fill="#007AFF"/>
  <rect x="8" y="8" width="8" height="8" fill="#FF3B30"/>
  <path d="M6 6L18 18" stroke="#000" stroke-width="2"/>
</svg>
```

👍 GOOD (single continuous path element for entire icon):
```
<svg width="24" height="24" viewBox="0 0 24 24">
  <path d="M12 2C6.48 2 2 6.48 2 12s4.48 10 10 10 10-4.48 10-10S17.52 2 12 2zm-2 15l-5-5 1.41-1.41L10 14.17l7.59-7.59L19 8l-9 9z"/>
</svg>
```

Multiple elements need individual targeting in CSS, it adds complexity:</br>
❌ BAD:
```
.multi-element-icon {
  circle { fill: var(--icon-bg); }
  line { stroke: var(--icon-line); }
  rect { fill: var(--icon-square); }
  
  &:hover {
    circle { fill: var(--icon-bg-hover); }
    line { stroke: var(--icon-line-hover); }
    ...
  }
}
```

Single path = single CSS property:</br>
👍 GOOD:
```
.single-path-icon {
  fill: var(--icon-color);
  
  &:hover {
    fill: var(--icon-color-hover);
  }
}
```

However, multiple paths are necessary for multi-color icons, and for icons with animated parts with different animation timing or different hover states. This will affect performance, because the browser will have to parse multiple DOM nodes and execute multiple paint operations, so use it when you have a clear reason.


<h2 id="#css-approaches">CSS Appoaches</h2>

### Variables

Define global design tokens (e.g., colors, z-index) as CSS variables in the *_variables.scss* to avoid duplication and make their use consistent across the project.

```
css
// text
$color-text-primary: rgb(32 31 31 / 100%);
$color-text-secondary: rgb(113 113 113 / 100%);

// z-index
$overlay: 90;
$mobile-nav: 100;
$cookie-banner: 200;
```

### Mixins 
Mixins help you reuse common style patterns across your project without duplicating code. 
Define mixins for frequently used patterns like breakpoints, typography styles, and visual effects.

```
@mixin text-primary {
  font-family: Inter, sans-serif;
  font-size: 16px;
  ...
}

@mixin underline {
  position: relative;
  text-decoration: none;

  &::after {
    content: "";
    position: absolute;
    left: 0;
    bottom: 0;
    width: 100%;
    height: 1px;
    background-color: $color-black;
    transform-origin: bottom left;
    transition: transform 0.25s ease-out;
  }
}
```

Use `@content` for flexible responsive mixins and interactive state mixins (like hover-focus):

```
@mixin hover-focus {
  &:hover,
  &:focus {
      @content;
  }
}

@mixin desktop {
  @media (min-width: #{$desktop-width}) {
    @content;
  }
}
```