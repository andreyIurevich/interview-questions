**Mapped Types (Сопоставляемые типы)** — это инструмент метапрограммирования в TypeScript, который позволяет создавать новые типы на основе существующих, циклически перебирая их ключи (свойства).

## 🔥 Главный секрет: Массивы и Кортежи — это тоже Объекты

Мало кто знает, но в TypeScript Mapped Types умеют перебирать не только свойства обычных объектов, но и элементы массивов/кортежей, полностью сохраняя их структуру.

Под капотом TypeScript видит кортеж `[string, number]` как объект с ключами-строками `"0"`, `"1"` и свойством `length`. Поэтому, когда мы напускаем Mapped Type на массив, он не превращает его в плоский объект, а «умно» возвращает обновленный массив (или кортеж) той же длины.

## Пример «магии» на практике:

```typescript
// Универсальный трансформер: оборачивает каждое свойство в Promise
type Asyncify<T> = {
  [K in keyof T]: Promise<T[K]>
};

// 1. Работа с обычным ОБЪЕКТОМ
type User = { name: string; age: number };
type AsyncUser = Asyncify<User>; 
// Итог: { name: Promise<string>; age: Promise<number> }

// 2. РАБОТА С МАССИВОМ/КОРТЕЖЕМ (Тот самый скрытый хак!)
type Coordinates = [number, number];
type AsyncCoordinates = Asyncify<Coordinates>; 
// Итог: [Promise<number>, Promise<number>] — остался КОРТЕЖЕМ!
```

---

## 🛠 Базовый синтаксис и Анатомия

```typescript
type MyMappedType<T> = {
  [K in keyof T]: T[K];
};
```

- `keyof T` — берет все ключи (названия свойств) типа `T` и превращает их в Union (объединение строк), например: `"id" | "name" | "age"`.
- `K in ...` — оператор итерации. Переменная `K` на каждом шаге цикла становится одной конкретной строкой-ключом.
- `T[K]` — Lookup Type (тип поиска). Достает тип значения, которое лежит в исходном объекте `T` под ключом `K`.

---

## ➕ / ➖ Модификаторы (Управление Readonly и Optional)

Внутри Mapped Types можно использовать знаки `+` (добавить) или `-` (удалить) перед ключевыми словами `readonly` и `?`, чтобы массово менять свойства. Если знак не указан, по умолчанию подразумевается `+`.

## 1. Создание Readonly (Встроенный `Readonly<T>`)

```typescript
type CustomReadonly<T> = {
  readonly [K in keyof T]: T[K]; // Добавляет readonly к каждому полю
};
```

## 2. Превращение всех полей в обязательные (Встроенный `Required<T>`)

```typescript
type CustomRequired<T> = {
  [K in keyof T]-?: T[K]; // Минус удаляет знак вопроса (опциональность)
};
```

## 3. Полная очистка (Снимаем readonly и опциональность)

```typescript
type MutableRequired<T> = {
  -readonly [K in keyof T]-?: T[K]; // Делает объект полностью изменяемым и строгим
};
```

---

## 🔄 Продвинутый трюк: Переименование ключей через `as` (Key Remapping)

Начиная с версии TS 4.1, ключи при переборе можно на лету переименовывать (или полностью исключать) с помощью конструкции `as` и Template Literal Types (шаблонных строк).

## 1. Генерируем геттеры для объекта автоматического класса

```typescript
type Getters<T> = {
  [K in keyof T as `get${Capitalize<string & K>}`]: () => T[K]
};

type User = { name: string; age: number };
type UserGetters = Getters<User>;
// Итог: { getName: () => string; getAge: () => number; }
```

## 2. Фильтрация ключей (Аналог встроенного `Omit<T, K>`)

Если в блоке `as` условие возвращает `never`, это свойство выкидывается из финального объекта.

```typescript
// Оставляем только те поля, значения которых являются строками (string)
type OnlyStringFields<T> = {
  [K in keyof T as T[K] extends string ? K : never]: T[K]
};

type Mixed = { id: number; name: string; bio: string };
type Clean = OnlyStringFields<Mixed>;
// Итог: { name: string; bio: string; } (поле id удалено)
```
