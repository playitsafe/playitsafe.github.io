---
title: Challenge accepted - Typescript challenge series(2)
date: 2022-08-13 16:12:03
categories: "Front-end"
tags:
- Typescript
- Javascript
---

![logo](https://tsch.js.org/logo.svg)

# 00043 Easy Exclude

Implement the built-in `Exclude<T, U>`

```ts
type Result = MyExclude<'a' | 'b' | 'c', 'a'>
// 'b' | 'c'
```

## Tests
```ts
import type { Equal, Expect } from '@type-challenges/utils'

type cases = [
  Expect<Equal<MyExclude<'a' | 'b' | 'c', 'a'>, 'b' | 'c'>>,
  Expect<Equal<MyExclude<'a' | 'b' | 'c', 'a' | 'b'>, 'c'>>,
  Expect<Equal<MyExclude<string | number | (() => void), Function>, string | number>>,
]
```

## Solution

When both sides of `extends` are union types, `extends` will iterate through each type inside to check.


```ts
type MyExclude<T, U> = T extends U ? never : T
```

# Summary

## 👉 `T extends U`
Union type will iterate through each union type to check if a type in `T` exists in a type in `U`. It works like a `for` loop in javascript.

