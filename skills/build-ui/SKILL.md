---
name: build-ui
description: Build UI components and pages from Figma designs for Next.js apps with Tailwind CSS. Use when building UI from Figma, creating components, or implementing designs.
---

# Build UI from Figma - Next.js + Tailwind

## Step 1: Get Figma Design

First, check if Figma MCP tools are available (e.g., `mcp__figma__*`). If they are NOT available, stop and let the user know:

> It looks like the Figma MCP server isn't set up yet. To use this skill with Figma designs, follow the setup guide:
> https://help.figma.com/hc/en-us/articles/32132100833559-Guide-to-the-Figma-MCP-server
>
> In the meantime, you can provide a screenshot or description of the design and I'll build from that instead.

If Figma MCP IS available, use it to get the selected node:

1. Screenshot the element
2. Get code/properties from Figma
3. Determine if it's a **page/screen** or a **component**

---

## Step 2: Understand Project Structure

Before creating anything, explore the project to understand its structure:

1. Check for `package.json` to understand dependencies and package manager
2. Look for existing component directories (`src/components/`, `components/`, etc.)
3. Check for a design system / UI library (shadcn/ui, etc.)
4. Find the global styles file for design tokens (`globals.css`, `global.css`, etc.)
5. Check for existing patterns (barrel exports, component structure, naming conventions)

---

## Step 3: Component Placement Decision

### If it's a reusable component:

→ Create in the project's shared components directory (e.g., `src/components/ui/` or `components/ui/`)

### If it's a group of related components:

→ Create folder `src/components/[group-name]/`
→ Add barrel export `index.ts` like:

```ts
export { ComponentA } from "./component-a";
export { ComponentB } from "./component-b";
```

### If it's a page/screen:

→ Create in `src/app/[route]/page.tsx` (App Router)
→ Use route groups `(group-name)` for layouts without URL segments

---

## Step 4: Install Dependencies

Install any needed shadcn components:

```bash
npx shadcn@latest add [component]
```

- Primitives (button, input, card) → UI directory
- Block components → app's components directory

---

## Step 5: Follow Code Patterns

### Component Structure

```tsx
import { cn } from "@/lib/utils";

function MyComponent({
  children,
  className,
  ...props
}: React.ComponentProps<"div">) {
  return (
    <div
      data-slot="my-component"
      className={cn("base-classes", className)}
      {...props}
    >
      {children}
    </div>
  );
}

export { MyComponent };
```

### For variants, use CVA:

```tsx
import { cva, type VariantProps } from "class-variance-authority";

const myVariants = cva("base-classes", {
  variants: {
    variant: { default: "...", secondary: "..." },
    size: { default: "...", sm: "...", lg: "..." },
  },
  defaultVariants: { variant: "default", size: "default" },
});
```

### Forms (react-hook-form + zod)

```tsx
"use client";

import { useForm } from "react-hook-form";
import { zodResolver } from "@hookform/resolvers/zod";
import { Input } from "@/components/ui/input";
import { Button } from "@/components/ui/button";
import { mySchema, type MyFormData } from "@/lib/schemas";

const {
  register,
  handleSubmit,
  formState: { errors, isSubmitting },
} = useForm<MyFormData>({
  resolver: zodResolver(mySchema),
});

// In JSX:
<form onSubmit={handleSubmit(onSubmit)}>
  <div className="space-y-4">
    <div>
      <label>Label</label>
      <Input {...register("fieldName")} />
      {errors.fieldName && (
        <p className="text-sm text-destructive">{errors.fieldName.message}</p>
      )}
    </div>
    <Button type="submit" disabled={isSubmitting}>
      {isSubmitting ? "Loading..." : "Submit"}
    </Button>
  </div>
</form>;
```

---

## Step 6: Design System Reference

> **CRITICAL: Always use the project's design system tokens. NEVER use hardcoded values.**
>
> **Reference**: Check the project's global CSS file for color variables and typography classes. Read it when unsure about available tokens.

### Forbidden Patterns (DO NOT USE)

```tsx
// ❌ WRONG - Native HTML elements when design system components exist
<button onClick={...}>Click me</button>  // Use <Button>
<input type="text" />                    // Use <Input>
<select>...</select>                     // Use <Select>

// ❌ WRONG - Hardcoded hex colors
className = "bg-[#ffffff] text-[#3d414f] border-[#d8d9dc]";

// ❌ WRONG - Hardcoded font families
className = "font-['Helvetica'] font-['Space_Grotesk']";

// ❌ WRONG - Hardcoded font sizes/line-heights
className = "text-[15px] leading-[1.45] text-[13px] leading-[1.5]";

// ❌ WRONG - Arbitrary shadow values when tokens exist
className = "shadow-[2px_2px_15px_0px_rgba(0,0,0,0.02)]";
```

**CRITICAL: Always use design system components instead of native HTML elements:**

- `<button>` → `<Button>` from the UI library
- `<input>` → `<Input>` from the UI library
- `<select>` → `<Select>` from the UI library
- `<textarea>` → `<Textarea>` from the UI library
- `<label>` → `<Label>` from the UI library

This ensures consistent styling, accessibility, and behavior across the application.

### Correct Patterns (USE THESE)

