---
title: Challenge accepted - Typescript challenge series(3)
date: 2022-08-21 21:34:06
categories: "Front-end"
tags:
- Typescript
- Javascript
---

![logo](https://tsch.js.org/logo.svg)

# 00002 Medium Return Type

Implement the built-in `ReturnType<T>` generic without using it.

For example

```ts
const fn = (v: boolean) => {
  if (v)
    return 1
  else
    return 2
}

type a = MyReturnType<typeof fn> // should be "1 | 2"
```

## Tests
```ts
type cases = [
  Expect<Equal<string, MyReturnType<() => string>>>,
  Expect<Equal<123, MyReturnType<() => 123>>>,
  Expect<Equal<ComplexObject, MyReturnType<() => ComplexObject>>>,
  Expect<Equal<Promise<boolean>, MyReturnType<() => Promise<boolean>>>>,
  Expect<Equal<() => 'foo', MyReturnType<() => () => 'foo'>>>,
  Expect<Equal<1 | 2, MyReturnType<typeof fn>>>,
  Expect<Equal<1 | 2, MyReturnType<typeof fn1>>>,
]

type ComplexObject = {
  a: [12, 'foo']
  bar: 'hello'
  prev(): number
}

const fn = (v: boolean) => v ? 1 : 2
const fn1 = (v: boolean, w: any) => v ? 1 : 2
```

## Solutions

We need to use `infer` to get the returning type of the function.

```ts
type MyReturnType<T> = T extends () => infer R ? R : never
```

This will pass most of the tests, but not for those functions with parameters. So we need to specify parameters for the function type.

```ts
type MyReturnType<T> = T extends (...args: any) => infer R ? R : never
```

## Reference
https://www.typescriptlang.org/docs/handbook/2/conditional-types.html#inferring-within-conditional-types
