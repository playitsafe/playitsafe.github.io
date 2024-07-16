---
title: Typescript challenge series(2)
date: 2022-08-10 19:25:03
categories: "Front-end"
tags:
  - Typescript
  - Javascript
---

![logo](https://tsch.js.org/logo.svg)

# Prefix

I found a [repo](https://github.com/type-challenges/type-challenges) on Github about Typescript not long ago which provides a bunch of exercises which helps us to understand the fundamental of Typescript. As most of projects nowadays are using Typescript to implement static typing, I decided to follow along and see how I'll go with those challenges.

# How to play around with it?

There are questions of different levels in the `questions` directory, each of which has a `README`, `test-cases` and `template`. The challenge is to modify the type in `template` to make test cases all pass in `test-cases` .

Let's get started!

# 00014 Easy First of Array

Implement a generic `First<T>` that takes an Array `T` and returns it's first element's type.
For example:

```ts
type arr1 = ["a", "b", "c"];
type arr2 = [3, 2, 1];

type head1 = First<arr1>; // expected to be 'a'
type head2 = First<arr2>; // expected to be 3
```

## Tests

```ts
import type { Equal, Expect } from "@type-challenges/utils";

type cases = [
  Expect<Equal<First<[3, 2, 1]>, 3>>,
  Expect<Equal<First<[() => 123, { a: string }]>, () => 123>>,
  Expect<Equal<First<[]>, never>>,
  Expect<Equal<First<[undefined]>, undefined>>
];

type errors = [
  // @ts-expect-error
  First<"notArray">,
  // @ts-expect-error
  First<{ 0: "arrayLike" }>
];
```

## Solution 1

Our first thought was like this:

```ts
type First<T extends any[]> = T[0];
```

This will solve most of test cases, but failed when `T` is an empty array. So we can add a conditional return:

```ts
type First<T extends any[]> = T extends [] ? never : T[0];
```

## Solution 2

Instead of checking `T extends []`, we can check its length type. From previous tests we know there's a `length` property on array/tuple type, so we can do:

```ts
type First<T extends any[]> = T["length"] extends 0 ? never : T[0];
```

## Solution 3

We can also check if `T[0]` is in array:

```ts
type First<T extends any[]> = T[0] extends T[number] ? T[0] : never;
```

## Solution 4

We can also use `infer` to get the type of the first element of the array:

```ts
type First<T extends any[]> = T extends [infer FIRST, ...infer REST]
  ? FIRST
  : never;
```

# 00018 Easy Tuple Length

Create a generic Length, pick the length of the tuple.

For example:

```ts
type tesla = ["tesla", "model 3", "model X", "model Y"];
type spaceX = [
  "FALCON 9",
  "FALCON HEAVY",
  "DRAGON",
  "STARSHIP",
  "HUMAN SPACEFLIGHT"
];

type teslaLength = Length<tesla>; // expected 4
type spaceXLength = Length<spaceX>; // expected 5
```

## Tests

```ts
import type { Equal, Expect } from "@type-challenges/utils";

const tesla = ["tesla", "model 3", "model X", "model Y"] as const;
const spaceX = [
  "FALCON 9",
  "FALCON HEAVY",
  "DRAGON",
  "STARSHIP",
  "HUMAN SPACEFLIGHT",
] as const;

type cases = [
  Expect<Equal<Length<typeof tesla>, 4>>,
  Expect<Equal<Length<typeof spaceX>, 5>>,
  // @ts-expect-error
  Length<5>,
  // @ts-expect-error
  Length<"hello world">
];
```

## Solution

This is an easy one. From the examples and tests we can see `Length` accepts constant tuple. and we can use `length` property of array/tuple type to get the length. So it'll be like this:

```ts
type Length<T extends readonly any[]> = T["length"];
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

## 👉 `[KEY in KTYPES]`

A mapped type is a generic type which uses a union of PropertyKeys (frequently created via a keyof) to iterate through keys to create a type.

More on this:
https://www.typescriptlang.org/docs/handbook/2/mapped-types.html#handbook-content

## 👉 `TODO[KEY]`

Indexed access type. We can use an indexed access type to look up a specific property on another type:

```ts
type Person = { age: number; name: string; alive: boolean };
type Age = Person["age"];

// type Age = number
```

Indexed access type can also get the length of an array as a type.

```ts
const arr = [1, 3, 4];
type TLength = arr["length"];
```

More on this: https://www.typescriptlang.org/docs/handbook/2/indexed-access-types.html#handbook-content

## 👉 `extends`

The keyword `extends` stands for constraints when defining a generic type.

👉 Native `ReadOnly` type will do as the following:

```ts
 interface Todo {
   title: string
   description: string
 }

 const todo: Readonly<Todo> = {
   title: "Hey",
   description: "foobar"
 }

 // will make `todo` as a readonly object:
  const todo: <Todo> = {
   readonly title: "Hey",
   readonly description: "foobar"
 }
```

More on `ReadOnly`: https://www.tutorialsteacher.com/typescript/typescript-readonly

## 👉 `as const` operator

```ts
const tuple = ['tesla', 'model 3'] as const

// will be equivalent to

typeof tuple = readonly ['tesla', 'model 3']`
```

## 👉 To get all values from an array type:

We can do `ARR[number]` to get all values as an union type of the array type `ARR` and use `E in ARR[number]` to iterate through.

## 👉 To get type of the first element in an array

`T[0]`

## 👉 Get length of an array

`T['length']` //also known as indexed

## 👉 extends union type

If we check whether `a` extends `a|2|3`, it will check every type in union type to see if they match.

## 👉 `infer` in array destruction

```ts
T extends [infer FIRST, ...infer REST] ? FIRST : never
```

if `FIRST` can be successfully destructed, then return `FIRST`

```ts
const arr: any[] = []
const [a, ...b] = arr

a ==> undefined
b ==> []
```

To be continued...