```tsx
// ✅ CORRECT - Design system color tokens (check globals.css for available tokens)
className = "bg-background text-foreground border-border";

// ✅ CORRECT - Semantic color tokens
className = "bg-primary text-primary-foreground";
className = "bg-destructive text-destructive-foreground";
className = "bg-muted text-muted-foreground";

// ✅ CORRECT - Shadow tokens if defined in globals.css
className = "shadow-sm shadow-md shadow-lg";
```

> Before using any color or typography class, verify it exists in the project's globals.css or tailwind config.

---

## Step 7: Design Fidelity

**SVGs and Icons**:

- First, attempt to pull the SVG properly from Figma and use it
- If the SVG pulls correctly and renders properly → use it as-is
- If you encounter issues pulling from Figma, OR if visual verification shows the SVG is malformed/broken:
  - Replace with a **red rectangle SVG placeholder** matching the original dimensions:
    ```tsx
    <svg width="24" height="24" viewBox="0 0 24 24">
      <rect width="24" height="24" fill="red" />
    </svg>
    ```
  - **Inform the user** so they can provide the asset properly
- Red rectangles flag assets that need manual replacement

**Images**:

- Images MUST be downloaded and applied per the design
- Save to `public/` directory

**Visual Accuracy**:

- Match Figma designs as closely as possible
- Colors, spacing, typography must exactly match design specs
- Any deviations must be explicitly justified and documented
- Elements without a defined background → set `bg-transparent`

---

## Step 8: Responsive Design

All components and pages must be responsive across three breakpoints:

| Breakpoint      | Screen Size  | Tailwind Prefix        |
| --------------- | ------------ | ---------------------- |
| Desktop (wide)  | ≥1280px      | `xl:`                  |
| Tablet (medium) | 768px–1279px | `md:` / `lg:`          |
| Mobile (small)  | ≤767px       | default (mobile-first) |

**Requirements**:

- Layout adapts appropriately for each screen size
- Font sizes, spacing, and dimensions scale proportionally
- Navigation and interactive elements optimized for touch on smaller screens
- Use Tailwind responsive prefixes: `md:`, `lg:`, `xl:`

**Example**:

```tsx
<div className="flex flex-col gap-4 md:flex-row md:gap-6 xl:gap-8">
  <div className="w-full md:w-1/2 xl:w-1/3">...</div>
</div>
```

---

## Step 9: Additional Guidelines

**Semantic HTML**:

- Use proper HTML elements (`<nav>`, `<main>`, `<section>`, `<article>`, `<header>`, `<footer>`)
- Use headings in correct hierarchy (`h1` → `h2` → `h3`)

**Accessibility**:

- All interactive elements must be keyboard accessible
- Images need `alt` text
- Form inputs need associated labels
- Use `aria-*` attributes where appropriate
- Ensure sufficient color contrast

**CSS Practices**:

- Use Flexbox/Grid for layouts (not floats)
- Keep code DRY and maintainable
- Prefer Tailwind utility classes over custom CSS

**React Best Practices**:

- AVOID useEffect unless absolutely necessary - it's an escape hatch, not a go-to tool
- Client components marked with `"use client"` directive only when needed
- Prefer Server Components by default

---

## Step 10: Visual Verification

Use Playwright browser MCP to verify your implementation matches the design across all breakpoints.

**Process**:

1. Ensure the dev server is running
2. Navigate to the page/component you built
3. Take screenshots at each breakpoint to verify responsive behavior

**Breakpoint Verification**:

```
# Desktop (1280px+)
→ browser_resize width=1440 height=900
→ browser_navigate to the page
→ browser_take_screenshot

# Tablet (768px-1279px)
→ browser_resize width=1024 height=768
→ browser_take_screenshot

# Mobile (≤767px)
→ browser_resize width=375 height=812
→ browser_take_screenshot
```

**What to Verify**:

- Layout adapts correctly at each breakpoint
- Typography scales appropriately
- Spacing and padding are proportional
- No horizontal overflow or broken layouts
- Interactive elements are touch-friendly on mobile
- Colors and visual elements match Figma design

**If Issues Found**:

- Fix the implementation
- Re-screenshot to confirm the fix
- Document any intentional deviations

---

## Step 11: Checklist Before Done

**Code Quality**:

- [ ] Component uses `data-slot` attribute for styling hooks
- [ ] Uses `cn()` for className merging
- [ ] Forwards `className` and `...props` for flexibility
- [ ] Client components marked with `"use client"` directive only when needed
- [ ] No useEffect unless absolutely necessary
- [ ] If creating grouped components, barrel export exists in index.ts

**Design Fidelity**:

- [ ] **NO hardcoded hex colors** (e.g., `bg-[#ffffff]`) - use design tokens
- [ ] **NO hardcoded fonts** (e.g., `font-['Helvetica']`) - use typography classes
- [ ] Uses project's design tokens from globals.css
- [ ] SVG icons replaced with red rectangle placeholders if broken
- [ ] Images downloaded and placed in `public/`
- [ ] Visual elements match Figma specs exactly
- [ ] Deviations documented if any

**Responsive & Accessibility**:

- [ ] Responsive at all 3 breakpoints (mobile, tablet, desktop)
- [ ] Semantic HTML structure used
- [ ] Accessible (keyboard nav, alt text, labels, aria attributes)
- [ ] Touch-friendly interactive elements on mobile
