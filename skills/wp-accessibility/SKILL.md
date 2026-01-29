---
name: wp-accessibility
description: WordPress child theme accessibility skill implementing WCAG 2.1 AA compliance patterns, screen reader support, keyboard navigation, focus management, and automated testing for WordPress themes, blocks, and plugin forms.
tools:
  - Read
  - Write
  - Edit
  - Bash
  - Grep
  - Glob
---

# WordPress Child Theme Accessibility (WCAG 2.1 AA)

This skill provides definitive patterns for making WordPress child themes fully WCAG 2.1 AA compliant. Every code example targets child theme implementation without modifying parent themes or core plugins.

---

## 1. WCAG 2.1 AA Mapped to WordPress Theme Structure

### Perceivable

| Success Criterion | WordPress Implementation |
|---|---|
| 1.1.1 Non-text Content | `the_post_thumbnail()` with alt via attachment meta; empty `alt=""` for decorative images |
| 1.2.1 Audio/Video | Embed captions via `[video]` shortcode `caption` attribute or block `tracks` |
| 1.3.1 Info and Relationships | Semantic HTML5 in template parts: `<header>`, `<nav>`, `<main>`, `<footer>` |
| 1.3.2 Meaningful Sequence | DOM order matches visual order; avoid CSS-only reordering of content |
| 1.4.1 Use of Color | Never rely on color alone for status; pair with icons or text |
| 1.4.3 Contrast Minimum | 4.5:1 normal text, 3:1 large text (18px+ regular or 14px+ bold) |
| 1.4.11 Non-text Contrast | 3:1 for UI components and graphical objects (borders, icons, focus rings) |

### Operable

| Success Criterion | WordPress Implementation |
|---|---|
| 2.1.1 Keyboard | All interactive elements reachable and operable via Tab/Enter/Space/Escape |
| 2.4.1 Bypass Blocks | Skip link as first focusable element in `header.php` |
| 2.4.2 Page Titled | `wp_title()` or `document_title_parts` filter for descriptive `<title>` |
| 2.4.3 Focus Order | Logical tab order following DOM; avoid positive `tabindex` values |
| 2.4.6 Headings and Labels | Heading hierarchy (h1 > h2 > h3) without skipping levels |
| 2.4.7 Focus Visible | Visible focus indicator on all interactive elements (2px+ outline) |

### Understandable

| Success Criterion | WordPress Implementation |
|---|---|
| 3.1.1 Language of Page | `language_attributes()` in `<html>` tag outputs `lang` attribute |
| 3.1.2 Language of Parts | `lang` attribute on inline foreign-language text |
| 3.2.1 On Focus | No context change on focus alone; menus open on click/Enter |
| 3.3.1 Error Identification | Form errors described in text, linked to field via `aria-describedby` |
| 3.3.2 Labels or Instructions | Every input has a visible `<label>` with matching `for`/`id` pair |

### Robust

| Success Criterion | WordPress Implementation |
|---|---|
| 4.1.1 Parsing | Valid HTML output; run `wp_kses_post()` on user content |
| 4.1.2 Name, Role, Value | ARIA attributes on custom widgets; native HTML elements preferred |
| 4.1.3 Status Messages | `aria-live="polite"` regions for AJAX responses and notifications |

---

## 2. WordPress-Specific Accessibility Patterns

### Skip Link (First Focusable Element)

The skip link must be the very first focusable element in the DOM, before the site header.

```php
<!-- child theme header.php -->
<body <?php body_class(); ?>>
<?php wp_body_open(); ?>
<a class="skip-link screen-reader-text" href="#primary-content">
  <?php esc_html_e( 'Skip to content', 'oshin_child' ); ?>
</a>
<header id="masthead" role="banner">
```

