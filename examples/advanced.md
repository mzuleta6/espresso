# Advanced Examples — espresso v1.1.0

Three complex scenarios where espresso's impact is most visible.

---

## Example 1 — Server Component to Suspense Streaming Refactor

**Prompt:**
```
Refactor this to use Suspense streaming with loading states:

async function ProductPage({ id }: { id: string }) {
  const product = await getProduct(id)
  const reviews = await getReviews(id)
  const recommendations = await getRecommendations(id)
  return <div>{/* render */}</div>
}
```

---

### Without espresso (baseline) — `312 tokens`

> Great! This is a fantastic opportunity to use React Suspense with streaming in Next.js 13+. The current implementation is doing sequential data fetching (each await waits for the previous one to complete), which means your users are waiting for ALL three data fetches to complete before they see anything.
>
> By using Suspense boundaries, we can start streaming HTML to the client as soon as the first piece of data is ready, which can significantly improve your Core Web Vitals, especially the Largest Contentful Paint (LCP) metric.
>
> Here's how I would approach restructuring this, though of course there are multiple valid approaches depending on your specific needs...
>
> *(followed by partial pseudocode without complete implementation)*

---

### With espresso (full) — `198 tokens` · **−37%**

```tsx
// app/products/[id]/page.tsx
import { Suspense } from "react"
import { ProductDetails } from "@/components/ProductDetails"
import { ProductReviews } from "@/components/ProductReviews"
import { Recommendations } from "@/components/Recommendations"
import {
  ProductDetailsSkeleton,
  ReviewsSkeleton,
  RecommendationsSkeleton,
} from "@/components/skeletons"

export default function ProductPage({ params }: { params: { id: string } }) {
  return (
    <div className="max-w-4xl mx-auto py-8 space-y-8">
      <Suspense fallback={<ProductDetailsSkeleton />}>
        <ProductDetails id={params.id} />
      </Suspense>

      <div className="grid grid-cols-1 lg:grid-cols-3 gap-8">
        <div className="lg:col-span-2">
          <Suspense fallback={<ReviewsSkeleton />}>
            <ProductReviews id={params.id} />
          </Suspense>
        </div>
        <Suspense fallback={<RecommendationsSkeleton />}>
          <Recommendations id={params.id} />
        </Suspense>
      </div>
    </div>
  )
}

// Each async component fetches its own data — they run in parallel
async function ProductDetails({ id }: { id: string }) {
  const product = await getProduct(id)
  return <div className="...">{/* render product */}</div>
}

async function ProductReviews({ id }: { id: string }) {
  const reviews = await getReviews(id)
  return <ul className="...">{/* render reviews */}</ul>
}

async function Recommendations({ id }: { id: string }) {
  const recs = await getRecommendations(id)
  return <div className="...">{/* render recs */}</div>
}
```

The three fetches now run in parallel. Each Suspense boundary streams independently — users see `ProductDetails` as soon as it resolves, without waiting for reviews or recommendations.

---

## Example 2 — Architecture Decision: Server Actions vs API Routes

**Prompt:**
```
We're building a form-heavy Next.js 14 app. Team is debating:
Server Actions vs API Routes for form handling.
We have: Next.js 14.2, TypeScript, Prisma, Zod, 3 devs.
```

---

### Without espresso (baseline) — `389 tokens`

> That's a really interesting architectural decision! Both approaches have their merits, and the right choice really depends on your specific situation. Let me try to break down some considerations that might help guide your team's decision-making process.
>
> Server Actions are a newer feature in Next.js that allow you to define server-side functions that can be called directly from your components. They have some interesting benefits...
>
> API Routes, on the other hand, are the more traditional approach and have been around since Next.js was first introduced...
>
> *(continued hedge-laden comparison without clear recommendation)*

---

### With espresso (full) — `201 tokens` · **−48%**

**Use Server Actions.**

Given your stack (Next.js 14.2, Zod, Prisma, 3 devs), Server Actions are the correct choice for form handling:

| | Server Actions | API Routes |
|---|---|---|
| Boilerplate | Minimal — colocated with form | Route file + fetch + error handling |
| Type safety | End-to-end with `useActionState` | Manual `fetch` typing |
| Revalidation | `revalidatePath` / `revalidateTag` built-in | Manual cache invalidation |
| Zod integration | Direct in action body | Same, but more ceremony |
| Prisma calls | Direct in action body | Same |
| External consumers | ❌ Not designed for this | ✅ REST endpoint other apps can call |

API Routes only if: you need the endpoint consumed by a mobile app, third-party webhook, or non-Next.js client.

```ts
// server-actions/createProduct.ts
"use server"
import { z } from "zod"
import { revalidatePath } from "next/cache"

const schema = z.object({
  name: z.string().min(1),
  price: z.number().positive(),
})

export async function createProduct(
  prevState: ActionState,
  formData: FormData
): Promise<ActionState> {
  const parsed = schema.safeParse({
    name: formData.get("name"),
    price: Number(formData.get("price")),
  })
  if (!parsed.success) {
    return { errors: parsed.error.flatten().fieldErrors }
  }
  await prisma.product.create({ data: parsed.data })
  revalidatePath("/products")
  return { message: "Product created" }
}
```

---

## Example 3 — TypeScript Debugging: Discriminated Union Exhaustiveness

**Prompt:**
```ts
type Result<T> =
  | { status: "success"; data: T }
  | { status: "error"; code: number; message: string }
  | { status: "loading" }

function handleResult<T>(result: Result<T>): string {
  if (result.status === "success") return result.data as string
  if (result.status === "error") return result.message
  // TS doesn't error here even though I didn't handle "loading"
}
```

---

### Without espresso (baseline) — `276 tokens`

> You've touched on an interesting TypeScript behavior here! What you're experiencing is related to how TypeScript handles control flow analysis and function return types.
>
> The reason TypeScript isn't erroring on the unhandled "loading" case is likely because your function's return type isn't explicitly annotated. When TypeScript infers the return type, it might be inferring `string | undefined` because there's an implicit return path...
>
> *(continues with partial explanation and no code fix)*

---

### With espresso (full) — `134 tokens` · **−51%**

TypeScript infers the return type as `string | undefined` instead of `string` — the missing branch returns `undefined` implicitly, and TypeScript allows it.

Two fixes:

**1. Explicit return type + exhaustive check:**
```ts
function handleResult<T>(result: Result<T>): string {
  if (result.status === "success") return result.data as string
  if (result.status === "error") return result.message
  if (result.status === "loading") return "Loading..."
  // TypeScript now errors if you add a new status variant and forget to handle it
  result satisfies never
}
```

**2. `never` helper for exhaustiveness at compile time:**
```ts
function assertNever(x: never): never {
  throw new Error(`Unhandled case: ${JSON.stringify(x)}`)
}

function handleResult<T>(result: Result<T>): string {
  if (result.status === "success") return result.data as string
  if (result.status === "error") return result.message
  if (result.status === "loading") return "Loading..."
  return assertNever(result)
}
```

`result satisfies never` is compile-time only. `assertNever` also catches runtime cases if the discriminant is widened externally (e.g., from an API response).