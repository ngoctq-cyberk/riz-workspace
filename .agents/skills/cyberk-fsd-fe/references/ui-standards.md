# UI Standards: Icons

**Library**: Remix Icon (React Package)
**Package**: `@remixicon/react`

## Usage
**Do not use webfonts (`className="ri-home"`). Use React components.**

```tsx
import { RiHomeLine, RiDeleteBinFill } from "@remixicon/react";

// Standard sizes: 16, 20, 24, 32
<RiHomeLine size={24} />
<RiDeleteBinFill size={20} className="text-red-500" />
```

## Icons vs Buttons
- **Icon decoration**: Use plain icon component.
- **Action**: Wrap in button.
  ```tsx
  <Button variant="ghost" size="icon">
    <RiCloseLine size={24} />
  </Button>
  ```
