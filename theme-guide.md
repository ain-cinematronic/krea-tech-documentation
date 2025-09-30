# KreaTech Theme Guide

This document provides a comprehensive guide to the KreaTech design system, including colors, typography, and component variants.

## Design Philosophy

### Unified Hover Behavior

**All buttons, regardless of type, share the same hover state**: Accent Blue background (`#3943FF`) with white text and no borders. This creates a consistent, predictable user experience across the entire application.

## Colors

### Primary Colors

- **Visual Yellow**: `#FFF539` - Primary brand color for light backgrounds
- **Visual Black**: `#0C0A01` - Primary text color and dark variant
- **Accent Blue**: `#3943FF` - Universal hover state and interactive elements
- **Light Grey**: `#F4F4F4` - Secondary text and background elements

### Brand Color Usage

- `brand.primaryColorForDarkBg`: Visual Yellow (`#FFF539`)
- `brand.primaryColorForLightBg`: Visual Black (`#0C0A01`)
- `brand.secondaryColorForDarkBg`: Light Grey (`#F4F4F4`)
- `brand.secondaryColorForLightBg`: Visual Black (`#0C0A01`)
- `brand.hoverButtonBg`: Accent Blue (`#3943FF`) - **Universal hover background**
- `brand.hoverButtonText`: White - **Universal hover text color**

## Typography

### Font Family

- **Primary Font**: Roboto (all weights: 300, 400, 500, 600, 700, 800)
- **Body Text**: Roboto Regular (400)
- **Headings**: Roboto Medium/Semi-Bold/Bold (500-700)
- **Navigation**: Roboto Semi-Bold (600)
- **List Items**: Roboto Regular (400) — Fixed from previous bold appearance

### Typography Scale

- **H1**: 2.5rem, Bold (700), Line Height 1.2
- **H2**: 2rem, Semi-Bold (600), Line Height 1.3
- **H3**: 1.75rem, Medium (500), Line Height 1.4
- **H4**: 1.5rem, Medium (500), Line Height 1.5
- **H5**: 1.25rem, Regular (400), Line Height 1.6
- **H6**: 1rem, Regular (400), Line Height 1.7
- **Body**: Regular (400)
- **Navigation**: 1rem, Semi-Bold (600)

## Button Variants

> **Universal Hover State**: All buttons hover to Accent Blue background (`#3943FF`) with white text and no borders.

### Primary Buttons

#### `primary`

- **Background**: Visual Yellow (`#FFF539`)
- **Text**: Visual Black (`#0C0A01`)
- **Border**: None
- **Use Case**: Main call-to-action buttons on dark backgrounds

#### `primaryInverted`

- **Background**: Visual Black (`#0C0A01`)
- **Text**: White
- **Border**: None
- **Use Case**: Main call-to-action buttons on light backgrounds

### Secondary Buttons (Outlined)

#### `secondary`

- **Background**: Transparent
- **Text**: Light Grey (`#F4F4F4`)
- **Border**: 2px solid Light Grey
- **Use Case**: Secondary actions on dark backgrounds

#### `secondaryInverted`

- **Background**: Transparent
- **Text**: Visual Black (`#0C0A01`)
- **Border**: 2px solid Visual Black
- **Use Case**: Secondary actions on light backgrounds

### Specialized Buttons

#### `white`

- **Background**: Light Grey (`#F4F4F4`)
- **Text**: Visual Black (`#0C0A01`)
- **Border**: None
- **Use Case**: Buttons on colored backgrounds (like green completion banners)

#### `black`

- **Background**: Visual Black (`#0C0A01`)
- **Text**: Visual Yellow (`#FFF539`)
- **Border**: None
- **Use Case**: Buttons on yellow/light backgrounds

### Navigation Buttons

#### `navbar`

- **Background**: Transparent
- **Text**: Visual Black (`#0C0A01`)
- **Border**: None
- **Special**: Animated underline effect that turns blue on hover
- **Use Case**: Main navigation links

## Core Button System

### Essential Variants (6 Total)

1. **`primary`** - Yellow button for main actions on dark backgrounds
2. **`primaryInverted`** - Black button for main actions on light backgrounds  
3. **`secondary`** - Outlined button for secondary actions on dark backgrounds
4. **`secondaryInverted`** - Outlined button for secondary actions on light backgrounds
5. **`white`** - Light button for colored backgrounds
6. **`black`** - Dark button for yellow backgrounds

