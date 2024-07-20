---
title: Typescript challenge practice series(6)
date: 2023-08-26 20:44:45
categories: "Front-end"
tags:
  - Typescript
  - Javascript
---

![logo](https://tsch.js.org/logo.svg)

# 00009 Medium Deep Readonly

https://github.com/type-challenges/type-challenges/tree/main/questions/00009-medium-deep-readonly

Implement a generic `DeepReadonly<T>` which make every parameter of an object - and its sub-objects recursively - readonly.

You can assume that we are only dealing with Objects in this challenge. Arrays, Functions, Classes and so on do not need to be taken into consideration. However, you can still challenge yourself by covering as many different cases as possible.

For example:

```ts
type X = {
  x: {
    a: 1;
    b: "hi";
  };
  y: "hey";
};

type Expected = {
  readonly x: {
    readonly a: 1;
    readonly b: "hi";
  };
  readonly y: "hey";
};

type Todo = DeepReadonly<X>; // should be same as `Expected`
```

## Tests

```ts
import type { Equal, Expect } from "@type-challenges/utils";

type cases = [Expect<Equal<DeepReadonly<X>, Expected>>];

type X = {
  a: () => 22;
  b: string;
  c: {
    d: boolean;
    e: {
      g: {
        h: {
          i: true;
          j: "string";
        };
        k: "hello";
      };
      l: [
        "hi",
        {
          m: ["hey"];
        }
      ];
    };
  };
};

type Expected = {
  readonly a: () => 22;
  readonly b: string;
  readonly c: {
    readonly d: boolean;
    readonly e: {
      readonly g: {
        readonly h: {
          readonly i: true;
          readonly j: "string";
        };
        readonly k: "hello";
      };
      readonly l: readonly [
        "hi",
        {
          readonly m: readonly ["hey"];
        }
      ];
    };
  };
};
```

## Solution

This should be an easy one. The trick is to use mapped types and recursively apply this type.

```ts
type DeepReadonly<T> = {
  readonly [KEY in keyof T]: T extends object ? DeepReadonly<T[KEY]> : T[KEY];
};
```

This won't just make all the test pass as we need to deal with `Function` as it is a type of `object`

```ts
type DeepReadonly<T> = {
  readonly [KEY in keyof T]: T[KEY] extends Function
    ? T[KEY]
    : T[KEY] extends object
    ? DeepReadonly<T[KEY]>
    : T[KEY];
};
```

## Summary

👉 `Function` extends `object`

# 00010 Medium Tuple to Union

https://github.com/type-challenges/type-challenges/tree/main/questions/00010-medium-tuple-to-union

Implement a generic `TupleToUnion<T>` which covers the values of a tuple to its values union.

For example

```ts
type Arr = ["1", "2", "3"];

type Test = TupleToUnion<Arr>; // expected to be '1' | '2' | '3'
```

## Tests

```ts
import type { Equal, Expect } from "@type-challenges/utils";

type cases = [
  Expect<Equal<TupleToUnion<[123, "456", true]>, 123 | "456" | true>>,
  Expect<Equal<TupleToUnion<[123]>, 123>>
];
```

## Solution 1

The easiest solution is to use `Indexed Access Type` to iterate through the tuple type. From previous questions we know that we can iterate through an array/tuple to get all its elements as an union type:

```ts
type TupleToUnion<T extends any[]> = T[number];
```

## Solution 2

Another approach is to use `infer` to recursively get each element from the tuple to form an union type.

```ts
type TupleToUnion<T> = T extends [infer FIRST, ...infer REST]
  ? FIRST | TupleToUnion<REST>
  : never;
```

## Summary

👉 Array/Tuple type can be iterated by index access type as there is a `number` key on it.
