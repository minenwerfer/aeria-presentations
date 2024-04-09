---
layout: cover
theme: seriph
transition: slide-left
---

# Aeria

João G. Santos

github.com/minenwerfer

---
---

# Why

This method allows us to control the access to critical functions coming from HTTP endpoints atomically with no runtime overhead.

---
---

# Theory

- We must be able to tell which roles are allowed into a route

```typescript {6-8}
router.POST('/example', (context) => {
  return {
    success: true
  }
}, {
  roles: [
    'manager',
  ]
})
```

---
---

- The `context.token` object is narrowed to contain only roles specified in the third parameter

```typescript {2-3}
router.POST('/example', (context) => {
  // Argument of type '"visitor"' is not assignable to parameter of type '"manager"'.
  context.token.roles.includes('visitor')

  return {
    success: true
  }
}, {
  roles: [
    'manager',
  ]
})
```

---
---

Now, the signature of our functions must specify which individual role or set of roles are allowed to execute them.

```typescript
declare const businessLogic: (token: Token<'manager'>) => void

declare const token1: Token<['visitor', 'manager']>
// {
//   sub: ObjectId,
//   roles: readonly ("visitor" | "manager")[]
// }

declare const token2: Token<'visitor'>
// {
//   sub: ObjectId,
//   roles: readonly ("visitor")[]
// }

businessLogic(token1) // ok
businessLogic(token2) // Type '"visitor"' is not assignable to type '"manager"'.
```

---
---

# Real-world scenario

---
---

First, we create a function that only users with `projectManager` or `supervisor` roles should be able to trigger.

```typescript {*}{lines:true}
const notifySlack = async (notification: SlackNotification, token: Token<['projectManager', 'supervisor']>) => {
  // ...
}
```

Next, we use the function in a route that provides only the `employee` role (the role of the user is checked at the runtime and the request fails if it doesn't match).

```typescript {*}{lines:true,startLine:5}
router.POST('/insert', async (context) => {
  if( someBusinessLogic() ) {
    // Type '"employee"' is not assignable to type '"projectManager" | "supervisor"'.
    await notifySlack("something", context.token)
  }
}, {
  roles: [
    'employee'
  ]
})
```

TypeScript is able to catch the access control violation at the line 8.

---
layout: end
---

# The end

Star the Aeria repository [on Github](https://github.com/aeria-org/aeria)

