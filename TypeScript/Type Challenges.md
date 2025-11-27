Кастомный Exclude:

```typescript
type MyExclude<T, U> = T extends U ? never : T;
```

TypeScript рассматривает объединения ("union types") **дистрибутивно** в условных типах:
`A | B extends U ? ...`  
распадается на  `(A extends U ? ...) | (B extends U ? ...)`

Пример:
```typescript
type R = MyExclude<"a" | "b" | "c", "b" | "d">;
```

Выполняется как:
- `"a" extends "b" | "d"` → `false` → остаётся `"a"`
    
- `"b" extends "b" | "d"` → `true` → превращается в `never`
    
- `"c" extends "b" | "d"` → `false` → остаётся `"c"`

Конструкция if:

```typescript
type If<C, T, F> = C extends true ? T : F;
```

Concat:

```typescript
type Concat<T extends readonly unknown[], U extends readonly unknown[]> = [...T,...U];
```

Includes:

```typescript
type Includes<T extends readonly any[], U> =
  T extends [infer Head, ...infer Tail]
    ? Equal<Head, U> extends true
      ? true
      : Includes<Tail, U>
    : false;

// Полное сравнение двух типов (известный трюк)
type Equal<A, B> =
  (<T>() => T extends A ? 1 : 2) extends
  (<T>() => T extends B ? 1 : 2)
    ? true
    : false;
```

Вариант Includes:

```typescript
type Includes<T extends readonly any[], U> = {
  [P in T[number]]: true
}[U] extends true ? true : false;
```

не работает корректно, потому что:

1. `T[number]` превращается в union.
2. Ключи приводятся к string/number/symbol.
3. Lookup не делает строгого сравнения типов.
4. Работает неправильно при unions.
5. Некорректно работает с объектами и ссылочными типами.
6. Условие `U extends key` слишком слабое.

Поэтому этот способ _может работать для простых случаев_, но **не является корректной реализацией Includes** в общем случае.

Push:
```typescript
type Push<T extends any[], U> = [...T, U];
```

Unshift:
```typescript
type Unshift<T extends any[], U> = [U, ...T];
```

Получить типы параметров функции:

```typescript
// Извлечение типа первого параметра функции
type ParamType<T> =
  T extends (arg: infer P) => any
    ? P
    : never; 
    
function foo(x: { a: number }) {
  return x.a;
}

type A = ParamType<typeof foo>; 
// A = { a: number }

// Извлечение типов всех параметров функции
type Params<T> =
  T extends (...args: infer P) => any
    ? P
    : never;

function bar(a: string, b: number) {}

type B = Params<typeof bar>;
// B = [string, number]
```

`infer` — это особое ключевое слово, которое разрешено **только внутри условных типов**, и оно означает: Выведи тип сюда, если сможешь.

Пример:
```typescript
type Return<T> = T extends () => infer R ? R : never;
```

Потому что требуется механизм pattern matching — сопоставления с образцом — который доступен только в условных типах (`T extends Something ? ... : ...`).
