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


# 00003 Medium Omit

Implement the built-in `Omit<T, K>` generic without using it.

Constructs a type by picking all properties from T and then removing K

For example
```ts
interface Todo {
  title: string
  description: string
  completed: boolean
}

type TodoPreview = MyOmit<Todo, 'description' | 'title'>

const todo: TodoPreview = {
  completed: false,
}
```

## Tests
```ts
import type { Equal, Expect } from '@type-challenges/utils'

type cases = [
  Expect<Equal<Expected1, MyOmit<Todo, 'description'>>>,
  Expect<Equal<Expected2, MyOmit<Todo, 'description' | 'completed'>>>,
]

// @ts-expect-error
type error = MyOmit<Todo, 'description' | 'invalid'>

interface Todo {
  title: string
  description: string
  completed: boolean
}

interface Expected1 {
  title: string
  completed: boolean
}

interface Expected2 {
  title: string
}
```

## Solution
To solve this one, we'll need to use `Exclude` to remove the specified keys and get the rest key with their value in type.

```ts
type MyOmit<T, K extends keyof T> = {
  [KEY in Exclude<keyof T, K>]: T[KEY]
}
```

`Exclude<keyof T, K>` returns the remaining keys, and we can just use mapped types `KEY in TYPES` to get all remaining keys.

## Reference
https://www.typescriptlang.org/docs/handbook/utility-types.html#omittype-keys

https://www.typescriptlang.org/docs/handbook/utility-types.html#excludeuniontype-excludedmembers

# 00008 Medium Readonly 2

Implement a generic `MyReadonly2<T, K>` which takes two type argument `T` and `K`.

`K` specify the set of properties of `T` that should set to Readonly. When `K` is not provided, it should make all properties readonly just like the normal `Readonly<T>`.

For example

```ts
interface Todo {
  title: string
  description: string
  completed: boolean
}

const todo: MyReadonly2<Todo, 'title' | 'description'> = {
  title: "Hey",
  description: "foobar",
  completed: false,
}

todo.title = "Hello" // Error: cannot reassign a readonly property
todo.description = "barFoo" // Error: cannot reassign a readonly property
todo.completed = true // OK
```

## Tests
```ts
import type { Alike, Expect } from '@type-challenges/utils'

type cases = [
  Expect<Alike<MyReadonly2<Todo1>, Readonly<Todo1>>>,
  Expect<Alike<MyReadonly2<Todo1, 'title' | 'description'>, Expected>>,
  Expect<Alike<MyReadonly2<Todo2, 'title' | 'description'>, Expected>>,
]

interface Todo1 {
  title: string
  description?: string
  completed: boolean
}

interface Todo2 {
  readonly title: string
  description?: string
  completed: boolean
}

interface Expected {
  readonly title: string
  readonly description?: string
  completed: boolean
}
```

## Solution
From the example we know that we need to implement fields specified in `K` to be readonly. The idea is to separate those specified fields from the others and add `readonly` operator.

We can use omit to get those non-readonly properties via `Omit`

```ts
type MyReadonly2<T, K extends keyof T> = Omit<T, K> & {
  readonly [KEY in K]: T[KEY]
}
```

This would mostly make tests pass, except for the first one, which hasn't specified any `K`. This case we would make all fields readonly. So it would be equivalent as we specified all of its properties. So we need to give it a default value.

```ts
type MyReadonly2<T, K extends keyof T = keyof T> = Omit<T, K> & {
  readonly [KEY in K]: T[KEY]
}
```

## Summary
### 👉 Default generic type
Generic type can have a default type just like Javascript.

### 👉 Union types via `&`:
https://www.typescriptlang.org/docs/handbook/typescript-in-5-minutes-func.html#unions
### 👉 `Omit`: 
https://www.typescriptlang.org/docs/handbook/utility-types.html#omittype-keys