---
name: Premium Roast Operations
colors:
  surface: '#fff8f5'
  surface-dim: '#e1d8d4'
  surface-bright: '#fff8f5'
  surface-container-lowest: '#ffffff'
  surface-container-low: '#fbf2ed'
  surface-container: '#f5ece7'
  surface-container-high: '#efe6e2'
  surface-container-highest: '#e9e1dc'
  on-surface: '#1e1b18'
  on-surface-variant: '#51443e'
  inverse-surface: '#34302c'
  inverse-on-surface: '#f8efea'
  outline: '#83746d'
  outline-variant: '#d5c3bb'
  surface-tint: '#7d553f'
  primary: '#341706'
  on-primary: '#ffffff'
  primary-container: '#4d2c19'
  on-primary-container: '#c29279'
  inverse-primary: '#f0bba0'
  secondary: '#725a41'
  on-secondary: '#ffffff'
  secondary-container: '#ffdcbd'
  on-secondary-container: '#796046'
  tertiary: '#201f1d'
  on-tertiary: '#ffffff'
  tertiary-container: '#353432'
  on-tertiary-container: '#9f9c9a'
  error: '#ba1a1a'
  on-error: '#ffffff'
  error-container: '#ffdad6'
  on-error-container: '#93000a'
  primary-fixed: '#ffdbca'
  primary-fixed-dim: '#f0bba0'
  on-primary-fixed: '#301404'
  on-primary-fixed-variant: '#623e2a'
  secondary-fixed: '#ffdcbd'
  secondary-fixed-dim: '#e1c1a2'
  on-secondary-fixed: '#291805'
  on-secondary-fixed-variant: '#59422b'
  tertiary-fixed: '#e5e2df'
  tertiary-fixed-dim: '#c9c6c3'
  on-tertiary-fixed: '#1c1b1a'
  on-tertiary-fixed-variant: '#484745'
  background: '#fff8f5'
  on-background: '#1e1b18'
  surface-variant: '#e9e1dc'
typography:
  headline-lg:
    fontFamily: Inter
    fontSize: 32px
    fontWeight: '700'
    lineHeight: 40px
    letterSpacing: -0.02em
  headline-lg-mobile:
    fontFamily: Inter
    fontSize: 24px
    fontWeight: '700'
    lineHeight: 32px
    letterSpacing: -0.01em
  headline-md:
    fontFamily: Inter
    fontSize: 24px
    fontWeight: '600'
    lineHeight: 32px
    letterSpacing: -0.01em
  headline-sm:
    fontFamily: Inter
    fontSize: 20px
    fontWeight: '600'
    lineHeight: 28px
  body-lg:
    fontFamily: Inter
    fontSize: 16px
    fontWeight: '400'
    lineHeight: 24px
  body-md:
    fontFamily: Inter
    fontSize: 14px
    fontWeight: '400'
    lineHeight: 20px
  label-bold:
    fontFamily: Inter
    fontSize: 12px
    fontWeight: '600'
    lineHeight: 16px
  label-sm:
    fontFamily: Inter
    fontSize: 11px
    fontWeight: '500'
    lineHeight: 14px
  data-display:
    fontFamily: Inter
    fontSize: 28px
    fontWeight: '700'
    lineHeight: 32px
    letterSpacing: -0.5px
rounded:
  sm: 0.25rem
  DEFAULT: 0.5rem
  md: 0.75rem
  lg: 1rem
  xl: 1.5rem
  full: 9999px
spacing:
  base: 8px
  container-margin: 24px
  gutter: 16px
  stack-sm: 8px
  stack-md: 16px
  stack-lg: 32px
---

## Brand & Style

This design system establishes a sophisticated **Corporate Modern** aesthetic tailored for the financial and operational oversight of high-end retail. The personality is authoritative yet warm, bridging the gap between the tactile experience of a premium café and the analytical precision required for a dashboard environment.

The target audience consists of stakeholders, franchise owners, and corporate managers who require high data density without sacrificing visual elegance. The UI evokes a sense of reliability and refined taste, utilizing generous whitespace and a structured grid to manage complex financial information. The style avoids excessive ornamentation, focusing instead on clarity, subtle depth, and a palette that reinforces the brand's core identity.

