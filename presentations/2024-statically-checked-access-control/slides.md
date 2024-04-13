---
layout: cover
theme: seriph
transition: slide-left
---

# Statically checked access control

João G. Santos

github.com/minenwerfer

---
---

# Why

It is very common to have a core library separated from the server logic. This way developers can have a `doSomething()` function that will later be used in several endpoints across the application. Some teams may prefer this setting and even have different people taking care of core functions and server logic. This, however, makes room for access-control-related bugs (`someAdminAction()` being called in a context an unprivileged user is expected).

With the capabilities of TypeScript's type system it is possible to add an extra layer of security and atomically control the access to core functions without runtime overhead.

---
---

# Theory

The signature of the function that defines HTTP endpoints must provide a way to specify allowed roles.
This way we can narrow the type of the callback parameter `context` down to the specified roles.

```typescript
type RouteContext<TAcceptedRoles extends string> = {
  token: {
    sub: any
    roles: readonly (TAcceptedRoles)[]
  }
}

declare const route: <const TRoles extends string[]>(
  method: Uppercase<string>,
  path: `/${string}`,
  cb: (context: RouteContext<NoInfer<TRoles>[number]>) => any,
  options?: {
    roles: TRoles
  }
) => void
```

---
---

Next, our core functions must tell in their signatures which roles they expect with `RouteContext<T>`.
Attempting to pass a invalid context will result in a verbose TypeScript diagnostic telling exactly which roles were expected and which were received instead.

```typescript
declare const businessLogic: (context: RouteContext<'manager'>) => void

declare const context1: RouteContext<'manager'>
// {
//   token: {
//     sub: ObjectId
//     roles: readonly ("manager")[]
//   }
// }

declare const context2: RouteContext<'visitor'>
// {
//   token: {
//     sub: ObjectId
//     roles: readonly ("visitor")[]
//   }
// }

businessLogic(context1) // ok
businessLogic(context2) // Type '"visitor"' is not assignable to type '"manager"'.
```

---
---

# Putting the parts together

Now that our `route()` function narrows the type of `context` according to specified roles and our functions tell which roles they are expecting we can detect functions being called in an insecure context during dev/compile time.

```typescript
declare const businessLogic1: (context: RouteContext<'visitor'>) => void
declare const businessLogic2: (context: RouteContext<'manager'>) => void

route('GET', '/test', (context) => {
  businessLogic1(context) // ok
  businessLogic2(context) // Type '"visitor"' is not assignable to type '"manager"'.

  return {
    success: true
  }
}, {
  roles: [
    'visitor'
  ]
})
```

---
---

**Aeria** is a production-ready, security-focused backend framework that implements this concept out-of-the-box.
Give it a try (and a star!) [on Github](https://github.com/aeria-org/aeria).

---
layout: end
---

# The end

Hope you liked it!

