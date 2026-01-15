# Accessibility

## Accessible Website vs Version for Visually Impaired

Creating a single, accessible main website based on modern WCAG princples is of high importaince and we are expected to consider accessibility requirements on all our projects wherever possible.

We discarded the idea of creating separate versions for the visually impaired (VVI). VVI provides adjustable fonts and colors but excludes users with motor or other disabilities and is often unused because many such users rely on screen readers.

Modern Russian law, via GOST 2020 (based on WCAG), now prioritizes making the main site accessible for all. A well-designed site with responsive layouts, high contrast, and clear text meets these requirements inherently.

Thus, a single inclusive site is the best practice and legally sufficient. A separate VVI is outdated, potentially discriminatory, and provides an incomplete experience. Following WCAG ensures access for everyone, including those with temporary limitations.

## WCAG-based Guidelines for Developers

### Interaction with website

- Ability to navigate via keyboard (Tab, Enter); all interactive elements can receive visible focus.
    - Visible Focus
    - Logical Focus Order
    - All controls accessible via keyboard
    - Ability to skip navigation (skip link). The "skip to main content" button is hidden by default but becomes visible and in focus upon the first press of the Tab key. Pressing Enter then moves the focus directly to the main content, bypassing, for example, the navigation.

    *Step 1. Press Tab*

    *Step 2. Press Enter*

    - No keyboard traps (focus traps).

- Compatibility with screen readers; proper labels for all site elements are necessary.

### Color

- Avoid using color as the only visual means of conveying information or indicating an action; color should be combined with other identifiers (icons, explanatory text).
- Sufficient color contrast.

### Text

- Regular text and text in images must have a contrast ratio of at least 4.5:1 (Level AA), at least 7:1 (Level AAA).
- Font size can be increased up to 200% natively in the browser without causing a horizontal scrollbar (ensured by adaptive layout).
- A line should not exceed 80 characters (Level AAA). This practice from the era of small monitors aimed to improve code readability.
- Justified text alignment is prohibited (Level AAA). It is less readable than left-aligned text due to uneven word spacing.
- Line spacing should be at least 1.5 times the font size; paragraph spacing at least 2 times the font size; letter spacing at least 0.12 times the font size; word spacing at least 0.16 times the font size. (Level AA)
- Minimum line spacing of at least 1.5 and paragraph spacing at least 1.5 times greater (Level AAA).
- Text in images should be used only for decoration.
- Text on buttons and key elements should be informative.
- All input fields must have meaningful text labels.
- Font must be legible.

### Non-text Content

- Controls must have a description of their purpose (e.g., title attribute for links).
- Media content must be accompanied by synchronized captions.
- Content created for decorative purposes should be ignored by screen readers (images without semantic meaning should have an empty alt='' attribute).
- For CAPTCHA, its purpose should be described. Alternative forms of CAPTCHA accessible through other means of perception must be provided.
- Images and links must have alternative descriptions (alt="").

### Semantic Markup and Page Structure

- Appropriate semantic tags must be used.
- Page division into regions (W3C landmarks).
- Markup must be valid.
- Page has a <title>.
- One <h1> heading (mandatory).
- Logical heading structure (h1 -> h2 -> h3).

### Labels

- Present for all interactive controls.
- Programmatically associated with elements.
- Concise.
- Unique.
- Visible and located near the control.
