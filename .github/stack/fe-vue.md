# Frontend Best Practices — Vue 3 + TypeScript

<!-- DRAFT — minimal starting point. Fill in your project's specific conventions. -->

---

## Mandatory Conventions

1. **No `any` types** — use proper TypeScript interfaces/types
2. **Composition API only** (`<script setup>`) — no Options API
3. **Props typed with `defineProps<{...}>()`** with explicit interfaces
4. **Emits typed with `defineEmits<{...}>()`**
5. **No direct DOM manipulation** — use template refs and `useTemplateRef`
6. **Side effects in composables** — pure `<script setup>` block for component logic

---

## Component Structure

```vue
<!-- MyComponent.vue -->
<script setup lang="ts">
interface Props {
  title: string;
  isLoading?: boolean;
}

interface Emits {
  confirm: [id: number];
}

const props = withDefaults(defineProps<Props>(), {
  isLoading: false,
});
const emit = defineEmits<Emits>();

const count = ref(0);

function handleConfirm() {
  emit('confirm', count.value);
}
</script>

<template>
  <div>
    <h2>{{ title }}</h2>
    <button :disabled="isLoading" @click="handleConfirm">
      Confirm
    </button>
  </div>
</template>
```

---

## Testing Conventions

### Test Types

| Type | Framework | When |
|------|-----------|------|
| Unit | Vitest | Composables, utilities |
| Component | Vitest + Vue Test Utils | Component render and interaction |

### Coverage Targets

- **Line coverage:** ≥ 80%
- **Branch coverage:** ≥ 70%

### Component Test Structure

```typescript
// MyComponent.test.ts
import { describe, it, expect, vi } from 'vitest';
import { mount } from '@vue/test-utils';
import MyComponent from './MyComponent.vue';

describe('MyComponent', () => {
  it('renders title', () => {
    // Arrange
    const wrapper = mount(MyComponent, { props: { title: 'Test Title' } });

    // Assert
    expect(wrapper.find('h2').text()).toBe('Test Title');
  });

  it('emits confirm when button clicked', async () => {
    // Arrange
    const wrapper = mount(MyComponent, { props: { title: 'Test' } });

    // Act
    await wrapper.find('button').trigger('click');

    // Assert
    expect(wrapper.emitted('confirm')).toHaveLength(1);
  });
});
```

### Test Naming Convention

`describe('ComponentName') > it('verb + expected outcome when condition')`