```css
/* child theme style.css */
.skip-link {
  position: absolute;
  top: -100%;
  left: 0;
  z-index: 100000;
  padding: 0.75rem 1.5rem;
  background: #000;
  color: #fff;
  font-size: 1rem;
  text-decoration: none;
}
.skip-link:focus {
  top: 0;
  outline: 3px solid #ffbf47;
  outline-offset: 0;
}
```

### Landmark Roles Mapped to Template Parts

```php
<!-- header.php -->
<header id="masthead" role="banner">
  <nav id="site-navigation" role="navigation" aria-label="<?php esc_attr_e( 'Primary Menu', 'oshin_child' ); ?>">
    <?php wp_nav_menu( array( 'theme_location' => 'primary', 'container' => false ) ); ?>
  </nav>
</header>

<!-- single.php / page.php -->
<main id="primary-content" role="main">
  <?php the_content(); ?>
</main>

<!-- sidebar.php -->
<aside id="secondary" role="complementary" aria-label="<?php esc_attr_e( 'Sidebar', 'oshin_child' ); ?>">
  <?php dynamic_sidebar( 'sidebar-1' ); ?>
</aside>

<!-- footer.php -->
<footer id="colophon" role="contentinfo">
```

### Widget Area Labeling

```php
// child theme functions.php
register_sidebar( array(
  'name'          => __( 'Footer Widgets', 'oshin_child' ),
  'id'            => 'footer-widgets',
  'before_widget' => '<section id="%1$s" class="widget %2$s" role="region" aria-label="%1$s">',
  'after_widget'  => '</section>',
  'before_title'  => '<h2 class="widget-title">',
  'after_title'   => '</h2>',
) );
```

---

## 3. Color Contrast

### CSS Custom Properties Meeting Contrast Requirements

```css
:root {
  /* 4.5:1+ on white backgrounds */
  --color-text-primary: #1a1a1a;       /* 16.15:1 on #fff */
  --color-text-secondary: #505050;     /* 7.08:1 on #fff  */
  --color-text-muted: #6d6d6d;         /* 4.83:1 on #fff  */
  /* Large text (3:1 minimum) */
  --color-text-large-accent: #767676;  /* 4.54:1 on #fff  */
  /* Interactive elements */
  --color-link: #0055a4;               /* 7.26:1 on #fff  */
  --color-link-hover: #003d75;         /* 10.5:1 on #fff  */
  --color-focus-ring: #005fcc;         /* 6.58:1 on #fff  */
  /* Backgrounds */
  --color-bg-primary: #ffffff;
  --color-bg-secondary: #f5f5f5;
}

body {
  color: var(--color-text-primary);
  background: var(--color-bg-primary);
}
a {
  color: var(--color-link);
}
a:hover, a:active {
  color: var(--color-link-hover);
}
```

### Verification Tools

Run from the command line during development:

```bash
# pa11y for page-level checks
npx pa11y https://zentratec.local --standard WCAG2AA --reporter cli

# axe-core via CLI
npx @axe-core/cli https://zentratec.local --tags wcag2a,wcag2aa

# Lighthouse accessibility audit
npx lighthouse https://zentratec.local --only-categories=accessibility --output=json
```

---

## 4. Form Accessibility

### Label Association Pattern

```html
<div class="form-group">
  <label for="user-email">
    <?php esc_html_e( 'Email Address', 'oshin_child' ); ?>
    <span class="required" aria-hidden="true">*</span>
  </label>
  <input
    type="email"
    id="user-email"
    name="email"
    required
    aria-required="true"
    aria-describedby="email-hint email-error"
  />
  <span id="email-hint" class="field-hint">
    <?php esc_html_e( 'We will never share your email.', 'oshin_child' ); ?>
  </span>
  <span id="email-error" class="field-error" role="alert" aria-live="assertive"></span>
</div>
```

### Fieldset and Legend for Grouped Controls

