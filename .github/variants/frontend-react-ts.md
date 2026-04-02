# Frontend Best Practices — React + TypeScript

<!-- VARIANT FILE: Starting-point conventions for React/TS projects.
     Copy this file to context/best-practices/frontend.md in your project.
     Then edit it to match your project's specific conventions, component library,
     state management approach, and testing setup.
-->

---

## Mandatory Conventions

1. **No `any` types** — use proper TypeScript interfaces/types; `unknown` is acceptable for unvalidated input
2. **Functional components only** — no class components
3. **Props typed with interfaces** — one `interface Props` per component file
4. **Side effects in hooks** — no imperative code in component body outside hooks
5. **No direct DOM manipulation** — use refs (`useRef`) when needed, never `document.querySelector`
6. **State management:** local state (`useState`) for UI state, shared state via Context or dedicated store (see `context/architecture.md`)
7. **No hardcoded strings for user-visible text** — use i18n library or constants file
8. **Accessibility:** semantic HTML + `aria-*` attributes on interactive non-semantic elements

---

## Component Structure

```tsx
// MyComponent.tsx
interface Props {
  title: string;
  onConfirm: (id: number) => void;
  isLoading?: boolean;
}

export function MyComponent({ title, onConfirm, isLoading = false }: Props) {
  const [count, setCount] = useState(0);

  const handleClick = useCallback(() => {
    onConfirm(count);
  }, [count, onConfirm]);

  return (
    <div>
      <h2>{title}</h2>
      <button
        onClick={handleClick}
        disabled={isLoading}
        aria-busy={isLoading}
      >
        Confirm
      </button>
    </div>
  );
}
```

---

## Custom Hook Structure

```tsx
// useMyFeature.ts
interface UseMyFeatureOptions {
  initialValue?: number;
}

interface UseMyFeatureReturn {
  value: number;
  increment: () => void;
  reset: () => void;
}

export function useMyFeature({ initialValue = 0 }: UseMyFeatureOptions = {}): UseMyFeatureReturn {
  const [value, setValue] = useState(initialValue);

  const increment = useCallback(() => setValue(v => v + 1), []);
  const reset = useCallback(() => setValue(initialValue), [initialValue]);

  return { value, increment, reset };
}
```

---

## API Call Pattern

```tsx
// services/myService.ts — pure fetch, no component coupling
export async function fetchItem(id: number): Promise<MyItem> {
  const response = await fetch(`/api/items/${id}`);
  if (!response.ok) throw new ApiError(response.status, await response.text());
  return response.json() as Promise<MyItem>;
}

// In component: use TanStack Query, SWR, or equivalent
const { data, isLoading, error } = useQuery({
  queryKey: ['item', id],
  queryFn: () => fetchItem(id),
});
```

---

## Testing Conventions

### Test Types

| Type | Framework | When |
|------|-----------|------|
| Unit | Jest | Hooks, pure functions, utilities |
| Component | Jest + React Testing Library | Component render and interaction |
| Integration | Jest + RTL | Multi-component flows, Context |

### Coverage Targets

- **Line coverage:** ≥ 80%
- **Branch coverage:** ≥ 70%

### Component Test Structure

```tsx
// MyComponent.test.tsx
import { render, screen, fireEvent } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import { MyComponent } from './MyComponent';

describe('MyComponent', () => {
  const defaultProps = {
    title: 'Test Title',
    onConfirm: jest.fn(),
  };

  beforeEach(() => {
    jest.clearAllMocks();
  });

  it('renders title', () => {
    // Arrange
    render(<MyComponent {...defaultProps} />);

    // Assert
    expect(screen.getByRole('heading', { name: 'Test Title' })).toBeInTheDocument();
  });

  it('calls onConfirm when button clicked', async () => {
    // Arrange
    const user = userEvent.setup();
    render(<MyComponent {...defaultProps} />);

    // Act
    await user.click(screen.getByRole('button', { name: 'Confirm' }));

    // Assert
    expect(defaultProps.onConfirm).toHaveBeenCalledTimes(1);
  });

  it('disables button when isLoading is true', () => {
    // Arrange
    render(<MyComponent {...defaultProps} isLoading />);

    // Assert
    expect(screen.getByRole('button', { name: 'Confirm' })).toBeDisabled();
  });
});
```

### Hook Test Structure

```tsx
// useMyFeature.test.ts
import { renderHook, act } from '@testing-library/react';
import { useMyFeature } from './useMyFeature';

describe('useMyFeature', () => {
  it('initialises with default value', () => {
    const { result } = renderHook(() => useMyFeature());
    expect(result.current.value).toBe(0);
  });

  it('increments value', () => {
    const { result } = renderHook(() => useMyFeature());
    act(() => result.current.increment());
    expect(result.current.value).toBe(1);
  });
});
```

### Query Selectors Priority (RTL)

Use in this order (most accessible → least):
1. `getByRole` — most preferred
2. `getByLabelText`
3. `getByPlaceholderText`
4. `getByText`
5. `getByTestId` — last resort; use `data-testid` attribute sparingly

Never query by CSS class or element tag directly.

### Test Naming Convention

`describe('ComponentName') > it('verb + expected outcome when condition')`

Examples:
- `it('renders error message when fetch fails')`
- `it('disables submit button while loading')`
- `it('calls onDelete with item id when delete clicked')`

---

## Accessibility Checklist (per component)

- [ ] Interactive elements are focusable via keyboard
- [ ] Buttons have a discernible accessible name
- [ ] Form inputs have associated `<label>` or `aria-label`
- [ ] Loading states communicated via `aria-busy` or `aria-live`
- [ ] Images have `alt` text (empty `""` for decorative images)

---

## Logging/Error Convention

```tsx
// Unexpected errors → report to monitoring service
try {
  await riskyOperation();
} catch (error) {
  monitoring.captureError(error, { context: 'MyComponent:handleSubmit' });
  setErrorMessage('Something went wrong. Please try again.');
}

// NEVER console.log in production code — use monitoring or remove
```
