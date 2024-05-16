---
layout: ../layouts/BlogPost.astro
title: Type Safety & Error Resistant API's
slug: typeSafeErrorResistantApi
description: >-
  Tweaking Josh Tried Upstash fully type-safe API with automatic error retries
  code
tags:
  - technical
added: 2024-05-16T14:07:35.670Z
---

Yesterday I watched a [fantastic video](\[https://youtu.be/Hfpr39kiODM?si=utn105mVzGX_Yn1F]\(https://youtu.be/Hfpr39kiODM?si=utn105mVzGX_Yn1F\)) by [@joshtriedcoding](\[https://twitter.com/joshtriedcoding]\(https://twitter.com/joshtriedcoding\)) on creating an api workflow that uses [QStash](\[https://console.upstash.com/qstash]\(https://console.upstash.com/qstash\)) for its automatic retries.

QStash is something we have recently been taking advantage of for some of our long running background jobs at Spot Ship, and has allowed us to greatly improve throughput. So I was keen to look at how we can incorporate it in other parts of our system.

One problem with an async queue workflow, is that it can lead to very scattered and repetitive code, which makes the whole codebase more difficult to maintain. Or you end up relying on matching flow diagrams to the code, in order to piece everything back together.

However with the workflow created by Josh you are left with something nice and neat, that lets you concentrate on the code and not the piecing together of events.

```typescript
const example = new Workflow();

export const { POST } = example.createWorkflow(
  (step) => {
    step.create(async (input) => {
      // Step 1
      const result = await someWorkload(input)
      return { success: true, result };
    })
    .create((input) => {
      // Step 2
      const result = someWorkload(input)
      return { success: true, result };
    })
    .create(async (input) => {
      // Step 3
      const result = await someWorkload(input)
      return { success: true, result };
    })
    .finally(async (input) => {
      // Step 4
      const result = await someWorkload(input)
      return;
    });
  },
);
```

You can now have multiple steps neatly within one file whilst in reality it all runs on an asynchronous queue calling each step with [automatic retries](https://upstash.com/docs/qstash/features/retry).

## The Problem

I thought we could greatly benefit from this simplicity, so set about implementing it in a branch of one of our codebases. I soon came across a pitfall however, we use Zod heavily e.g. to introduce type certainty on the body whenever you make an api call. If I make a call, on the initial step, input is of type `any`.

I originally found two potential solutions to this:

1. I could add a schema into my first step to and then run `schema.safeParse(input)` to have certainty of the types for the proceeding code.
2. You can hardcode the type of the initial step `example.createWorkflow(
     (step: Step<{example: string}>)`.

In the developers eyes this problem is now solved, I have beautiful types in my IDE and everything is perfect. 👏🏻

Or not 🙈

What happens now when I call that endpoint with the wrong type of body? I get back that everything went okay and move on with my life. However in Qstash it will now continously retry (upto the max configured) and fail on the first step.

I also now have a bunch of code I have to remember to include on every endpoint, so lets improve this.

## Refactoring

First I refactored the code to cut down on duplicated logic and code e.g. by introducing a new function:

```typescript
const nextStep = async (path: string, body: string) => {
      try {
        if (env.NEXT_PUBLIC_VERCEL_ENV === "local") {
          fetch(path, {
            method: "POST",
            headers: {
              "content-type": "application/json",
            },
            body: JSON.stringify(body),
          });
        } else {
          await qstash.publish({
            url: path,
            method: "POST",
            headers: {
              "content-type": "application/json",
            },
            body: JSON.stringify(body),
          });
        }

        return new Response("OK");
      } catch (err) {
        console.error(err);
        return new Response("Workflow error", { status: 500 });
      }
    };
```

Next I modified the createWorkflow method signature to take a zod object for the schema:

```typescript
createWorkflow = (
    setupStep: (step: Step<any>) => void,
    schema: z.ZodObject<any>,
) { ... }
```

This schema is later used on the initial call when `step === null`:

```typescript
if (step === null) {
  const bodyCheck = schema.safeParse(body);
  if (!bodyCheck.success) {
    return new Response(bodyCheck.error.message, { status: 500 });
  }
  return await nextStep(`${baseUrl}${pathname}?step=0`, body);
}
```

At this point we now have concrete knowledge of what the initial step is going to return but your IDE will still say the type is `any`.

So we can wrap this up with on of my favourite uses of zod, inferring types from their schema 🎉

This is done through using `z.infer<typeof schema>`

## Final Solution

Leaving us with something like the following:

```typescript
const schema = z.object({
  id: z.string(),
  type: z.string(),
});

export const { POST } = example.createWorkflow(
  (step: Step<z.infer<typeof schema>>) => {
    step.create((input) => {
      // The first input is now type safe and so you can carry on with whatever you like
      return { success: true };
    });
  },
  schema,
);
```

I later intend to further upgrade this to support [Failure-Callbacks](https://upstash.com/docs/qstash/features/callbacks#what-is-a-failure-callback) and the many other features to productionise this system but it is a fantastic basis to build upon.

You can find all the adapted code [here](https://gist.github.com/Designibl-Mark/4239a20bdb9cb2fde2e5ffe72f807ab1).

I highly recommend watching [Josh's video](https://www.youtube.com/watch?v=Hfpr39kiODM\&list=WL\&index=69\&t=2s), it is a great insight in how to create code like this and has an excellent explanation/use case for typescript generics.

Let me know if any of this was helpful or revise it yourself and share your gist.
