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