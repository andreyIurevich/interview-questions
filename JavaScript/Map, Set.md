Map – это коллекция ключ/значение, как и `Object`. Но основное отличие в том, что `Map` позволяет использовать ключи любого типа.

Методы и свойства:

- [`new Map()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Map/Map) – создаёт коллекцию.
- [`map.set(key, value)`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Map/set) – записывает по ключу `key` значение `value`.
- [`map.get(key)`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Map/get) – возвращает значение по ключу или `undefined`, если ключ `key` отсутствует.
- [`map.has(key)`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Map/has) – возвращает `true`, если ключ `key` присутствует в коллекции, иначе `false`.
- [`map.delete(key)`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Map/delete) – удаляет элемент (пару «ключ/значение») по ключу `key`.
- [`map.clear()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Map/clear) – очищает коллекцию от всех элементов.
- [`map.size`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Map/size) – возвращает текущее количество элементов.

Перебор (в цикле for of):

```javascript
for (let entry of recipeMap)
```

Set - это особый вид коллекции: «множество» значений (без ключей), где каждое значение может появляться только один раз.

Основные методы:
1. set.add(value) - добавление значения
2. set.has(value) - возвращает `true` если значение присутвует в множестве, иначе `false`
3. set.size - возвращает количество элементов в множестве.

Перебор элементов осуществляется в цикле for of

[Подробнее](https://learn.javascript.ru/map-set#object-fromentries-object-iz-map)

Передача параметров в Set

```javascript
let set1 = new Set('abcdc');

for (let ch of set1) {
	console.log(ch);
}

// Вывод: a b c d - уникальные значения
```

```javascript
let set2 = new Set([1, 2, 2, 3]);

for (let item of set2) {
	console.log(item);
}

// Вывод: 1, 2, 3 - уникальные значения
```

