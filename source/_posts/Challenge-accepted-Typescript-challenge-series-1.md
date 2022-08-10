---
title: Challenge accepted - Typescript challenge series(1)
date: 2022-08-06 14:16:03
categories: "Front-end"
tags:
- Typescript
- Javascript
---

![logo](https://tsch.js.org/logo.svg)
# Prefix
I found a [repo](https://procomponents.ant.design/en-US/components/field-set#proformcaptcha) on Github about Typescript not long ago which provides a bunch of exercises which helps us to understand the fundamental of Typescript. As most of projects nowadays are using Typescript to implement static typing, I decided to follow along and see how I'll go with those challenges.

# How to play around with it?
There are questions of different levels in the `questions` directory, each of which has a `README`, `test-cases` and `template`. The challenge is to modify the type in `template` to make test cases all pass in `test-cases` .

Let's get started from the easy ones!

# 00004 Easy Pick

Implement the built-in `Pick<T, K>` generic without using it.

Constructs a type by picking the set of properties `K` from `T`

For example:

```ts
interface Todo {
  title: string
  description: string
  completed: boolean
}

type TodoPreview = MyPick<Todo, 'title' | 'completed'>

const todo: TodoPreview = {
    title: 'Clean room',
    completed: false,
}
```

## Tests
```ts
import type { Equal, Expect } from '@type-challenges/utils'

type cases = [
  Expect<Equal<Expected1, MyPick<Todo, 'title'>>>,
  Expect<Equal<Expected2, MyPick<Todo, 'title' | 'completed'>>>,
  // @ts-expect-error
  MyPick<Todo, 'title' | 'completed' | 'invalid'>,
]

interface Todo {
  title: string
  description: string
  completed: boolean
}

interface Expected1 {
  title: string
}

interface Expected2 {
  title: string
  completed: boolean
}
```

## Solution
type `MyPick` should work as the same as the native `Pick` type in Typescript, which will generated a new type the the picked property keys from the given type/interface with same value type of the picked key. See `Pick` in Typescript docs here: https://www.typescriptlang.org/docs/handbook/utility-types.html#picktype-keys

First of all, we need to ensure the second generic type will extend `T`'s properties:
```ts
type MyPick<T, KEYS extends keyof T> = {}
```

Then, we can use mapped types to specify all key types of `KEYS`, and use indexed access type to specify the value type:
```ts
type MyPick<T, KEYS extends keyof T> = {
  [K in KEYS]: T[K]
}
```

At this stage, all test cases should pass with no complaining errors.

# 00007 Easy Readonly
Implement the built-in `Readonly<T>` generic without using it.

Constructs a type with all properties of T set to readonly, meaning the properties of the constructed type cannot be reassigned.

For example:

```ts
interface Todo {
  title: string
  description: string
}

const todo: MyReadonly<Todo> = {
  title: "Hey",
  description: "foobar"
}

todo.title = "Hello" // Error: cannot reassign a readonly property
todo.description = "barFoo" // Error: cannot reassign a readonly property
```

## Tests
```ts
import type { Equal, Expect } from '@type-challenges/utils'

type cases = [
  Expect<Equal<MyReadonly<Todo1>, Readonly<Todo1>>>,
]

interface Todo1 {
  title: string
  description: string
  completed: boolean
  meta: {
    author: string
  }
}
```

## Solution
This is a simple one. The solution is to use mapped type to iterate through `Todo` and add `readonly` to each of its keys.

```ts
type MyReadonly<T> = {
  readonly [KEY in keyof T]: T[KEY]
}
```
This shall make its tests all pass.

# 00007 Easy Tuple

Give an array, transform into an object type and the key/value must in the given array.

For example:

```ts
const tuple = ['tesla', 'model 3', 'model X', 'model Y'] as const

type result = TupleToObject<typeof tuple> // expected { tesla: 'tesla', 'model 3': 'model 3', 'model X': 'model X', 'model Y': 'model Y'}
```

## Tests

```ts
import type { Equal, Expect } from '@type-challenges/utils'

const tuple = ['tesla', 'model 3', 'model X', 'model Y'] as const
const tupleNumber = [1, 2, 3, 4] as const
const tupleMix = [1, '2', 3, '4'] as const

type cases = [
  Expect<Equal<TupleToObject<typeof tuple>, { tesla: 'tesla'; 'model 3': 'model 3'; 'model X': 'model X'; 'model Y': 'model Y' }>>,
  Expect<Equal<TupleToObject<typeof tupleNumber>, { 1: 1; 2: 2; 3: 3; 4: 4 }>>,
  Expect<Equal<TupleToObject<typeof tupleMix>, { 1: 1; '2': '2'; 3: 3; '4': '4' }>>,
]

// @ts-expect-errorr
type error = TupleToObject<[[1, 2], {}]>
```

## Solution

First we need to ensure that this type accepts `const` arrays(tuples):
```ts
type TupleToObject<T extends readonly any[]> = any
```

Next, we want to get all of the value inside of it as a union type to iterate through the tuple just like we use mapped type to iterate through an object type.
In array/tuple, we cannot use `keypf T` operator, instead we can treat array as an object-like type that has `number` as key of each of its element - we can do `T[number]` to get all of its values. So it's going to be like this:

```ts
type TupleToObject<T extends readonly any[]> = {
  [VAL in T[number]]: VAL
}
```

Till now we solve most of the test cases, except for the error handling case:
```ts
type error = TupleToObject<[[1, 2], {}]>
```
We can see it doesn't like empty object or array/tuple element to be inside. So the  simplest way is to use `extends` to constrain the type to be `number` or `string`

```ts
type TupleToObject<T extends readonly (number | string)[]> = {
  [VAL in T[number]]: VAL
}
```

No it's all good!

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
const arr = [1, 3, 4]
type TLength = arr['length']
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