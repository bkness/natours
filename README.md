# Natours — Nature Tour Landing Page

A fully responsive landing page built as a deep-dive into advanced CSS and Sass architecture. Every visual effect — the animated hero, flipping tour cards, skewed testimonials, and pure-CSS hamburger navigation — is implemented without a single line of JavaScript.

**Live Demo:** _coming soon_

![Full page overview](assets/images/overview.png)

---

## Screenshots

| Hero | About |
|------|-------|
| ![Hero section](assets/images/hero.png) | ![About section](assets/images/about.png) |

| Features | Tours |
|----------|-------|
| ![Features section](assets/images/features.png) | ![Tours section](assets/images/tours.png) |

| Tours (card flip) | Testimonials |
|-------------------|--------------|
| ![Tours hover state](assets/images/tours-hover.png) | ![Testimonials](assets/images/testimonials.png) |

| Booking | Navigation |
|---------|-----------|
| ![Booking form](assets/images/booking.png) | ![Pure CSS navigation](assets/images/navigation.png) |

---

## Tech Stack

- **HTML5** — semantic markup
- **Sass (Dart Sass)** — compiled via `sass --watch`
- No JavaScript. No frameworks. No dependencies beyond the compiler.

---

## Sass Architecture

The project follows the **7-1 pattern** — seven partial directories, one entry point.

```
sass/
├── abstracts/
│   ├── _variables.scss    # Design tokens: colors, spacing, grid, typography
│   ├── _mixins.scss       # Reusable patterns: clearfix, centerAbs, respond()
│   └── _functions.scss    # Custom Sass functions
├── base/
│   ├── _base.scss         # Reset, box-sizing, fluid root font-size
│   ├── _animations.scss   # @keyframes definitions
│   ├── _typography.scss   # Heading and text styles
│   └── _utilities.scss    # Helper classes
├── components/
│   ├── _button.scss       # Animated CTA + text buttons
│   ├── _card.scss         # 3D flip tour cards
│   ├── _composition.scss  # Overlapping photo collage
│   ├── _feature-box.scss  # Icon + feature rows
│   ├── _story.scss        # Skewed testimonial cards
│   ├── _form.scss         # Floating label booking form
│   ├── _bg-video.scss     # Background video wrapper
│   └── _popup.scss        # Full-screen popup overlay
├── layout/
│   ├── _grid.scss         # Custom float grid with calc() columns
│   ├── _header.scss       # Hero with clip-path diagonal cut
│   ├── _navigation.scss   # Pure CSS checkbox-hack full-screen nav
│   └── _footer.scss       # Footer layout
├── pages/
│   └── _home.scss         # Page-level section styles
└── main.scss              # Single @import manifest
```

---

## Sass Features

### Variables — centralized design tokens

All colors, spacing, and sizing live in `_variables.scss`. Modifying the palette or grid requires changing a single file.

```scss
$color-primary:       #55c57a;
$color-primary-light: #7ed56f;
$color-primary-dark:  #28b485;

$grid-width:            114rem;
$gutter-horizontal:     6rem;
$default-font-size:     1.6rem;
$default-border-radius: 3px;
```

### Responsive Design Mixin with `@content`

The `respond()` mixin in `_mixins.scss` centralizes every media query. Breakpoints are written in `em` units (not `px`) for consistent scaling across browsers and user zoom settings.

```scss
@mixin respond($breakpoint) {
  @if $breakpoint == phone {
    @media (max-width: 37.5em) { @content; }   // 600px
  }
  @if $breakpoint == tab-port {
    @media (max-width: 56.25em) { @content; }  // 900px
  }
  @if $breakpoint == tab-land {
    @media (max-width: 75em) { @content; }     // 1200px
  }
  @if $breakpoint == big-desktop {
    @media (min-width: 112.5em) { @content; }  // 1800px
  }
}
```

The `@content` directive passes a block of styles into the mixin at the call site, keeping responsive rules co-located with the component they modify:

```scss
html {
  font-size: 62.5%; // 1rem = 10px

  @include respond(tab-land)    { font-size: 56.25%; } // 1rem = 9px
  @include respond(tab-port)    { font-size: 50%; }    // 1rem = 8px
  @include respond(big-desktop) { font-size: 75%; }    // 1rem = 12px
}
```

### BEM + Parent Selector (`&`)

Components use BEM naming enforced through Sass nesting. The `&` parent selector generates BEM modifier and element classes without repeating the block name, and composes compound selectors for interactive states.

```scss
.card {
  &__side {
    &--front { background-color: $color-white; }
    &--back  { transform: rotateY(180deg);

      &-1 { background-image: linear-gradient(to right bottom, $color-secondary-light, $color-secondary-dark); }
      &-2 { background-image: linear-gradient(to right bottom, $color-primary-light, $color-primary-dark); }
      &-3 { background-image: linear-gradient(to right bottom, $color-tertiary-light, $color-tertiary-dark); }
    }
  }

  // Compound selector: targets a child only when the parent is hovered
  &:hover &__side--front { transform: rotateY(-180deg); }
  &:hover &__side--back  { transform: rotateY(0); }
}
```

The same pattern drives the pure-CSS navigation. The checkbox state propagates to sibling elements through the general sibling combinator (`~`), toggling the full-screen overlay and hamburger-to-X icon transform entirely in CSS:

