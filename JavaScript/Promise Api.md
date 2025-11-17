`Promise.all` позволяет запустить множество промисов параллельно и дождаться, пока все они выполнятся. Если любой из промисов завершится с ошибкой, то промис, возвращённый `Promise.all`, немедленно завершается с этой ошибкой.

```js
let urls = [
'https://api.github.com/users/iliakan',
'https://api.github.com/users/remy',
'https://api.github.com/users/jeresig'
];

// Преобразуем каждый URL в промис, возвращённый fetch
let requests = urls.map(url => fetch(url));

// Promise.all будет ожидать выполнения всех промисов
Promise.all(requests)
	.then(responses => responses.forEach(
		response => alert(`${response.url}: ${response.status}`
)));
```

`Promise.allSettled` всегда ждёт завершения всех промисов. В массиве результатов будет

- `{status:"fulfilled", value:результат}` для успешных завершений,
- `{status:"rejected", reason:ошибка}` для ошибок.

`Promise.race` - очень похож на `Promise.all`, но ждёт только первый _выполненный_ промис, из которого берёт результат (или ошибку).

`Promise.any` - похож на `Promise.race`, но ждёт только первый _успешно выполненный_ промис, из которого берёт результат.

`Promise.resolve(value)` – возвращает успешно выполнившийся промис с результатом `value`.

`Promise.reject(error)` – возвращает промис с ошибкой `error`

[Подробнее](https://learn.javascript.ru/promise-api#promise-resolve-reject)