### Special Cases

- **`navbar`** - Navigation links with underline animation

### Note on Variants

While additional variants like `tertiary` exist in the theme for specific edge cases, the 6 core variants above cover 95% of use cases and provide the cleanest, most maintainable approach.

## Usage Guidelines

### Button Selection Decision Tree

1. **What's the background?**
   - Dark background → Use non-inverted variants (`primary`, `secondary`)
   - Light background → Use inverted variants (`primaryInverted`, `secondaryInverted`)
   - Colored background → Use `white` variant
   - Yellow background → Use `black` variant

2. **What's the action importance?**
   - Primary action → Use `primary` or `primaryInverted`
   - Secondary action → Use `secondary` or `secondaryInverted`

3. **Special contexts?**
   - Navigation → Use `navbar`
   - Colored banners → Use `white`

### Hover Behavior

**Every button follows the same pattern**: Hover transforms any button to Accent Blue background (`#3943FF`) with white text and removes all borders. This creates a unified interaction language across the application.

### Accessibility

- All button variants maintain WCAG AA contrast ratios
- Universal hover state provides consistent visual feedback
- Focus states include proper keyboard navigation support

## Hero Sections

### Dynamic Content Heroes

Dynamic content pages (Courses, Modules, Lessons) use configurable hero settings managed through the **Front End Theme** global in Payload CMS.

#### Background Effects

- **Blur Intensity**: Configurable 0-100% blur effect on hero background images
  - Internally maps to CSS `filter: blur()` with 0-20px range
  - Default: 40% (8px blur)
  - Provides visual depth while maintaining readability

#### Overlay Settings

- **Overlay Color**: Customizable HEX color for hero overlays
  - Default: `#000000` (black)
  - Supports any valid HEX color code
- **Overlay Opacity**: Configurable transparency level
  - Range: 0-100% (0 = fully transparent, 100 = fully opaque)
  - Default: 70%
  - Creates readable text contrast over background images

#### Implementation Details

- Settings are fetched server-side for optimal performance
- Uses 60-second cache for fast updates
- Fallback to defaults if settings are unavailable
- Processed by `resolveDynamicHeroStyle()` utility

#### Static vs Dynamic Heroes

- **Static Pages**: Homepage, Courses Overview, News Overview manage hero styles independently through their respective globals
- **Dynamic Pages**: Course, Module, and Lesson pages use Front End Theme settings
- This separation allows fine-tuned control for different page types

### Hero Usage Guidelines

1. **Dynamic Content**: Rely on Front End Theme settings for consistency
2. **Static Pages**: Configure hero styles directly in page-specific globals
3. **Background Images**: Ensure sufficient contrast with overlay settings
4. **Performance**: Hero settings are cached for optimal loading times

## Implementation

### Using Theme Buttons

```tsx
// Primary action
<Button variant="primary">Submit</Button>

// Secondary action on dark background
<Button variant="secondary">Cancel</Button>

// Secondary action on light background
<Button variant="secondaryInverted">Skip</Button>

// On colored backgrounds
<Button variant="white">Update Rating</Button>
```

### Font Application

```tsx
// Automatic via theme (recommended)
<Box>Text automatically uses Roboto</Box>

// Rich text content automatically uses Roboto
<RichText content={content} />

// No manual font imports needed in layout.tsx
```

## Font System Architecture

### Centralized Font Configuration

- **Font Loading**: `@fontsource/roboto` packages imported in `src/app/theme/fonts.ts`
- **Theme Integration**: Fonts configured in Chakra UI theme via `src/app/theme/index.ts`
- **Global Application**: Applied automatically through Chakra UI theme system
- **Rich Text Consistency**: All rich text editors use same font system as UI components

## Examples

### Rating System

- **Progress Banner Button**: `white` variant (on green background)
- **Rating Submit Button**: `primary` variant (main action)
- **Rating Cancel Button**: `secondary` variant (secondary action)

### Course Cards

- **Enroll Buttons**: `black` variant (on light backgrounds)
- **Preview Buttons**: `secondaryInverted` variant

### Navigation

- **Main Menu**: `navbar` variant
- **Mobile Menu**: `primaryInverted` variant

