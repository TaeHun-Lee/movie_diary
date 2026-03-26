# Design System Strategy: The Ethereal Archive

## 1. Overview & Creative North Star
**The Creative North Star: "The Ethereal Archive"**

This design system rejects the "database" aesthetic of traditional movie trackers in favor of a curated, sensory experience. We are moving beyond a flat list of titles to create a digital sanctuary for cinema lovers. By merging the tactile soft-touch of **Neuromorphism** with the depth and clarity of **Glassmorphism**, we achieve a "Modern-Organic" hybrid. 

The layout breaks the rigid web grid through **Intentional Asymmetry**. Movie posters should not sit in a perfect military line; they should overlap slightly with glass-morphic metadata cards. We use a high-contrast typography scale to create an editorial, magazine-like feel, ensuring the app feels like a personal diary rather than a utility.

---

## 2. Colors & Surface Philosophy
The palette is built on a foundation of high-key neutrals (`surface-bright` and `surface-container-lowest`) to maintain an airy, breathable atmosphere.

### The "No-Line" Rule
**Explicit Instruction:** Traditional 1px solid borders are strictly prohibited for sectioning. Boundaries must be defined solely through background color shifts or tonal transitions. To separate a movie review from the main feed, place a `surface-container-low` card on a `surface` background.

### Surface Hierarchy & Nesting
Treat the UI as a series of physical layers. Use the following tiers to define importance without adding visual noise:
- **Base Layer:** `surface` (#f7f9fc)
- **Primary Content Areas:** `surface-container-low` (#f0f4f8)
- **Interactive Cards:** `surface-container-lowest` (#ffffff)
- **Elevated Overlays:** `surface-container-high` (#e3e9ee)

### The Glass & Gradient Rule
To achieve "The Ethereal Archive" look, floating elements (like Navigation Bars or Quick-Action Modals) must use **Glassmorphism**:
- **Background:** `surface-container-lowest` at 60% opacity.
- **Effect:** `backdrop-blur: 20px`.
- **Signature Texture:** Primary CTAs should use a subtle linear gradient transitioning from `primary` (#4a50c8) to `primary-container` (#787ff9) at a 135-degree angle. This adds "soul" and depth that flat hex codes cannot provide.

---

## 3. Typography: Editorial Contemporary
We pair **Plus Jakarta Sans** (Display/Headlines) with **Manrope** (Body/Labels) to balance character with readability.

*   **Display (Plus Jakarta Sans):** Used for movie titles and "Diary" headers. Use `display-lg` (3.5rem) for hero titles to create an authoritative, cinematic focal point.
*   **Headline (Plus Jakarta Sans):** Use `headline-md` (1.75rem) for section headers. The wide apertures of this font feel "open" and "airy," aligning with our light tone.
*   **Body & Title (Manrope):** Use `body-lg` (1rem) for movie synopses. Manrope’s geometric yet functional structure ensures high legibility during long reading sessions.
*   **Labels:** Use `label-md` (0.75rem) in `on-surface-variant` for metadata (runtime, genre, year).

---

## 4. Elevation & Depth: The New Tactility
Instead of traditional drop shadows, we use **Tonal Layering** and **Ambient Light Simulation**.

*   **The Layering Principle:** Place a `surface-container-lowest` card on a `surface-container` background. The subtle shift from #ffffff to #eaeef3 creates a soft, natural lift.
*   **Ambient Shadows (Neuromorphic):** When a floating effect is required, shadows must be extra-diffused. 
    *   **Light Shadow:** -4px -4px 12px `surface-container-lowest`
    *   **Dark Shadow:** 4px 4px 12px `surface-dim` (at 40% opacity)
*   **The Ghost Border Fallback:** If a border is required for accessibility, use `outline-variant` (#acb3b9) at **15% opacity**. Never use 100% opaque lines.
*   **Glassmorphism Depth:** For modals, use a 1px inner "shine" stroke using `surface-container-lowest` at 50% opacity to simulate the edge of a glass pane.

---

## 5. Components

### Buttons
*   **Primary:** Gradient (`primary` to `primary-container`), `rounded-md` (1.5rem). Use a soft shadow tinted with `primary-dim` at 10% opacity.
*   **Secondary (Neuromorphic):** `surface-container-low` background with a concave inner shadow to look "pressed" into the surface.
*   **Tertiary:** Ghost style, using `primary` text and `title-sm` typography.

### Chips (Genre & Tags)
*   **Style:** `surface-container-highest` background, `rounded-full`. 
*   **Interaction:** On selection, transition to `secondary-container` with an `on-secondary-container` text color.

### Cards & Lists
*   **The "No-Divider" Rule:** Forbid the use of divider lines. Separate list items using `spacing-4` (1.4rem) of vertical white space or by alternating between `surface-container-lowest` and `surface-container-low`.
*   **Movie Posters:** Use `rounded-lg` (2rem) and a very subtle `surface-dim` ambient shadow.

### Input Fields
*   **Style:** Inset Neuromorphic style. Use `surface-container-high` for the background with a 2px inner-shadow to create a "carved" look.
*   **Active State:** The "Ghost Border" becomes `primary` at 30% opacity.

### Featured Movie Backdrop (Signature Component)
*   A full-width `surface-bright` container with a blurred, desaturated version of the movie poster bleeding into the background. Overlay with a Glassmorphic card containing the movie title and "Watchlist" CTA.

---

## 6. Do's and Don'ts

### Do:
*   **Do** use `spacing-8` (2.75rem) and `spacing-12` (4rem) to give elements significant "breathing room."
*   **Do** overlap elements (e.g., a "Rate" button overlapping the edge of a movie poster) to create a custom, high-end feel.
*   **Do** use `backdrop-filter: blur(15px)` on all bottom navigation bars.

### Don't:
*   **Don't** use pure black (#000000) for text. Always use `on-surface` (#2c3338) to maintain the airy tone.
*   **Don't** use standard `rounded-sm` corners. Everything in this system should feel soft; stick to `md` (1.5rem) or `lg` (2rem).
*   **Don't** use "Drop Shadows" from a default library. If it looks like a 2010s shadow, it's too heavy. It should look like light passing through mist.
