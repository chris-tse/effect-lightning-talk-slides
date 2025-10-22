---
title: TypeScript Lies — and Effect Tells the Truth
theme: default
layout: cover
---

# **TypeScript Lies**  
### and Effect Tells the Truth

<hr class="mt-20">

Chris Tse  
_Software Engineer @ Redfin_

---
layout: center
class: text-center
---

## I ❤️ TypeScript 

Type safety • Autocomplete • Refactor confidence

<v-click>

…but it’s still built on JavaScript.  
No standard library, same old runtime quirks.

</v-click>

---
layout: fact
---

## 😬 And worst of all…

<v-click>
<h1>TypeScript Lies!</h1>
</v-click>
---
layout: two-cols
class: text-md
---


<div class="flex flex-col justify-center items-center h-full">

```ts
async function fetchUser(id: string): Promise<User> {
  const res = await fetch(`/api/users/${id}`)
  return res.json()
}
```
</div>

::right::


<div class="flex flex-col justify-center items-center h-full">
<v-click>
  <h3 >TypeScript tells us:</h3>
  <p class="pb-10">"This function returns a <code>Promise&lt;User&gt;</code>!"</p>
  </v-click>

<v-click>
  <h3>But what happens when:</h3>
  <ul class="pl-10">
    <li>The network fails?</li>
    <li>We get a 500?</li>
    <li>The response shape is not a `User`?</li>
  </ul>

<p class="mt-4 text-center">
  TypeScript can’t guarantee the type is true.
</p>
</v-click>
</div>
---
layout: center
class: text-center
---

# What if TypeScript told the truth?

## What if it *knew* about failure?

---
layout: center
---

## Enter Effect

The `Effect` type:

```ts
Effect<A, E, R>
```

- A = success
- E = error
- R = environment/dependencies

Lets the compiler know not just the type, but how it can fail and any dependencies.

---
layout: two-cols
class: text-sm
---

### Before

<div class="flex flex-col justify-center items-center h-full">

```ts
const parseNumber = (s: string) => {
  if (isNaN(Number(s))) 
    throw new Error("Not a number")
  return Number(s)
}

//      ┌─── number
//      ▼
const parsed = parseNumber("42")
console.log(`Parsed: ${parsed}`)
```

<v-click>
<ul class="mt-8">
  <li>Errors are implicit</li>
  <li>Type system doesn't know it throws</li>
</ul>
</v-click>

</div>

::right::

### After


<div class="flex flex-col justify-center items-center h-full">

<v-click>

```ts
import { Effect } from "effect"


function parseNumber(s: string) {
  return isNaN(Number(s))
    ? Effect.fail(new Error("Not a number"))
    : Effect.succeed(Number(s))
}

const program = Effect.gen(function*() {
  //       ┌─── Effect.Effect<number, Error, never>
  //       ▼
  const parsed = yield* parseNumber("42")
  yield* Effect.log(`Parsed: ${parsed}`)
}).pipe(NodeRuntime.runMain)
```
</v-click>

<v-click>
<ul class="mt-8">
  <li>✅ Errors are explicit</li>
  <li>✅ TypeScript enforces awareness</li>
  <li>✅ Still just JavaScript</li>
</ul>
</v-click>

</div>


---
layout: center
---

## I’m not here to convince you to use Effect.

<v-click>

If you're writing **production‑grade TypeScript**,  
you’ll eventually end up creating what Effect provides:

- Retry logic  
- Cancellation  
- Dependency Injection  
- Typed error modeling

</v-click>

---
layout: center
---

```ts
const fetchUser = (
  id: number
): Effect.Effect<
  unknown,
  HttpClientError | TimeoutException
> =>
  httpClient.get(`/todos/${id}`).pipe(
    Effect.andThen((response) => response.json),
    Effect.timeout("1 second"),
    Effect.retry({
      schedule: Schedule.exponential(1000),
      times: 3
    }),
    Effect.withSpan("getTodo", { attributes: { id } })
  )
```
---
layout: center
---

# 🧊 This is just the **tip of the iceberg**

<v-click>

> For FP fans:  
> That `Effect<A, E, R>` type? It’s a **monad**.  
> Everything in Effect is **functional and composable**.

</v-click>


<v-click>

And it ships with a full toolkit:  
- Dependency injection  
- Concurrency & scheduling  
- Queues & streams  
- Schemas, tracing, logging

</v-click>

---
layout: quote
---

TypeScript makes JavaScript safer.  
**Effect makes TypeScript honest.**

---
layout: default
---

## Resources

- [Official docs](https://effect.website)
- [Visual Effect](https://effect.kitlangton.com)  