```scss
.navigation {
  &__checkbox:checked ~ &__background { transform: scale(80); }
  &__checkbox:checked ~ &__nav        { opacity: 1; width: 100%; }

  &__checkbox:checked + &__button &__icon         { background-color: transparent; }
  &__checkbox:checked + &__button &__icon::before { top: 0; transform: rotate(135deg); }
  &__checkbox:checked + &__button &__icon::after  { top: 0; transform: rotate(-135deg); }
}
```

### Custom Float Grid with `calc()` and Sass Interpolation

The grid is hand-rolled using floats and `calc()`. Sass variables are interpolated into `calc()` expressions using `#{}` to break out of the string context, allowing dynamic column width calculations:

```scss
.col-1-of-2 {
  width: calc((100% - #{$gutter-horizontal}) / 2);
}
.col-1-of-3 {
  width: calc((100% - 2 * #{$gutter-horizontal}) / 3);
}
.col-2-of-3 {
  width: calc(2 * ((100% - 2 * #{$gutter-horizontal}) / 3) + #{$gutter-horizontal});
}
```

The attribute selector `[class^="col-"]` targets all column classes generically via a prefix match, applying shared float and gutter behavior without duplicating rules per variant.

### `clearfix` and `centerAbs` Utility Mixins

```scss
@mixin clearfix {
  &::after {
    content: "";
    display: table;
    clear: both;
  }
}

@mixin centerAbs {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  border-radius: $default-border-radius;
}
```

`clearfix` uses the `&` parent selector to attach the clearing pseudo-element directly to the calling class. `centerAbs` encapsulates the classic absolute-center pattern and is reused wherever elements need centering inside a positioned parent — the navigation list, the card CTA, and the popup.

### `::before` / `::after` Pseudo-Elements

The button ripple effect is built entirely with `::after`. The pseudo-element matches the button's size and position, then scales out and fades on hover — no extra markup required.

```scss
.btn {
  &::after {
    content: "";
    display: inline-block;
    height: 100%;
    width: 100%;
    border-radius: 10rem;
    position: absolute;
    top: 0;
    left: 0;
    z-index: -1;
    transition: all 0.4s;
  }

  &:hover::after {
    transform: scaleX(1.4) scaleY(1.6);
    opacity: 0;
  }
}
```

The hamburger icon is a single `<span>` — its top and bottom bars are `::before` and `::after` pseudo-elements, repositioned and rotated into an X when the checkbox is checked.

### `clip-path` and Diagonal Section Breaks

The hero uses `clip-path: polygon()` to cut a diagonal bottom edge, creating the geometric layered layout without images or SVG.

```scss
.header {
  clip-path: polygon(0 0, 100% 0, 100% 75%, 0 100%);

  @include respond(phone) {
    clip-path: polygon(0 0, 100% 0, 100% 85%, 0 100%);
  }
}
```

Card images use the same technique for an angled bottom edge. At tablet breakpoints, the card back switches from a 3D flip to a clipped stacked reveal — the `clip-path` adapts instead of removing the effect.

### 3D Card Flip — `perspective` and `backface-visibility`

Tour cards flip on hover using CSS 3D transforms. `perspective` is set on the parent container to define the depth of the 3D space. `backface-visibility: hidden` prevents the reverse face from bleeding through during rotation.

```scss
.card {
  perspective: 150rem;
  position: relative;

  &__side {
    backface-visibility: hidden;
    transition: all 0.8s ease;
  }

  &__side--back          { transform: rotateY(180deg); }
  &:hover &__side--front { transform: rotateY(-180deg); }
  &:hover &__side--back  { transform: rotateY(0); }
}
```

### `shape-outside` — CSS Float Text Wrapping

The testimonials use `shape-outside: circle()` to make text flow around the circular avatar naturally, paired with `clip-path: circle()` for the visual crop.

```scss
.story__shape {
  float: left;
  shape-outside: circle(50% at 50% 50%);
  clip-path:     circle(50% at 50% 50%);
  transform: translateX(-3rem) skewX(12deg);
}
```

The entire card is skewed with `transform: skewX(-12deg)`, and the inner content is counter-skewed with `skewX(12deg)` to keep text upright — creating the parallelogram silhouette without distorting readability.

### Floating Label Form — `:placeholder-shown`

The booking form uses the adjacent sibling combinator (`+`) and `:placeholder-shown` to hide the label when the field is empty, then animate it into view as the user types.

```scss
&__input:placeholder-shown + &__label {
  opacity: 0;
  visibility: hidden;
  transform: translateY(-4rem);
}
```

When the user types and the placeholder disappears, the label transitions back into position — no JavaScript state management needed.

### Pure CSS Custom Radio Buttons

The native radio inputs are hidden and replaced with a styled `::after` pseudo-element on a sibling span. The `:checked` state drives visibility through the general sibling combinator:

```scss
&__radio-input:checked ~ &__radio-label &__radio-button::after {
  opacity: 1;
}
```

---

## Getting Started

```bash
npm install
npm run compile:sass
```

Open `index.html` directly in a browser. The Sass watcher recompiles on every save.

---

## Contact

Brandon — [DevBrandon@icloud.com](mailto:DevBrandon@icloud.com)