```html
<fieldset>
  <legend><?php esc_html_e( 'Preferred Contact Method', 'oshin_child' ); ?></legend>
  <label>
    <input type="radio" name="contact_method" value="email" /> Email
  </label>
  <label>
    <input type="radio" name="contact_method" value="phone" /> Phone
  </label>
</fieldset>
```

### Contact Form 7 Accessibility Fix

CF7 does not output labels by default. Override in child theme:

```php
// child theme functions.php
add_filter( 'wpcf7_form_elements', 'oshin_child_cf7_a11y_fix' );
function oshin_child_cf7_a11y_fix( $content ) {
  // Add aria-label to inputs missing associated labels
  $content = preg_replace(
    '/<input([^>]*?)class="wpcf7-form-control[^"]*"([^>]*?)placeholder="([^"]*?)"/',
    '<input$1class="wpcf7-form-control"$2placeholder="$3" aria-label="$3"',
    $content
  );
  return $content;
}
```

### Fluent Forms Error Focus Management

```javascript
// child theme assets/js/a11y-forms.js
document.addEventListener('fluentform_submission_failed', function(e) {
  const firstError = document.querySelector('.ff-el-is-error .ff-el-input input, .ff-el-is-error .ff-el-input textarea');
  if (firstError) {
    firstError.focus();
    firstError.setAttribute('aria-invalid', 'true');
  }
});
```

---

## 5. Navigation Accessibility

### Keyboard-Navigable Dropdown Menu (Disclosure Pattern)

The disclosure pattern is recommended over the menubar pattern for site navigation.

```php
<!-- child theme nav template -->
<nav aria-label="<?php esc_attr_e( 'Main Navigation', 'oshin_child' ); ?>">
  <ul class="nav-menu">
    <li class="menu-item-has-children">
      <a href="/services"><?php esc_html_e( 'Services', 'oshin_child' ); ?></a>
      <button
        class="submenu-toggle"
        aria-expanded="false"
        aria-controls="submenu-services"
        aria-label="<?php esc_attr_e( 'Services submenu', 'oshin_child' ); ?>"
      >
        <span class="icon-arrow" aria-hidden="true"></span>
      </button>
      <ul id="submenu-services" class="sub-menu" role="list">
        <li><a href="/consulting">Consulting</a></li>
        <li><a href="/engineering">Engineering</a></li>
      </ul>
    </li>
  </ul>
</nav>
```

```javascript
// child theme assets/js/a11y-nav.js
document.querySelectorAll('.submenu-toggle').forEach(function(button) {
  button.addEventListener('click', function() {
    var expanded = this.getAttribute('aria-expanded') === 'true';
    this.setAttribute('aria-expanded', String(!expanded));
    var submenu = document.getElementById(this.getAttribute('aria-controls'));
    submenu.hidden = expanded;
  });

  button.addEventListener('keydown', function(e) {
    if (e.key === 'Escape') {
      this.setAttribute('aria-expanded', 'false');
      document.getElementById(this.getAttribute('aria-controls')).hidden = true;
      this.focus();
    }
  });
});
```

### Current Page Indicator

```php
// child theme functions.php
add_filter( 'nav_menu_css_class', 'oshin_child_aria_current', 10, 2 );
function oshin_child_aria_current( $classes, $item ) {
  return $classes;
}

add_filter( 'nav_menu_link_attributes', 'oshin_child_aria_current_attr', 10, 2 );
function oshin_child_aria_current_attr( $atts, $item ) {
  if ( $item->current ) {
    $atts['aria-current'] = 'page';
  }
  return $atts;
}
```

### Breadcrumb Navigation

```php
<nav aria-label="<?php esc_attr_e( 'Breadcrumb', 'oshin_child' ); ?>">
  <ol class="breadcrumb-list">
    <li><a href="<?php echo esc_url( home_url() ); ?>"><?php esc_html_e( 'Home', 'oshin_child' ); ?></a></li>
    <li><a href="<?php echo esc_url( get_post_type_archive_link( 'post' ) ); ?>"><?php esc_html_e( 'Blog', 'oshin_child' ); ?></a></li>
    <li aria-current="page"><?php the_title(); ?></li>
  </ol>
</nav>
```

