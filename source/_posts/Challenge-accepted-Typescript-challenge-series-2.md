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

## Summary

### 👉 `T extends U`
Union type will iterate through each union type to check if every type in `T` exists in a type in `U`. It works like a `for` loop in javascript. And it will return the result as a union type as well.

# 00043 Easy Concat

Implement the JavaScript `Array.concat` function in the type system. A type takes the two arguments. The output should be a new array that includes inputs in ltr order.

For example:
```ts
type Result = Concat<[1], [2]> // expected to be [1, 2]
```

## Tests
```ts
type cases = [
  Expect<Equal<Concat<[], []>, []>>,
  Expect<Equal<Concat<[], [1]>, [1]>>,
  Expect<Equal<Concat<[1, 2], [3, 4]>, [1, 2, 3, 4]>>,
  Expect<Equal<Concat<['1', 2, '3'], [false, boolean, '4']>, ['1', 2, '3', false, boolean, '4']>>,
]
```

## Solution

This one can be easily done by using spread operator:

```ts
type Concat<T extends any[], U extends any[]> = [...T, ...U]
```

## Summary

### 👉 Spread operator `...`
Spread operator can also be used in Typescript.


