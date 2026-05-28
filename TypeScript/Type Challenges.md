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

Потому что требуется механизм pattern matching — сопоставления с образцом — который доступен только в условных типах (`T extends Something ? ... : 

MyRecord

```ts
type PageInfo = {
	title: string
}

type MyRecord<K extends string, T> = {
	[P in K]: T
}

type Pages = MyRecord<"home" | "about" | "contact", PageInfo>

const pages: Pages = {
	home: { title: "Home" },
	about: { title: "About" },
	contact: { title: "Contact" }
}
```

В TypeScript дженерик `K extends string` автоматически умеет работать как с одной строкой, так и с целым объединением (Union) строк.

Если ты передашь в `K` объединение строк, оператор `[P in K]` сам по очереди переберет каждую строку из этого списка и создаст для нее ключ в объекте.

В TypeScript для этого есть встроенный глобальный тип-объединение **`PropertyKey`** (который состоит из `string | number | symbol`).

MyNonNullable

```ts
type MyNonNullable<T> = T extends null | undefined ? never : T;

type Result = MyNonNullable<string | number | null | undefined>

const value1: Result = "hello"

const value2: Result = null
```

MyConstructorParameters

Напишите тип **MyConstructorParameters** который извлекает типы параметров конструктора класса.

```ts
class User {
	constructor(public name: string, public age: number) {}
}

type MyConstructorParameters<T> = T extends new (...args: infer R) => any ? R : never;

type Params = MyConstructorParameters<typeof User>

const params1: Params = ["John", 25]
// const params2: Params = [25, "John"] type error
```

В TypeScript класс на уровне типов определяется как функция-конструктор. Это функция, которую можно вызвать через оператор `new`, и она вернет объект (экземпляр класса).

Чтобы проверить, является ли переданный тип классом, нужно проверить его на соответствие сигнатуре конструктора: `new (...args: any[]) => any`.

Вот как выглядит реализация такого условного типа:

```typescript
type IsClass<T> = T extends new (...args: any[]) => any ? true : false;
```

---

## Примеры проверки

Давай проверим разные типы данных, чтобы увидеть, как это работает:

```typescript
class UserService {
  constructor(public id: number) {}
}

function regularFunction() {}

type Test1 = IsClass<typeof UserService>;    // true (Передали сам класс)
type Test2 = IsClass<typeof regularFunction>;// false (Обычная функция)
type Test3 = IsClass<string>;                // false (Примитив)
type Test4 = IsClass<UserService>;           // false (Внимание! Это тип ИНСТАНСА класса, а не сам класс)
```

## Важнейший нюанс: `UserService` против `typeof UserService`

В TypeScript имя класса несет в себе двойной смысл:

1. `type MyInstance = UserService` — это тип экземпляра (объекта, который создается через `new`). Сам по себе он классом-конструктором не является.
2. `type MyClass = typeof UserService` — это тип самого класса (его функции-конструктора), у которого есть метод `new()`, статические методы и свойства.

Поэтому, если ты передаешь класс в функцию как значение, тебе нужно проверять его через `typeof`:

```typescript
function classFactory<T>(Target: T) {
  type IsTargetClass = IsClass<T>; // Будет true, если передали класс
}

classFactory(UserService); // Внутри T определится как typeof UserService -> true
```

MyInstanceType

Напишите тип **MyInstanceType** который извлекает тип экземпляра класса.

```ts
class User {
	constructor(public name: string) {}
	greet() {
		return `Hello, ${this.name}`
	}
}

type MyInstanceType<T> = T extends new (args: any) => infer R ? R : never;
type UserInstance = MyInstanceType<typeof User>

const user1: UserInstance = new User("John")
// const user2: UserInstance = { name: "John" } type error
```



