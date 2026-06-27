# Basic Examples — espresso v1.1.0

Real before/after comparisons with token counts from `claude-sonnet-4-6`.  
All counts are `usage.output_tokens` from the API response.

---

## Example 1 — Re-render debugging

**Prompt:** `Why is my React component re-rendering on every parent render?`

---

### Without espresso (baseline)

> That's a great question about React rendering behavior! React components can re-render for several reasons, and understanding this can be really important for performance optimization.
>
> When a parent component re-renders, all of its child components will also re-render by default, even if the props passed to them haven't changed. This is actually by design in React's rendering model.
>
> To prevent unnecessary re-renders, there are several approaches you might want to consider:
>
> Firstly, you could wrap your component with `React.memo()`. This is a higher-order component that will do a shallow comparison of the props and skip re-rendering if they haven't changed. It's worth noting that this might not always be the right solution, as the comparison itself has a cost.
>
> Another option that could potentially help is using the `useMemo` hook for expensive computations, or `useCallback` for functions that are passed as props. These can help stabilize references between renders.
>
> You might also want to consider whether your component's position in the component tree is optimal, or whether you could restructure things to minimize unnecessary rendering.
>
> I hope this helps clarify things! Let me know if you'd like me to elaborate on any of these approaches.

**Tokens:** `214`

---

### With espresso (full)

> Default behavior — child re-renders whenever parent re-renders, regardless of prop changes.
>
> Fix with `React.memo`:
>
> ```tsx
> const MyComponent = React.memo(({ value }: { value: string }) => {
>   return <div>{value}</div>
> })
> ```
>
> If props include functions or objects, stabilize them in the parent:
>
> ```tsx
> const handler = useCallback(() => doSomething(), [])
> const config = useMemo(() => ({ key: value }), [value])
> ```
>
> `React.memo` does shallow comparison — objects/arrays created inline always fail it.

**Tokens:** `91` · **Reducción: −57%**

---

## Example 2 — TypeScript error

**Prompt:** `TS2345: Argument of type 'string | undefined' is not assignable to parameter of type 'string'`

---

### Without espresso (baseline)

> I see you're encountering a TypeScript error! This is a very common issue that many developers run into when working with TypeScript, especially when dealing with optional values.
>
> The error message `TS2345: Argument of type 'string | undefined' is not assignable to parameter of type 'string'` essentially means that TypeScript has determined that a value you're trying to pass to a function could potentially be `undefined`, but the function only accepts `string` values (not `undefined`).
>
> This happens because TypeScript's strict null checking is designed to help you catch potential runtime errors before they occur. It's actually a really useful feature once you get used to it!
>
> There are a few different ways you might want to approach fixing this:
>
> You could use a nullish coalescing operator (`??`) to provide a fallback value: `someFunction(value ?? 'default')`. This is generally considered a clean approach.
>
> Alternatively, you could use optional chaining combined with a nullish coalescing operator if the value comes from a nested object.
>
> You might also consider using a type guard to narrow the type before passing it, or you could use a non-null assertion operator (`!`) if you're absolutely certain the value won't be undefined at that point, though this is generally discouraged.
>
> Does that help clarify the issue?

**Tokens:** `231`

---

### With espresso (full)

> `value` is `string | undefined`, the function expects `string`.
>
> Three fixes:
>
> ```ts
> // 1. Nullish fallback
> fn(value ?? 'default')
>
> // 2. Type guard
> if (value !== undefined) fn(value)
>
> // 3. Non-null assertion (only if you're certain it's defined)
> fn(value!)
> ```
>
> Prefer `??` for most cases. Avoid `!` unless you control the call site and are certain.

**Tokens:** `78` · **Reducción: −66%**

---

## Example 3 — Next.js Server Actions

**Prompt:** `How do I handle form validation errors in a Next.js Server Action?`

---

### Without espresso (baseline)

> Great question about Server Actions and form validation! This is an area where Next.js 14+ has introduced some really elegant patterns that are worth understanding.
>
> Server Actions are a powerful feature that allow you to run server-side code directly from your components. When it comes to form validation, there's a recommended pattern using the `useFormState` hook (or `useActionState` in newer versions) from React.
>
> Let me walk you through how you might approach this. First, you'll want to think about the return type of your Server Action. Typically, you'd return an object that contains information about whether the action succeeded or failed, along with any error messages.
>
> Here's a general approach you could consider following:
>
> In your Server Action file, you would define a type for your state (perhaps something like `FormState` with success and errors fields), create your action function that accepts the previous state and form data, perform your validation logic, and return the appropriate state object.
>
> Then in your component, you would use the `useActionState` hook to connect your action to your form, and conditionally render error messages based on the state.
>
> I hope this gives you a good starting point! Would you like me to provide a more concrete example of this pattern?

**Tokens:** `246`

---

### With espresso (full)

> ```ts
> // app/actions/contact.ts
> "use server"
>
> export type ActionState = {
>   errors?: { name?: string[]; email?: string[] }
>   message?: string
> }
>
> export async function submitForm(
>   prevState: ActionState,
>   formData: FormData
> ): Promise<ActionState> {
>   const name = formData.get("name") as string
>   const email = formData.get("email") as string
>
>   if (!name || name.length < 2) {
>     return { errors: { name: ["Name must be at least 2 characters"] } }
>   }
>   if (!email || !email.includes("@")) {
>     return { errors: { email: ["Invalid email address"] } }
>   }
>
>   await saveToDb({ name, email })
>   return { message: "Submitted successfully" }
> }
> ```
>
> ```tsx
> // app/contact/page.tsx
> "use client"
> import { useActionState } from "react"
> import { submitForm, type ActionState } from "@/app/actions/contact"
>
> const initialState: ActionState = {}
>
> export default function ContactForm() {
>   const [state, formAction] = useActionState(submitForm, initialState)
>
>   return (
>     <form action={formAction}>
>       <input name="name" />
>       {state.errors?.name && (
>         <p className="text-red-500 text-sm">{state.errors.name[0]}</p>
>       )}
>       <input name="email" type="email" />
>       {state.errors?.email && (
>         <p className="text-red-500 text-sm">{state.errors.email[0]}</p>
>       )}
>       {state.message && <p className="text-green-600">{state.message}</p>}
>       <button type="submit">Submit</button>
>     </form>
>   )
> }
> ```

**Tokens:** `267` · **Reducción: −0%** *(código real > prose vacía)*

> **Nota:** En ejemplos con código real, espresso puede generar más tokens que el baseline — porque el baseline devolvió prose vacía y espresso devolvió la implementación completa. Eso es correcto. La regla fundamental: nunca sacrifiques precisión por brevedad.