---

## 6. Screen Reader Support

### WordPress Standard `.screen-reader-text` Class

```css
.screen-reader-text {
  border: 0;
  clip: rect(1px, 1px, 1px, 1px);
  clip-path: inset(50%);
  height: 1px;
  margin: -1px;
  overflow: hidden;
  padding: 0;
  position: absolute;
  width: 1px;
  word-wrap: normal !important;
}
.screen-reader-text:focus {
  background-color: #f1f1f1;
  clip: auto !important;
  clip-path: none;
  color: #21759b;
  display: block;
  font-size: 0.875rem;
  font-weight: 700;
  height: auto;
  left: 5px;
  line-height: normal;
  padding: 15px 23px 14px;
  text-decoration: none;
  top: 5px;
  width: auto;
  z-index: 100000;
}
```

### `aria-label` vs `aria-labelledby`

Use `aria-label` when no visible text exists. Use `aria-labelledby` when referencing existing visible text.

```html
<!-- aria-label: no visible text to reference -->
<button aria-label="<?php esc_attr_e( 'Close dialog', 'oshin_child' ); ?>">
  <svg aria-hidden="true"><!-- X icon --></svg>
</button>

<!-- aria-labelledby: visible heading exists -->
<section aria-labelledby="section-heading-about">
  <h2 id="section-heading-about"><?php esc_html_e( 'About Us', 'oshin_child' ); ?></h2>
</section>
```

### Live Regions for AJAX Content

```html
<!-- Status container in template -->
<div id="ajax-status" class="screen-reader-text" aria-live="polite" aria-atomic="true"></div>
```

```javascript
// After AJAX load completes
function announceToScreenReader(message) {
  var status = document.getElementById('ajax-status');
  status.textContent = '';
  requestAnimationFrame(function() {
    status.textContent = message;
  });
}

// Example: infinite scroll loaded
announceToScreenReader('6 new articles loaded. Now showing 18 articles total.');
```

---

## 7. Focus Management

### Modal / Dialog Focus Trapping

```javascript
// child theme assets/js/a11y-modal.js
function trapFocus(modal) {
  var focusable = modal.querySelectorAll(
    'a[href], button:not([disabled]), textarea, input:not([type="hidden"]), select, [tabindex]:not([tabindex="-1"])'
  );
  var first = focusable[0];
  var last = focusable[focusable.length - 1];

  modal.addEventListener('keydown', function(e) {
    if (e.key === 'Tab') {
      if (e.shiftKey && document.activeElement === first) {
        e.preventDefault();
        last.focus();
      } else if (!e.shiftKey && document.activeElement === last) {
        e.preventDefault();
        first.focus();
      }
    }
    if (e.key === 'Escape') {
      closeModal(modal);
    }
  });

  first.focus();
}

function openModal(modal, trigger) {
  modal.setAttribute('role', 'dialog');
  modal.setAttribute('aria-modal', 'true');
  modal.hidden = false;
  modal.dataset.triggerElement = trigger.id;
  document.body.style.overflow = 'hidden';
  trapFocus(modal);
}

function closeModal(modal) {
  modal.hidden = true;
  document.body.style.overflow = '';
  var trigger = document.getElementById(modal.dataset.triggerElement);
  if (trigger) trigger.focus();
}
```

### Visible Focus Indicators

```css
/* Base focus style for all interactive elements */
a:focus-visible,
button:focus-visible,
input:focus-visible,
select:focus-visible,
textarea:focus-visible,
[tabindex]:focus-visible {
  outline: 3px solid var(--color-focus-ring, #005fcc);
  outline-offset: 2px;
  border-radius: 2px;
}

/* Remove default and re-apply for browsers that support :focus-visible */
a:focus:not(:focus-visible),
button:focus:not(:focus-visible) {
  outline: none;
}
```

