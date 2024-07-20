---
title: Typescript challenge practice series(4)
date: 2023-01-05 19:32:15
categories: "Front-end"
tags:
  - Typescript
  - Javascript
---

![logo](https://tsch.js.org/logo.svg)

# 00268 Easy If

https://github.com/type-challenges/type-challenges/tree/main/questions/00268-easy-if

Implement a util `If` which accepts condition `C`, a truthy return type `T`, and a falsy return type `F`. `C` is expected to be either true or false while `T` and `F` can be any type.

For example:

```ts
type A = If<true, "a", "b">; // expected to be 'a'
type B = If<false, "a", "b">; // expected to be 'b'
```

## Tests

```ts
import type { Equal, Expect } from "@type-challenges/utils";

type cases = [
  Expect<Equal<If<true, "a", "b">, "a">>,
  Expect<Equal<If<false, "a", 2>, 2>>
];

// @ts-expect-error
type error = If<null, "a", "b">;
```

## Solution

This is an easy one. From previous questions we know we can restrict `C` to extend `boolean` and use a tertiary expression to return `T` or `F`.

```ts
type If<C extends boolean, T, F> = C extends true ? T : F;
```

By now we should get all tests pass. If there's still any error in `type error = If<null, 'a', 'b'>`, it is because in TS non-strict mode, `null` can extend `boolean`. We can simple turn on strict mode.

```json
    "strict": true, // Enable all strict type-checking options.
    "strictNullChecks": true, // When type checking, take into account 'null' and 'undefined'.
```

# 00898 Easy includes

https://github.com/type-challenges/type-challenges/tree/main/questions/00898-easy-includes

Implement the JavaScript `Array.includes` function in the type system. A type takes the two arguments. The output should be a boolean `true` or `false`.

For example:

```ts
type isPillarMen = Includes<["Kars", "Esidisi", "Wamuu", "Santana"], "Dio">; // expected to be `false`
```

## Test cases

```ts
import type { Equal, Expect } from "@type-challenges/utils";
import { Includes } from "./template";

type cases = [
  Expect<
    Equal<Includes<["Kars", "Esidisi", "Wamuu", "Santana"], "Kars">, true>
  >,
  Expect<
    Equal<Includes<["Kars", "Esidisi", "Wamuu", "Santana"], "Dio">, false>
  >,
  Expect<Equal<Includes<[1, 2, 3, 5, 6, 7], 7>, true>>,
  Expect<Equal<Includes<[1, 2, 3, 5, 6, 7], 4>, false>>,
  Expect<Equal<Includes<[1, 2, 3], 2>, true>>,
  Expect<Equal<Includes<[1, 2, 3], 1>, true>>,
  Expect<Equal<Includes<[{}], { a: "A" }>, false>>,
  Expect<Equal<Includes<[boolean, 2, 3, 5, 6, 7], false>, false>>,
  Expect<Equal<Includes<[true, 2, 3, 5, 6, 7], boolean>, false>>,
  Expect<Equal<Includes<[false, 2, 3, 5, 6, 7], false>, true>>,
  Expect<Equal<Includes<[{ a: "A" }], { readonly a: "A" }>, false>>,
  Expect<Equal<Includes<[{ readonly a: "A" }], { a: "A" }>, false>>,
  Expect<Equal<Includes<[1], 1 | 2>, false>>,
  Expect<Equal<Includes<[1 | 2], 1>, false>>,
  Expect<Equal<Includes<[null], undefined>, false>>,
  Expect<Equal<Includes<[undefined], null>, false>>
];
```

## Solution

My first thought on this one would be this:

```ts
type Includes<T extends readonly any[], U> = U extends T[number] ? true : false;
```

Unfortunately, it fails on some tests. The reason is the expression `U extends T[number] ? true : false` will compare the type of every elements in `T` with `U`, and return `true` or `false` for each comparison, in the end, the type would be a union type of all `true`/`false` results.
`true | true` would be `true` and `true | false` will be `boolean`

We can use another approach to use `infer` to get each element of the array and use the provided `Equal` function to recursively do comparisons.

```ts
type Includes<T extends readonly any[], U>
  = T extends [infer FIRST, ...infer REST]
    ? Equal<FIRST, U> ? true : Includes<REST, U>
    : false
```

## Summary

We can recursively use this `Include` type by passing `REST` as a generic type.

# 03057 Easy Push & 03060 Easy Unshift

https://github.com/type-challenges/type-challenges/tree/main/questions/03057-easy-push
https://github.com/type-challenges/type-challenges/tree/main/questions/03060-easy-unshift

Implement the generic version of `Array.push`

For example:

```typescript
type Result = Push<[1, 2], "3">; // [1, 2, '3']
```

Implement the type version of `Array.unshift`

For example:

```typescript
type Result = Unshift<[1, 2], 0>; // [0, 1, 2,]
```

The Solution would be the same as `concat`, which is to use the `spread` operator to create a new array type.

```ts
// Push
type Push<T extends any[], U> = [...T, U];

// Unshift
type Push<T extends any[], U> = [...T, U];
```

## Summary

The spread operator can also be used on type.

# 03312 Easy Parameter

Implement the built-in `Parameters<T>` generic without using it.

For example:

```ts
const foo = (arg1: string, arg2: number): void => {};

type FunctionParamsType = MyParameters<typeof foo>; // [arg1: string, arg2: number]
```

## Testing

```ts
import type { Equal, Expect } from "@type-challenges/utils";

const foo = (arg1: string, arg2: number): void => {};
const bar = (arg1: boolean, arg2: { a: "A" }): void => {};
const baz = (): void => {};

type cases = [
  Expect<Equal<MyParameters<typeof foo>, [string, number]>>,
  Expect<Equal<MyParameters<typeof bar>, [boolean, { a: "A" }]>>,
  Expect<Equal<MyParameters<typeof baz>, []>>
];
```

## Solution

We can use `infer` to get the type of function parameter.

```ts
type MyParameters<T extends (...args: any[]) => any> = T extends (...infer ARGS) => any ? ARGS : never

```

Till now, we have finish all EASY level questions.🎉🎉
