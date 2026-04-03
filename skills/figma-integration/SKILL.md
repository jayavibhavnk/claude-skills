---
name: figma-integration
description: Connect Figma designs to code - extract styles, components, and generate implementation code.
metadata:
  priority: 8
  docs:
    - "https://www.figma.com/developers/api"
  pathPatterns:
    - "**/figma/**"
    - "**/design/**"
  bashPatterns:
    - '\bfigma\b'
  promptSignals:
    phrases:
      - "figma"
      - "design"
      - "component"
    anyOf:
      - "design"
      - "ui"
      - "styles"
---

## Figma Integration

### Figma API Setup

```typescript
import fetch from 'node-fetch';

const FIGMA_ACCESS_TOKEN = process.env.FIGMA_ACCESS_TOKEN;

async function figmaRequest(endpoint: string) {
  const response = await fetch(`https://api.figma.com/v1${endpoint}`, {
    headers: { 'X-Figma-Token': FIGMA_ACCESS_TOKEN },
  });
  return response.json();
}
```

### Get File Data

```typescript
async function getFigmaFile(fileKey: string) {
  const file = await figmaRequest(`/files/${fileKey}`);
  return file;
}

// Get specific nodes
async function getFigmaNodes(fileKey: string, nodeIds: string[]) {
  const nodes = await figmaRequest(
    `/files/${fileKey}/nodes?ids=${nodeIds.join(',')}`
  );
  return nodes;
}
```

### Extract Colors

```typescript
function extractColors(document: any): Record<string, string> {
  const colors: Record<string, string> = {};

  function traverse(node: any) {
    if (node.fills) {
      node.fills.forEach((fill: any) => {
        if (fill.type === 'SOLID' && fill.color) {
          const hex = rgbToHex(fill.color);
          colors[node.name] = hex;
        }
      });
    }
    if (node.children) {
      node.children.forEach(traverse);
    }
  }

  traverse(document);
  return colors;
}

function rgbToHex(color: { r: number; g: number; b: number }): string {
  const r = Math.round(color.r * 255);
  const g = Math.round(color.g * 255);
  const b = Math.round(color.b * 255);
  return `#${r.toString(16).padStart(2, '0')}${g.toString(16).padStart(2, '0')}${b.toString(16).padStart(2, '0')}`;
}
```

### Extract Typography

```typescript
interface TextStyle {
  fontFamily: string;
  fontSize: number;
  fontWeight: number;
  lineHeight: number;
  letterSpacing: number;
}

function extractTypography(styles: any): Record<string, TextStyle> {
  const typography: Record<string, TextStyle> = {};

  for (const [name, style] of Object.entries(styles)) {
    if ((style as any).style) {
      const s = (style as any).style;
      typography[name] = {
        fontFamily: s.fontFamily,
        fontSize: s.fontSize,
        fontWeight: s.fontWeight,
        lineHeight: s.lineHeight,
        letterSpacing: s.letterSpacing,
      };
    }
  }

  return typography;
}
```

### Convert to Tailwind

```typescript
function figmaColorToTailwind(color: string): string {
  // Convert hex to Tailwind color format
  const hex = color.replace('#', '');
  return `#[${hex}]`;
}

function generateTailwindConfig(colors: Record<string, string>) {
  const config = {
    colors: Object.fromEntries(
      Object.entries(colors).map(([name, hex]) => [name, hex])
    ),
  };
  return config;
}
```

### Export Assets

```typescript
async function exportFigmaImage(
  fileKey: string,
  nodeId: string,
  format: 'png' | 'svg' | 'jpg' = 'png'
) {
  const response = await figmaRequest(
    `/images/${fileKey}?ids=${nodeId}&format=${format}`
  );

  const imageUrl = response.images[nodeId];
  return imageUrl;
}
```

### Best Practices

1. Use consistent naming conventions in Figma
2. Organize designs with proper component structure
3. Use auto-layout for responsive designs
4. Document design tokens clearly
5. Version control your design files