---

## 8. Images and Media

### Post Thumbnail with Enforced Alt Text

```php
// child theme functions.php
function oshin_child_accessible_thumbnail( $post_id = null ) {
  if ( ! has_post_thumbnail( $post_id ) ) return;

  $thumb_id  = get_post_thumbnail_id( $post_id );
  $alt_text  = get_post_meta( $thumb_id, '_wp_attachment_image_alt', true );

  if ( empty( $alt_text ) ) {
    $alt_text = get_the_title( $post_id );
  }

  echo wp_get_attachment_image( $thumb_id, 'large', false, array(
    'alt'     => esc_attr( $alt_text ),
    'loading' => 'lazy',
  ) );
}
```

### Figure / Figcaption Pattern

```php
<?php
$caption = wp_get_attachment_caption( $image_id );
$alt     = get_post_meta( $image_id, '_wp_attachment_image_alt', true );
?>
<figure>
  <?php echo wp_get_attachment_image( $image_id, 'large', false, array( 'alt' => esc_attr( $alt ) ) ); ?>
  <?php if ( $caption ) : ?>
    <figcaption><?php echo esc_html( $caption ); ?></figcaption>
  <?php endif; ?>
</figure>
```

### Decorative Images

Images that are purely decorative must have an empty `alt` attribute and be hidden from assistive tech.

```html
<img src="decorative-divider.svg" alt="" role="presentation" />
```

### Accessible Video Embed

```html
<figure>
  <video controls preload="metadata">
    <source src="intro.mp4" type="video/mp4" />
    <track kind="captions" src="intro-captions-en.vtt" srclang="en" label="English" default />
    <track kind="descriptions" src="intro-descriptions-en.vtt" srclang="en" label="English Audio Descriptions" />
  </video>
  <figcaption>Introduction to our engineering services. <a href="intro-transcript.html">Read transcript</a></figcaption>
</figure>
```

---

## 9. Block Editor Accessibility

### Accessible Custom Block (register_block_type)

```php
// child theme blocks/testimonial/render.php
function oshin_child_render_testimonial( $attributes ) {
  $name  = esc_html( $attributes['authorName'] ?? '' );
  $quote = esc_html( $attributes['quote'] ?? '' );
  $id    = 'testimonial-' . wp_unique_id();

  return sprintf(
    '<blockquote class="wp-block-testimonial" aria-labelledby="%s">
      <p>%s</p>
      <footer>
        <cite id="%s">%s</cite>
      </footer>
    </blockquote>',
    esc_attr( $id ),
    $quote,
    esc_attr( $id ),
    $name
  );
}

register_block_type( 'oshin-child/testimonial', array(
  'render_callback' => 'oshin_child_render_testimonial',
  'attributes'      => array(
    'authorName' => array( 'type' => 'string', 'default' => '' ),
    'quote'      => array( 'type' => 'string', 'default' => '' ),
  ),
) );
```

### InnerBlocks Accessibility Wrapper

```javascript
// block edit.js (Gutenberg editor component)
import { InnerBlocks, useBlockProps } from '@wordpress/block-editor';

export default function Edit() {
  const blockProps = useBlockProps({
    role: 'region',
    'aria-label': 'Content section',
  });

  return (
    <section {...blockProps}>
      <InnerBlocks
        allowedBlocks={['core/paragraph', 'core/heading', 'core/image']}
        template={[['core/heading', { level: 2, placeholder: 'Section title' }]]}
      />
    </section>
  );
}
```

### Dynamic Block Server Render Accessibility

