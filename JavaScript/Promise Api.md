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

Пример реализации `Promise.all`

```js
function myPromiseAll(taskList) {
  //to store results 
  const results = [];
  
  //to track how many promises have completed
  let promisesCompleted = 0;

  // return new promise
  return new Promise((resolve, reject) => {

    taskList.forEach((promise, index) => {
     //if promise passes
      promise.then((val) => {
        //store its outcome and increment the count 
        results[index] = val;
        promisesCompleted += 1;
        
        //if all the promises are completed, 
        //resolve and return the result
        if (promisesCompleted === taskList.length) {
          resolve(results)
        }
      })
         //if any promise fails, reject.
        .catch(error => {
          reject(error)
        })
    })
  });
}
```

Пример реализации `Promise.allSettled`

```javascript
function promiseAllSettled(promises) {
  return new Promise(resolve => {
    const results = [];
    let pending = promises.length;

    if (pending === 0) {
      return resolve([]);
    }

    promises.forEach((p, i) => {
      Promise.resolve(p)
        .then(value => {
          results[i] = { status: "fulfilled", value };
        })
        .catch(reason => {
          results[i] = { status: "rejected", reason };
        })
        .finally(() => {
          pending--;
          if (pending === 0) {
            resolve(results);
          }
        });
    });
  });
};
const p1 = Promise.resolve(42);
const p2 = Promise.reject("Ошибка");
const p3 = new Promise(res => setTimeout(() => res("Готово"), 200));

promiseAllSettled([p1, p2, p3]).then(console.log);
```

Пример реализации `Promise.race`

```javascript
function promiseRace(promises) {
  return new Promise((resolve, reject) => {
    for (const p of promises) {
      Promise.resolve(p)
        .then(resolve)
        .catch(reject);
    }
  });
};

const p1 = new Promise(res => setTimeout(() => res("Первый!"), 1000));
const p2 = new Promise(res => setTimeout(() => res("Второй!"), 500));
const p3 = new Promise((_, rej) => setTimeout(() => rej("Ошибка!"), 200));

promiseRace([p1, p2, p3])
  .then(result => console.log("Результат:", result))
  .catch(error => console.log("Ошибка:", error));
// через 200 мс p3 завершится с ошибкой
```

Пример реализации `Promise.any`

```javascript
function promiseAny(promises) {
  return new Promise((resolve, reject) => {
    let errors = [];
    let pending = promises.length;

    if (pending === 0) {
      // По спецификации — сразу ошибка AggregateError
      return reject(new AggregateError([], "All promises were rejected"));
    }

    for (let i = 0; i < promises.length; i++) {
      Promise.resolve(promises[i])
        .then(value => {
          resolve(value);          // Первый успешно завершившийся — победитель
        })
        .catch(err => {
          errors[i] = err;
          pending--;

          if (pending === 0) {
            // Если все завершились с ошибкой
            reject(new AggregateError(errors, "All promises were rejected"));
          }
        });
    }
  });
};
const p1 = new Promise((_, rej) => setTimeout(() => rej("Ошибка 1"), 300));
const p2 = new Promise((_, rej) => setTimeout(() => rej("Ошибка 2"), 100));
const p3 = new Promise(res => setTimeout(() => res("Успех!"), 200));

promiseAny([p1, p2, p3])
  .then(result => console.log("Результат:", result))
  .catch(err => console.log("AggregateError:", err.errors));
// через 200 мс p3 успешно завершится  
```

Разные задачи про промисы

```javascript
function resolveAfter2Seconds(x) {
	return new Promise((resolve) => {
		setTimeout(() => {
			resolve(x);
		}, 2000);
	});
}

async function add1(x) {
	const a = await resolveAfter2Seconds(20); // 2
	const b = await resolveAfter2Seconds(30); // 2

	return x + a + b;
}

async function add2(x) {
	const promise_a = resolveAfter2Seconds(20);
	const promise_b = resolveAfter2Seconds(30);

	return x + (await promise_a) + (await promise_b); // 2
}

add1(10).then(console.log) // вернёт результат через 4 секунды
add2(20).then(console.log) // вернёт результат через 2 секунды
```

Задачи на "проваливание" промисов.

Если then() передаётся что-то отличное от функции (например, промис), это интерпретируется как then(null) и в следующий по цепочке промис «проваливается» результат предыдущего.

```javascript
Promise
  .reject('a')
  .catch(p => p + 'b')
  .catch(p => p + 'c')
  .then(p => p + 'd')
  .finally(p => p + 'e')
  .then(p => { console.log(p); })
  
// Вывод: abd
```

```javascript
Promise
  .resolve('a')
  .catch(p => p + 'f')
  .then(p => p + 'c')
  .then(p => p + 's')
  .then(p => { console.log(p); })
  
// Вывод: acs
```

```javascript
Promise
  .reject('a')
  .then(p => p + '1', p => p + '2')
  .catch(p => p + 'b')
  .catch(p => p + 'c')
  .then(p => p + 'd1')
  .then('d2')
  .then(p => p + 'd3')
  .finally(p => p + 'e')
  .then(p => console.log(p));
  
// Вывод: a2d1d3
```

```javascript
const fn = async (n) => {
	await new Promise(res => setTimeout(res, 100));
	return n*n;
};

asyncLimit(fn, 50)(5); // rejected: Превышен лимит времени исполнения
asyncLimit(fn, 150)(5); // resolved: 25

const fn2 = async (a, b) => {
	await new Promise(res => setTimeout(res, 120));
	return a + b;
};

asyncLimit(fn2, 100)(1, 2); // rejected: Превышен лимит времени исполнения
asyncLimit(fn2, 150)(1, 2); // resolved: 3

const asyncLimit = (fn, delay) => {
	return async (...args) => {
		return Promise.race([
			fn(...args),
			new Promise((_, reject) => {
				setTimeout(() => reject("Timeout error"), delay)
			})
		])
	}
};

```

