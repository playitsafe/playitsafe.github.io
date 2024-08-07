---
title: Typescript challenge practice series(3)
date: 2022-08-13 16:12:03
categories: "Front-end"
tags:
  - Typescript
  - Javascript
---

![logo](https://tsch.js.org/logo.svg)

# 00043 Easy Exclude

https://github.com/type-challenges/type-challenges/tree/main/questions/00043-easy-exclude

Implement the built-in `Exclude<T, U>`

```ts
type Result = MyExclude<"a" | "b" | "c", "a">;
```

## Tests

```ts
import type { Equal, Expect } from "@type-challenges/utils";

type cases = [
  Expect<Equal<MyExclude<"a" | "b" | "c", "a">, "b" | "c">>,
  Expect<Equal<MyExclude<"a" | "b" | "c", "a" | "b">, "c">>,
  Expect<
    Equal<MyExclude<string | number | (() => void), Function>, string | number>
  >
];
```

## Solution

When both sides of `extends` are union types, `extends` will iterate through each type inside to check.

```ts
type MyExclude<T, U> = T extends U ? never : T;
```

## Summary

### 👉 `T extends U`

Union type will iterate through each union type to check if every type in `T` exists in a type in `U`. It works like a `for` loop in javascript. And it will return the result as a union type as well.

# 00533 Easy Concat

https://github.com/type-challenges/type-challenges/tree/main/questions/00533-easy-concat

Implement the JavaScript `Array.concat` function in the type system. A type takes the two arguments. The output should be a new array that includes inputs in ltr order.

For example:

```ts
type Result = Concat<[1], [2]>; // expected to be [1, 2]
```

## Tests

```ts
type cases = [
  Expect<Equal<Concat<[], []>, []>>,
  Expect<Equal<Concat<[], [1]>, [1]>>,
  Expect<Equal<Concat<[1, 2], [3, 4]>, [1, 2, 3, 4]>>,
  Expect<
    Equal<
      Concat<["1", 2, "3"], [false, boolean, "4"]>,
      ["1", 2, "3", false, boolean, "4"]
    >
  >
];
```

## Solution

This one can be easily done by using spread operator:

```ts
type Concat<T extends any[], U extends any[]> = [...T, ...U];
```

## Summary

### 👉 Spread operator `...`

Spread operator can also be used in Typescript.

# 00189 Easy Awaited

We need to get a type which is inside the wrapped type.

```ts
type ExampleType = Promise<string>;

type Result = MyAwaited<ExampleType>; // string
```

## Tests

```ts
import type { Equal, Expect } from "@type-challenges/utils";

type X = Promise<string>;
type Y = Promise<{ field: number }>;
type Z = Promise<Promise<string | number>>;
type Z1 = Promise<Promise<Promise<string | boolean>>>;

type cases = [
  Expect<Equal<MyAwaited<X>, string>>,
  Expect<Equal<MyAwaited<Y>, { field: number }>>,
  Expect<Equal<MyAwaited<Z>, string | number>>,
  Expect<Equal<MyAwaited<Z1>, string | boolean>>
];

// @ts-expect-error
type error = MyAwaited<number>;
```

## Solution

First of all, we can see `MyAwaited` only accepts `Promise` type as its generic type, so we can do this first:

```ts
type MyAwaited<T extends Promise<any>> = any;
```

Then, we can use `infer` to get the type inside the `Promise`

```ts
type MyAwaited<T extends Promise<any>> = T extends Promise<infer INSIDE>
  ? INSIDE
  : never;
```

At this stage, we have solved first two of the test cases. Remaining test cases required us to get the type inside the nested `Promise` type. So we need to check the `INSIDE` type is a `Promise` type as well. If so, we can recursively use `MyAwaited` type.

```ts
type MyAwaited<T extends Promise<any>> = T extends Promise<infer INSIDE>
  ? INSIDE extends Promise<any>
    ? MyAwaited<INSIDE>
    : INSIDE
  : never;
```

By using `MyAwaited` recursively, we can eventually get the type inside.

## Summary

### 👉 `infer` can only be used in conditional type.

More info on

https://www.typescriptlang.org/docs/handbook/release-notes/typescript-4-5.html

https://www.typescriptlang.org/docs/handbook/release-notes/typescript-2-8.html#type-inference-in-conditional-types
