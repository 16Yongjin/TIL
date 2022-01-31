# 타입 팁 & 트릭

## 객체 Merge

```ts
// 객체 2개 합치기
type Merge<F, S> = {
  [K in keyof F | keyof S]: K extends keyof S
    ? S[K]
    : K extends keyof F
    ? F[K]
    : never
}

type Merge<A, B> = Pick<A & B, keyof A | keyof B>

// &로 묶인 객체 하나로 합치기 ex) A & B => AB
type Merge<T> = Omit<T, never>

type Merge<T> = Pick<T, keyof T>

type Merge<T> = { [K in keyof T]: T[K] }
```

## 유니온 연산

```ts
type AorB = 'a' | 'b'
type BorC = 'b' | 'c'

// 타입 모두 합치기
type AorBorC = AorB | BorC // 'a' | 'b' | 'c'

// 겹치는 타입만
type B = AorB & BorC // 'a'
```

## 빈 객체

모든 객체는 `{}`에서 시작하므로 `{}`를 `extend`한다고 빈 객체가 아닐 수 있음

```ts
type True = { a: 1 } extends {} ? true : false // true
```

빈 객체는 `Record<any, never>`나 `{ [key: string]: never }`임

```ts
type Empty = Record<any, never>
type True = {} extends Empty ? true : false
```

## `never`인지 판별하기

`never`에 값을 할당할 수 없어서, 배열에도 넣을 수가 없음

```ts
type IsNever<T> = [T] extends [never] ? true : false
```

## 배열의 접근 키는 `number`

```ts
type Array = ['a', 'b', 'c']
type Elements = Array[number] // 'a' | 'b' | 'c'
```

## 유니온인지 확인하기

`extends` 앞에 나온 파리미터로 유니온 요소가 하나씩 들어감. `for-of` 문으로 유니온 내 타입들을 순회하는 느낌

```ts
type IsUnion<T, C = T> = T extends C ? ([C] extends [T] ? false : true) : never
```

`T`가 `string | number`일 때, `[T]`는 `[]`안으로 유니온 값이 하나씩 들어가지만,

`[C]`는 유니온 타입이 통째로 `[]`로 들어감

```
[T] = [string] | [number]
[C] = [string | number]
```

## 인덱스 시그니쳐 없애기

객체의 키로 `string`이나 `number` 타입이 쓰이는데,

해당 타입인지 여부는 `string extends T`로 해당 타입을 `string`이 확장하는지 확인한다.

> `T extends string` 이런식으로 일반 타입을 `extends` 뒤에만 써왔는데 충격..

```ts
type True = 'foo' extends string ? true : false // true
type False = string extends 'foo' ? true : false // false

type RemoveIndexSignature<T> = {
  [K in keyof T as string extends K
    ? never
    : number extends K
    ? never
    : K]: T[K]
}
```

## 퍼센트 파싱하기

`/^(\+|\-)?(\d*)?(\%)?$/`의 정규식을 같는 `"+85%"`, `85%`, `85` 문자를 파싱해서 [`플러스 or 마이너스`, `숫자`, `단위`]로 반환해야 한다.

```ts
type ParseSign<T extends string> = T extends `${infer S}${any}`
  ? S extends '+' | '-'
    ? S
    : ''
  : ''

type ParsePercent<T extends string> = T extends `${any}%` ? '%' : ''

type ParseNumber<T extends string> =
  T extends `${ParseSign<T>}${infer N}${ParsePercent<T>}` ? N : ''

type PercentageParser<A extends string> = [
  ParseSign<A>,
  ParseNumber<A>,
  ParsePercent<A>
]
```

+, - 기호와 퍼센트 단위를 파싱하는 타입을 만들고

이를 활용해 숫자만 파싱하는 타입을 만들고

마지막에 모든 타입을 합쳐서 퍼센트를 파싱한다.

> 예술적이다

## 문자열 포함여부

### 문자열 앞에 들어간 경우

```ts
type StartsWith<T, U extends string> = T extends `${U}${any}` ? true : false
```

### 문자열 끝에 들어간 경우

```ts
type StartsWith<T, U extends string> = T extends `${any}${U}` ? true : false
```

### 중간에 들어간 경우

```ts
type Contains<T, U extends string> = T extends `${any}${U}${any}` ? true : false
```