```php
// Ensure dynamic blocks output valid, accessible HTML
function oshin_child_render_cta_block( $attributes, $content ) {
  $heading = esc_html( $attributes['heading'] ?? 'Learn More' );
  $url     = esc_url( $attributes['url'] ?? '#' );
  $desc    = esc_attr( $attributes['description'] ?? '' );

  $aria = $desc ? sprintf( 'aria-describedby="cta-desc-%s"', wp_unique_id() ) : '';
  $desc_el = $desc ? sprintf( '<p id="cta-desc-%s" class="screen-reader-text">%s</p>', wp_unique_id(), esc_html( $desc ) ) : '';

  return sprintf(
    '<div class="wp-block-cta" role="region" aria-label="%s">
      %s
      <a href="%s" class="cta-button" %s>%s</a>
    </div>',
    esc_attr( $heading ),
    $desc_el,
    $url,
    $aria,
    $heading
  );
}
```

---

## 10. Testing Checklist

### Automated Testing (Run in CI or Locally)

```bash
# 1. axe-core: catch common WCAG violations
npx @axe-core/cli https://zentratec.local --tags wcag2a,wcag2aa --exit

# 2. pa11y: WCAG 2.1 AA page-level audit
npx pa11y https://zentratec.local --standard WCAG2AA --reporter cli

# 3. Lighthouse: accessibility score (target 95+)
npx lighthouse https://zentratec.local \
  --only-categories=accessibility \
  --output=json --output-path=./a11y-report.json

# 4. HTML validation (catches ARIA misuse)
npx html-validate https://zentratec.local
```

### Keyboard-Only Testing (Manual)

1. Unplug mouse. Navigate entire page with Tab, Shift+Tab, Enter, Space, Escape, Arrow keys.
2. Verify: skip link works and moves focus to `#primary-content`.
3. Verify: every interactive element has a visible focus indicator.
4. Verify: dropdown menus open with Enter/Space, close with Escape, return focus to toggle.
5. Verify: modals trap focus inside and restore focus to trigger on close.
6. Verify: no keyboard traps exist (can always Tab away from any element).
7. Verify: form validation errors receive focus and are announced.

### Screen Reader Testing (Manual)

| Test | VoiceOver (macOS) | NVDA (Windows) |
|---|---|---|
| Page landmarks | Rotor > Landmarks | NVDA+F7 > Landmarks |
| Heading hierarchy | Rotor > Headings | NVDA+F7 > Headings |
| Image alt text | VO reads alt on focus | NVDA reads alt on focus |
| Form labels | VO reads label on input focus | NVDA reads label on focus |
| Live regions | VO announces polite updates | NVDA announces on change |
| Skip link | Tab once, VO reads "Skip to content" | Tab once, NVDA announces link |

### WCAG 2.1 AA Manual Review Checklist

- [ ] All images have appropriate alt text (descriptive or empty for decorative)
- [ ] Color contrast meets 4.5:1 (normal text) and 3:1 (large text, UI components)
- [ ] All form inputs have visible labels with correct `for`/`id` association
- [ ] Error messages are linked to inputs via `aria-describedby`
- [ ] Page has correct heading hierarchy (no skipped levels)
- [ ] Skip link is first focusable element and targets `<main>`
- [ ] All landmark regions have unique labels when duplicated
- [ ] Focus is never lost during page interactions (modal open/close, AJAX load)
- [ ] No content flashes more than three times per second
- [ ] `lang` attribute is set on `<html>` and on foreign-language passages
- [ ] `aria-current="page"` is set on the active navigation link
- [ ] All interactive elements are reachable and operable by keyboard alone
- [ ] Touch targets are at least 44x44 CSS pixels
- [ ] Content reflows without loss at 320px viewport width (400% zoom)
- [ ] Animations respect `prefers-reduced-motion` media query

### Reduced Motion Support

```css
@media (prefers-reduced-motion: reduce) {
  *, *::before, *::after {
    animation-duration: 0.01ms !important;
    animation-iteration-count: 1 !important;
    transition-duration: 0.01ms !important;
    scroll-behavior: auto !important;
  }
}
```
