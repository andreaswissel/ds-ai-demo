# DS-AI Component Library

This directory contains all reusable components for the DS-AI design system.

## Structure

Each component should follow this structure:

```
src/lib/
  <component-name>/
    <component-name>.component.ts      # Component implementation
    <component-name>.component.scss    # Component styles (use ds-ai- prefix)
    <component-name>.component.spec.ts # Karma/Jasmine tests
    <component-name>.stories.ts        # Storybook stories (CSF format)
    <component-name>.mdx               # Storybook documentation
```

## Adding New Components

1. **Fetch Figma specification** using MCP tools before starting
2. Create component directory in `src/lib/`
3. Implement component using Angular 20+ standalone component pattern
4. Create comprehensive tests covering all variants and states
5. Create Storybook stories and MDX documentation
6. Export component from `src/lib/index.ts`
7. Update component status in Figma

See `.github/copilot-instructions.md` for detailed guidelines.
