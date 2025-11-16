<!-- Use this file to provide workspace-specific custom instructions to Copilot. For more details, visit https://code.visualstudio.com/docs/copilot/copilot-customization#_use-a-githubcopilotinstructionsmd-file -->

# DS-AI Angular Component Library

This is a **Figma-driven design system** built with Angular 20+ (zoneless, standalone components) and integrated with Storybook. Components are implemented directly from Figma specifications using the **Frametide MCP server** (configured in `.vscode/mcp.json`).

## Architecture Overview

- **Angular 20+**: Zoneless change detection (`provideZonelessChangeDetection()`), standalone components only, new control flow syntax (`@if`, `@for`)
- **Library Structure**: Component library in `/src/lib` - all reusable components go here, NOT in `/src/app`
- **Figma Integration**: Use MCP tools (`mcp_frametide-1_*`) to fetch component specs before implementation - NEVER assume component properties
- **Styling**: SCSS with `ds-ai-` selector prefix, scoped state styles per hierarchy/variant (critical: don't mix state colors across variants)
- **Testing**: Karma + Jasmine (run via `npm test`)
- **Storybook**: Configured to load stories from `/src/lib/**/*.stories.ts` - place stories next to components
- **Formatting**: Prettier configured in `package.json` (100 char width, single quotes, Angular HTML parser)

## Development Workflow

### Starting Development

```bash
npm start              # Dev server on localhost:4200
npm test              # Run Karma tests
npm run build         # Production build
ng run ds-ai:storybook # Run Storybook (CTRL+C to stop)
```

### Component Implementation Flow

1. **Fetch Figma spec first**: Use `mcp_frametide-1_get-component-specification` with fileId and componentId
2. **Analyze variants**: If Figma shows 24 variants, understand the combination (e.g., 6 types × 4 states)
3. **Create component**: Standalone, use signals, prefix selectors with `ds-ai-`
4. **Verify compilation**: Check TypeScript errors before proceeding to stories
5. **Create stories**: CSF format next to component, verify argTypes match actual @Input types
6. **Create MDX docs**: Omit `tags: ['autodocs']` to avoid conflicts with MDX
7. **Update status**: Mark component as implemented in Figma via MCP tools

## Critical Angular 20+ Patterns

### Component Template Example

```typescript
import { Component, input, output } from '@angular/core';
import { NgClass } from '@angular/common'; // Explicit import required!

@Component({
  selector: 'ds-ai-button',
  standalone: true,
  imports: [NgClass], // Nothing is globally available
  template: `
    @if (loading()) {
    <span>Loading...</span>
    }
    <button [ngClass]="getClasses()" [attr.aria-disabled]="disabled()">
      <ng-content />
    </button>
  `,
  styleUrl: './button.scss',
})
export class ButtonComponent {
  hierarchy = input<'primary' | 'secondary'>('primary');
  disabled = input(false);
  // NEVER name @Output same as method: avoid onClick() when you have @Output() click
  buttonClick = output<void>();

  protected getClasses() {
    /* ... */
  }
}
```

### State Styles (Critical Pattern)

```scss
// WRONG: Global state styles mix colors across hierarchies
.ds-ai-button:hover {
  background: var(--hover-bg);
}

// CORRECT: Scope states within each hierarchy
.ds-ai-button {
  &.primary {
    background: var(--primary-bg);
    &:hover {
      background: var(--primary-hover);
    }
    &:focus {
      outline: 2px solid var(--primary-focus);
    }
    &:active {
      background: var(--primary-active);
    }
  }
  &.secondary {
    background: var(--secondary-bg);
    &:hover {
      background: var(--secondary-hover);
    }
    // ... etc
  }
}
```

## Figma Integration Rules

**ALWAYS use MCP tools before implementation:**

- `mcp_frametide-1_list-components` - See all available components
- `mcp_frametide-1_get-component-specification` - Get full component spec (properties, states, styles)
- `mcp_frametide-1_update-component-status` - Mark implementation status

**Property Mapping:**

- Figma: `Hierarchy = Secondary Gray` → Angular template: `hierarchy="secondary-gray"`
- Figma: `Size = sm` → Angular: `size` input → SCSS: `.size-sm` class
- Use camelCase in TypeScript, kebab-case in templates/attributes

## Storybook Conventions

### File Structure

```
src/lib/
  button/
    button.component.ts
    button.component.scss
    button.component.spec.ts     # Karma/Jasmine tests
    button.component.stories.ts  # CSF format, next to component
    button.component.mdx         # Docs (no autodocs tag!)
```

### Stories Pattern (CSF)

```typescript
import type { Meta, StoryObj } from '@storybook/angular';
import { ButtonComponent } from './button.component';

const meta: Meta<ButtonComponent> = {
  component: ButtonComponent,
  // Omit tags: ['autodocs'] when using MDX!
};
export default meta;

type Story = StoryObj<ButtonComponent>;

export const Primary: Story = {
  args: {
    hierarchy: 'primary', // Match ACTUAL @Input types from component
  },
  render: (args) => ({
    props: args,
    template: `<ds-ai-button [hierarchy]="hierarchy">Click me</ds-ai-button>`,
  }),
};
```

**Common Mistakes to Avoid:**

- Don't create argTypes for non-existent properties
- Match data structures exactly: if component expects `string[]`, don't pass `{value, label}[]`
- Verify component exports before importing (class name might differ from filename)

## Component Development Checklist

**Before Writing Code:**

1. Fetch Figma specification via MCP (`mcp_frametide-1_get-component-specification`)
2. Count variants (e.g., 24 = 6 types × 4 states) to understand property combinations
3. Read existing component code if modifying (don't assume interfaces)

**During Implementation:**

1. Create standalone component with explicit imports (NgClass, etc.)
2. Use signals for reactive state, new control flow (`@if`, `@for`)
3. Prefix selectors with `ds-ai-` in SCSS
4. Scope state styles inside hierarchy blocks (never global hover/focus)
5. Use `[attr.aria-*]` for ARIA attributes (not direct bindings)

**After Implementation:**

1. Verify TypeScript compilation (fix errors before stories)
2. Create CSF stories with matching argTypes
3. Create MDX docs (omit `tags: ['autodocs']`)
4. Test all variants/states in Storybook
5. Update Figma status via `mcp_frametide-1_update-component-status`

## Known Configuration Details

- **No ESLint**: Project uses `.editorconfig` + Prettier only
- **Karma config**: Inline in `angular.json` (no `karma.conf.js`)
- **Figma token**: Stored in `.env` (FIGMA_ACCESS_TOKEN), loaded by Frametide MCP
- **MCP server**: Configured in `.vscode/mcp.json` (Node.js stdio server)
- **TypeScript strict mode**: All strict flags enabled in `tsconfig.json`
- **Library structure**: Components live in `src/lib/`, NOT in `src/app/`

## Testing Patterns

### Component Test Example (Karma + Jasmine)

```typescript
import { ComponentFixture, TestBed } from '@angular/core/testing';
import { ButtonComponent } from './button.component';

describe('ButtonComponent', () => {
  let component: ButtonComponent;
  let fixture: ComponentFixture<ButtonComponent>;

  beforeEach(async () => {
    await TestBed.configureTestingModule({
      imports: [ButtonComponent], // Standalone component
    }).compileComponents();

    fixture = TestBed.createComponent(ButtonComponent);
    component = fixture.componentInstance;
    fixture.detectChanges();
  });

  it('should create', () => {
    expect(component).toBeTruthy();
  });

  it('should apply correct hierarchy class', () => {
    fixture.componentRef.setInput('hierarchy', 'primary');
    fixture.detectChanges();

    const button = fixture.nativeElement.querySelector('button');
    expect(button.classList.contains('primary')).toBeTruthy();
  });

  it('should emit click event', () => {
    let clicked = false;
    component.buttonClick.subscribe(() => (clicked = true));

    const button = fixture.nativeElement.querySelector('button');
    button.click();

    expect(clicked).toBeTruthy();
  });

  it('should set aria-disabled when disabled', () => {
    fixture.componentRef.setInput('disabled', true);
    fixture.detectChanges();

    const button = fixture.nativeElement.querySelector('button');
    expect(button.getAttribute('aria-disabled')).toBe('true');
  });
});
```

### Key Testing Practices

- Use `fixture.componentRef.setInput()` for setting signal inputs in tests
- Test all component variants and states
- Verify ARIA attributes are correctly set
- Test event emissions using `subscribe()`
- Use `fixture.detectChanges()` after input changes

## Accessibility Requirements

- Use semantic HTML (`<button>`, `<input>`) over `<div>` with click handlers
- Include ARIA attributes: `[attr.aria-disabled]`, `[attr.aria-label]`, etc.
- Ensure keyboard navigation (Tab, Enter, Space) works
- Test with screen readers where applicable
- Follow WCAG guidelines for color contrast and focus indicators

## Coding guidelines

### setup

- Storybook is already configured to load stories from `/src/lib/**/*.stories.ts`
- All components must be created in `/src/lib/` directory, organized by component name
- Delete sample content from Storybook's `/src/stories` folder if present

### working

- different variations of a component should be managed through inputs on the component itself. For example, `Size = sm` in Figma should convert to a property called size on the component, that also refers to a specific class in the component styles
- try to keep the component HTML clean of any functions related to styling. Applying styles should be managed through properties, corresponding functions should reside in the functional code of the component
- use encapsulated native HTML elements where possible to avoid re-implementing accessibility features
- make sure you are following WCAG guidelines for accessible components
- component attributes should be kebab-case (e.g. `Hierarchy = Secondary Gray` should convert to hierarchy="secondary-gray")
- make sure to include hover and focus states as well
- analyze the Figma design for each component and ensure that all properties, states, and variations are implemented in the component. Pay attention that all background colors, text colors, and other styles are correctly applied
- when composing Storybook stories, make sure to include actual samples that load the component in a proper way. For a button, you will also need to provide a template that renders content and connects component attributes. Make sure to have stories for all states of a component, e.g. Primary, Secondary, etc
- use standalone components only, make sure to include them in the correct way in Storybook
- for every stories file you create, also create an .mdx documentation file
- put stories files next to the component files, not in a separate folder
- remember to omit `tags: ['autodocs']` when creating MDX files for components, as this is conflicting
- use CSF for writing stories, do not use the legacy format or render functions if possible
- When using Angular standalone components, always explicitly import any required Angular features (such as NgClass, NgIf, NgFor, etc.) in the component's imports array. Do not assume they are globally available.
- For native HTML attributes that are not standard Angular inputs (like aria-\* attributes), always use the [attr.attributeName] binding syntax (e.g., [attr.aria-disabled]) instead of [attributeName].
- Use the new Angular control flow syntax (@if, @for, etc.) instead of legacy structural directives like *ngIf and *ngFor, and ensure any required features (e.g., NgComponentOutlet) are also imported.
- Double-check that all template bindings are valid for the element or component they are used on.
- Do not use the same name for an @Output() event and a method or property in the same component. For example, avoid having both an @Output() click and a method named onClick. Always use unique names for @Output() events to prevent accidental recursion or event binding conflicts.
- When implementing state styles (hover, focus, active, disabled) for components with multiple hierarchies or variants, always scope state styles within each hierarchy/variant selector. Do not define global state styles that apply to all variants, as this can cause state color/style mixing.
- For each hierarchy (e.g., primary, secondary, tertiary), explicitly define the hover, focus, and active state styles inside the corresponding hierarchy's SCSS block. This ensures each variant has its own correct state colors and effects.
- Review and test each variant in Storybook to confirm that its interactive states (hover, focus, active, disabled) match the intended design for that hierarchy.

### Critical lessons learned from implementation challenges

#### Figma integration and component specification

- **Always fetch Figma specifications first** before implementing any component to understand the exact properties, states, and data structures required
- **Read component properties carefully** from Figma specifications and match them exactly in the Angular component - don't assume properties that aren't explicitly defined
- **Check component complexity** - if Figma shows 24 variants, understand how they break down (e.g., 6 types × 4 states = 24 variants) to implement the right combination of properties
- **Use actual Figma property names** when possible, but convert them to Angular-friendly formats (camelCase for TypeScript, kebab-case for templates)

#### Component architecture and type safety

- **Always check existing component exports** before creating stories - use the actual exported class name, not assumed names
- **Match component property types exactly** - if a component uses `string[]`, don't create stories with `{value: string, label: string}[]` objects
- **Read the component implementation** before writing stories to understand the actual input/output interface
- **Keep component types consistent** with Figma specifications - if Figma shows 'default', 'icon-leading', etc., use those exact values in TypeScript enums

#### Storybook stories best practices

- **Always verify component property types** before creating argTypes in stories - incorrect types cause TypeScript errors
- **Use simple data structures** for component options unless the component specifically expects complex objects
- **Match story names to actual component capabilities** - don't create stories for properties that don't exist on the component
- **Test stories compilation** before considering the task complete - TypeScript errors in stories prevent proper functionality

#### File naming and organization

- **Be consistent with file naming conventions** - if files are prefixed with `ds-ai-`, apply this consistently across all files
- **Check export names** before importing in other files - component class names may differ from file names
- **Update public APIs** immediately after creating components to ensure they can be imported properly
- **Maintain consistent naming** between component files, story files, and MDX files for better organization

#### Development workflow

- **Always verify component compilation** before moving to stories to catch basic TypeScript errors early
- **Check one file at a time** for errors rather than creating multiple files with potential issues
- **Use read_file to understand existing code** before making assumptions about interfaces or naming
- **Update implementation status** in external systems (like Figma) only after successful implementation and testing

#### Error prevention

- **Don't assume property names** - always verify what properties exist on the component before using them in stories
- **Read error messages carefully** - TypeScript errors often indicate exactly what's wrong (e.g., 'Property X does not exist on type Y')
- **Validate component interface** against stories interface to ensure compatibility
- **Test component integration** by checking if it compiles and exports properly before writing stories
