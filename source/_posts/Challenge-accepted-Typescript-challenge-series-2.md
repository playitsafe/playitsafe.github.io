---
title: Challenge accepted - Typescript challenge series(2)
date: 2022-08-13 16:12:03
categories: "Front-end"
tags:
- Typescript
- Javascript
---

![logo](https://tsch.js.org/logo.svg)

# 00018 Easy Tuple Length

Create a generic Length, pick the length of the tuple.

For example:

```ts
type tesla = ['tesla', 'model 3', 'model X', 'model Y']
type spaceX = ['FALCON 9', 'FALCON HEAVY', 'DRAGON', 'STARSHIP', 'HUMAN SPACEFLIGHT']

type teslaLength = Length<tesla>  // expected 4
type spaceXLength = Length<spaceX> // expected 5
```

## Tests
```ts
import type { Equal, Expect } from '@type-challenges/utils'

const tesla = ['tesla', 'model 3', 'model X', 'model Y'] as const
const spaceX = ['FALCON 9', 'FALCON HEAVY', 'DRAGON', 'STARSHIP', 'HUMAN SPACEFLIGHT'] as const

type cases = [
  Expect<Equal<Length<typeof tesla>, 4>>,
  Expect<Equal<Length<typeof spaceX>, 5>>,
  // @ts-expect-error
  Length<5>,
  // @ts-expect-error
  Length<'hello world'>,
]

```

## Solution

This is an easy one. From the examples and tests we can see `Length` accepts constant tuple. and we can use `length` property of array/tuple type to get the length. So it'll be like this:

```ts
type Length<T extends readonly any[]> = T['length']
```


# Summary

## 👉 `keyof T` Operator

To get all keys as a union type from a type or interface, we can use mapped types `in`.
```ts
type Point = { x: number; y: number };
type P = keyof Point;

// P is the same type as “x” | “y”
```
More on this: https://www.typescriptlang.org/docs/handbook/2/keyof-types.html#handbook-content



