Ключевое слово _infer_ дополняет условные типы и не может использоваться вне расширения. Это ключевое слово позволяет нам определить переменную внутри нашего ограничения, на которую можно ссылаться или возвращать.

Пример использования:

```typescript
type Flatten<Type> = Type extends Array<infer Item> ? Item : Type;
```

```typescript
type FunctionReturnType<FunctionType extends (args: any) => any> = 
	FunctionType extends (...args: any) => infer ReturnType ? ReturnType : any;
```

Вот что происходит в этом коде: 
- Здесь сказано, что `FunctionType` расширяет `(args: any) => any`.
- Мы указываем на то, что `FunctionReturnType` — это условный тип.
- Мы проверяем, расширяет ли `FunctionType (...args: any) => infer ReturnType`

Сделав всё это, мы можем извлечь возвращаемый тип любой функции:

```typescript
FunctionReturnType<typeof getRandomInteger>; // number
```