## Colors

The color strategy is rooted in the natural tones of the provided logo, utilizing a "Coffee and Cream" hierarchy. 

- **Primary (#4D2C19):** A deep espresso brown used for key brand touchpoints, primary actions, and high-level typography. It provides the grounding force of the UI.
- **Secondary (#D9B99B):** A warm latte cream used for accents, secondary highlights, and subtle callouts.
- **Tertiary/Surface (#F5F1EE):** A light milk-froth neutral used for card backgrounds and container fills to provide soft contrast against the page background.
- **Functional Neutrals:** A slate-toned charcoal (#2D2926) is used for secondary body text to ensure maximum readability, while a clean off-white (#F9F7F5) serves as the global canvas background to reduce eye strain during long-form data analysis.

## Typography

The typography uses **Inter** to achieve a clean, systematic look essential for financial dashboards. The hierarchy is strictly enforced to guide the eye through complex datasets.

Headlines use tighter letter-spacing and heavier weights to establish clear section breaks. A specialized `data-display` style is included for Key Performance Indicators (KPIs), ensuring large currency figures and percentages are legible at a glance. For mobile contexts, headlines scale down to prevent text wrapping, while body sizes remain constant to ensure accessibility.

## Layout & Spacing

This design system employs a **Fixed Grid** model on desktop (1280px max-width) and a **Fluid Grid** on mobile and tablet. 

- **Desktop:** 12-column grid with 24px margins and 16px gutters.
- **Tablet:** 8-column grid with 20px margins.
- **Mobile:** 4-column grid with 16px margins.

Spacing follows an 8px linear scale. For financial dashboards, vertical rhythm is maintained through standard stack units, ensuring that rows of data and input groups remain aligned and balanced. Components should prioritize an "Airy-Compact" balance—dense enough to show significant data, but with enough margin (16px internal card padding) to prevent visual clutter.

## Elevation & Depth

Visual hierarchy is achieved through a combination of **Tonal Layers** and **Ambient Shadows**. 

1.  **Level 0 (Base):** The canvas background (#F9F7F5).
2.  **Level 1 (Cards/Surface):** White or Tertiary-filled surfaces with a subtle 1px border (#E5E1DE) and a "Soft Bloom" shadow (0px 2px 4px rgba(77, 44, 25, 0.05)).
3.  **Level 2 (Hover/Active):** An increased shadow (0px 8px 16px rgba(77, 44, 25, 0.08)) to indicate interactivity.

Shadows must use a tiny percentage of the primary brown color (#4D2C19) rather than pure black to maintain the warm, premium feel of the retail brand.

## Shapes

The design system utilizes a **Rounded** shape language to reflect the organic nature of coffee culture while maintaining professional structure. 

- **Standard Elements:** Buttons, input fields, and small containers use a 0.5rem (8px) radius.
- **Large Containers:** Dashboard cards and modals use 1rem (16px) for a more distinct, soft-premium appearance.
- **Circular Elements:** Avatars and status indicators use a full pill/circle radius.

## Components

### Buttons
- **Primary:** Solid #4D2C19 with white text. 8px border radius.
- **Secondary:** Transparent with #4D2C19 border and text.
- **Ghost:** No background, #4D2C19 text, for low-priority actions in tables.

### Input Fields
- Clean, 1px bordered containers (#D9B99B when active). Labels use `label-bold` style positioned above the field. Error states use a high-contrast muted red that complements the warm palette.

### Cards
- White background, 16px padding, 16px border radius. Used to group financial metrics. Headers within cards should use `headline-sm`.

### Data Tables
- Row-based layout with subtle horizontal separators (1px #F0EEEC). Alternating row highlights are not used; instead, hover states provide row focus.

### Chips/Tags
- Used for status (e.g., "Paid", "Pending"). Backgrounds are high-transparency versions of the status color (e.g., 10% opacity) with dark text to ensure readability on data-heavy screens.