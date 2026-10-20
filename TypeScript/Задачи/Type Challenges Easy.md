`TupleToObject`:

```ts
type TupleToObject<T extends readonly string[]> = {
  [P in T[number]]: P
}
```

`First of Array`:

```ts
type First<T extends any[]> = T extends [] ? never : T[0];
type First<T extends any[]> = T['length'] extends 0 ? never : T[0];
```

`Length of Tuple`:

```ts
type Length<T extends readonly any[]> = T['length'];
```

Кастомный `Exclude`:

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

Конструкция `if`:

```typescript
type If<C, T, F> = C extends true ? T : F;
```

`Concat`:

```typescript
type Concat<T extends readonly unknown[], U extends readonly unknown[]> = [...T,...U];
```

`Includes`:

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

Вариант `Includes`:

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

`Push`:
```typescript
type Push<T extends any[], U> = [...T, U];
```

`Unshift`:
```typescript
type Unshift<T extends any[], U> = [U, ...T];
```


