# Basic Examples — espresso v1.1.0

## Example 1 — React re-render (−57% tokens)

**Prompt:** Why is my React component re-rendering on every parent render?

**With espresso:**
```tsx
const MyComponent = React.memo(({ value }: { value: string }) => {
  return <div>{value}</div>
})
const handler = useCallback(() => doSomething(), [])
const config = useMemo(() => ({ key: value }), [value])
```
Tokens: 91 · −57%

## Example 2 — TypeScript TS2345 (−66% tokens)

**Prompt:** TS2345: string | undefined not assignable to string

**With espresso:**
```ts
fn(value ?? 'default')
if (value !== undefined) fn(value)
fn(value!)
```
Tokens: 78 · −66%

## Example 3 — Server Actions form validation

Complete working implementation with useActionState, Zod, Prisma, revalidatePath.
Tokens: 267 (baseline was prose only — code wins over brevity)