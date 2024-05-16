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

Yesterday I watched a \[fantastic video]\([https://youtu.be/Hfpr39kiODM?si=utn105mVzGX\_Yn1F](https://youtu.be/Hfpr39kiODM?si=utn105mVzGX_Yn1F)) by  \[@joshtriedcoding]\([https://twitter.com/joshtriedcoding](https://twitter.com/joshtriedcoding)) on creating an api workflow that uses \[QStash]\([https://console.upstash.com/qstash](https://console.upstash.com/qstash)) for its automatic retries.\
\
QStash is something we have recently been taking advantage of for some of our long running background jobs at Spot Ship and has allowed us to greatly improve throughput. So I was keen to look at how we can incorporate it in other parts of our system. \
\
One problem with an async queue workflow, is that it can lead to very scattered and repetitive code, which makes the whole codebase more difficult to maintain. Or you end up relying on matching flow diagrams to the code, in order to piece everything back together.

However with the workflow created by Josh you are left with something nice and neat that lets you concentrate on the code and not the piecing together of events.\
\


```typescript
const example = new Workflow();

export const { POST } = example.createWorkflow(
  (step) => {
    step.create(async (input) => {
      // Step 1
      const result = await someWorkload(input)
      return { success: true, result };
    })
      .create(async (input) => {
        // Step 2
        const result = await someWorkload(input)
        return { success: true, result };
      })
      .create(async (input) => {
        // Step 3
        const result = await someWorkload(input)
        return { success: true, result };
      });
  },
);
```

\
\
